
* Oilpan gives you an option to have your objects on (Oilpan's) heap, which means their lifetime is controlled by Oilpan, not you (e.g. with RefCounted).
* At an arbitrary time after your objects become unreachable, Oilpan collects memory (sweep) for these objects.
* Reference cycle is allowed in on-heap objects, unlike RefCounted objects.



Oilpan is a garbage collector written in C++ for managing C++ memory.

This post is the first in a series of Oilpan blog posts which will provide an overview of the core principles of Oilpan and its C++ APIs. For this post we will cover some of the supported features, explain how they interact with various subsystems of the garbage collector, and do a deep dive into concurrently reclaiming objects in the sweeper.

#### Background

Oilpan implements a Mark-Sweep garbage collector where garbage collection is split among two phases: marking where the managed heap is scanned for live objects, and sweeping where dead objects on the managed heap are reclaimed.

To recap, scanning all objects for live ones can be seen as graph traversal where objects are nodes and pointers between objects are edges. Traversal starts at roots which are registers, native execution stack (which we will call stack from now on), and other globals, as described here.

```
class LinkedNode final : public GarbageCollected<LinkedNode> {
 public:
  LinkedNode(LinkedNode* next, int value) : next_(next), value_(value) {}
  void Trace(Visitor* visitor) const {
    visitor->Trace(next_);
  }
 private:
  Member<LinkedNode> next_;
  int value_;
};

LinkedNode* CreateNodes() {
  LinkedNode* first_node = MakeGarbageCollected<LinkedNode>(nullptr, 1);
  LinkedNode* second_node = MakeGarbageCollected<LinkedNode>(first_node, 2);
  return second_node;
}
```

In the example above, LinkedNode is managed by Oilpan as indicated by inheriting from GarbageCollected<LinkedNode>. When the garbage collector processes an object it discovers outgoing pointers by invoking the Trace method of the object. The type Member is a smart pointer that is syntactically similar to e.g. std::shared_ptr, which is provided by Oilpan and used to maintain a consistent state while traversing the graph during marking. All of this allows Oilpan to precisely know where pointers reside in its managed objects.

Avid readers probably noticed and may be scared that first_node and second_node are kept as raw C++ pointers on the stack in the example above. Oilpan does not add abstractions for working with the stack, relying solely on conservative stack scanning to find pointers into its managed heap when processing roots. This works by iterating the stack word-by-word and interpreting those words as pointers into the managed heap. This means that Oilpan does not impose a performance penalty for accessing stack-allocated objects. Instead, it moves the cost to the garbage collection time where it scans the stack conservatively. Oilpan as integrated in the renderer tries to delay garbage collection until it reaches a state where it’s guaranteed to have no interesting stack. Since the web is event based and execution is driven by processing tasks in event loops, such opportunities are plentiful.

Oilpan is used in Blink which is a large C++ codebase with lots of mature code and thus also supports:

Multiple inheritance through mixins and references to such mixins (interior pointers).
Triggering garbage collection during executing constructors.
Keeping objects alive from non-managed memory through Persistent smart pointers which are treated as roots.
Collections covering sequential (e.g. vector) and associative (e.g. set and map) containers with compaction of collection backings.
Weak references, weak callbacks, and ephemerons.
Finalizer callbacks that are executed before reclaiming individual objects.

#### Sweeping for C++

Stay tuned for a separate blog post on how marking in Oilpan works in detail. For this article we assume marking is done and Oilpan has discovered all reachable objects with the help of their Trace methods. After marking all reachable objects have their mark bit set.

Sweeping is now the phase where dead objects (those unreachable during marking) are reclaimed and their underlying memory is either returned to the operating system or made available for subsequent allocations. In the following we show how Oilpan’s sweeper works, both from a usage and constraint perspective, but also how it achieves high reclamation throughput.

The sweeper finds dead objects by iterating the heap memory and checking the mark bits. In order to preserve the C++ semantics, the sweeper has to invoke the destructor of each dead object before freeing its memory. Non-trivial destructors are implemented as finalizers.

From the programmer’s perspective, there is no defined order in which destructors are executed, as the iteration used by the sweeper does not consider construction order. This imposes a restriction that finalizers are not allowed to touch other on-heap objects. This is a common challenge for writing user-code that requires finalization order as managed languages generally do not support order in their finalization semantics (e.g. Java). Oilpan uses a Clang plugin that statically verifies, among many other things, that no heap objects are accessed during destruction of an object:

```
class GCed : public GarbageCollected<GCed> {
 public:
  void DoSomething();
  void Trace(Visitor* visitor) {
    visitor->Trace(other_);
  }
  ~GCed() {
    other_->DoSomething();  // error: Finalizer '~GCed' accesses
                            // potentially finalized field 'other_'.
  }
 private:
  Member<GCed> other_;
};
```

For the curious: Oilpan provides pre-finalization callbacks for complex use cases that require access to the heap before objects are destroyed. Such callbacks impose more overhead than destructors on each garbage collection cycle though and are only used sparingly in Blink.

#### Incremental and concurrent sweeping

Now that we have covered the restrictions of destructors in a managed C++ environment, it is time to look at how Oilpan implements and optimizes the sweeping phase in more detail.

Before diving into details it is important to recall how programs in general are executed on the web. Any execution, e.g., JavaScript programs but also garbage collection, is driven from the main thread by dispatching tasks in an event loop. The renderer, much like other application environments, supports background tasks that run concurrently to the main thread to aid processing any main-thread work.

Starting out simple, Oilpan originally implemented stop-the-world sweeping which ran as a part of the garbage collection finalization pause interrupting executing of the application on the main thread: