# 深度学习的JavaScript基础：从callbacks到sync/await

> 最近在读一本《基于浏览器的深度学习》，书比较薄，但是涉及的内容很多，因此在读的过程中不得不再查阅一些资料，以加深理解。我目前从事的本职工作就是浏览器研发，对于前端技术并不陌生。但是从前段时间开发微信小程序**识狗君**的过程来看，对JavaScript还是掌握得太少，特别是对一些前端框架以及一些比较新的JavaScript语法和编程模型，了解的不够。在修改tfjs-core源码时，就体会到这种痛苦。好吧，既然无法避开，那就正面刚吧。

这篇文章就谈一谈JavaScript中的异步编程。文章参考了网上的一些资料，主要示例代码来自**Async JavaScript: From Callbacks, to Promises, to Async/Await**一文，点击公众号的阅读原文，可以跳转该文章。

在编写微信小程序时，就被代码中的回调、sync/await整得一脸懵。对于程序员来说，多线程应该是再熟不过的概念，碰到耗时的IO操作，为了不阻塞用户界面的响应，首先想到的方法多半是采用多线程。然而对于前端开发来说，这种方法是不可行的，因为Javascript采用了单线程运行模型。注意，JavaScript只在一个线程上运行，不代表JavaScript引擎只有一个线程。事实上，JavaScript引擎有多个线程，单个脚本只能在一个线程上运行，其他线程都是在后台配合。

JavaScript之所以采用单线程，而不是多线程，跟历史有关系。JavaScript从诞生起就是单线程，原因是不想让浏览器变得太复杂，因为多线程需要共享资源、且有可能修改彼此的运行结果，对于一种网页脚本语言来说，这就太复杂了。后来 HTML5 引入了web workers，为Web内容在后台线程中运行脚本提供了一种简单的方法。但这种方法还未被广泛使用，不在本文讨论范围之内。

虽然JavaScript脚本运行在单线程中，但一些耗时或需要等待的操作，可以通过异步回调方式实现，这就是本文将要谈到的第一种方法：callbacks。

### callbacks

在JavaScript中，callbacks是一个比较宽泛的概念，当你将函数的引用作为参数传递给一个函数时，这个作为参数传递的函数就称作回调函数。比如：

```javascript
function add (x, y) {
  return x + y
}

function addFive (x, addReference) {
  return addReference(x, 5) // 15 - Press the button, run the machine.
}

addFive(10, add) // 15
```

上述代码中add函数就可以称作回调函数。所以说，callabcks通常有两种用途，一种就是作为处理函数，对数据进行处理，前端程序员应该很熟悉如下的用法：

```javascript
[1,2,3].map((i) => i + 5)

_.filter([1,2,3,4], (n) => n % 2 === 0 );
```

代码中使用了lambda表达式，算是一种匿名函数。另一种使用方法更为广泛，延迟执行某个函数，到特定的时间、或者等到数据，或者是等用户进行了操作：

```javascript
$('#btn').on('click', () =>
  console.log('Callbacks are everywhere')
)

const id = 'tylermcginnis'

$.getJSON({
  url: `https://api.github.com/users/${id}`,
  success: updateUI,
  error: showError,
})
```

这也是本文所谈到的异步编程。在上面的代码中getJSON调用会立即返回，不会阻塞主线程运行，数据获取成功之后，会调用updateUI，如果失败，则调用showError。

看似异步编程在JavaScript中得到了解决，但callbacks这种方案并不完美。第一个不足之处，就是所谓的“回调地狱”。看以下一段代码：

```javascript
// updateUI, showError, and getLocationURL are irrelevant.
// Pretend they do what they sound like.

const id = 'tylermcginnis'

$("#btn").on("click", () => {
  $.getJSON({
    url: `https://api.github.com/users/${id}`,
    success: (user) => {
      $.getJSON({
        url: getLocationURL(user.location.split(',')),
        success (weather) {
          updateUI({
            user,
            weather: weather.query.results
          })
        },
        error: showError,
      })
    },
    error: showError
  })
})
```

有没有觉得晕，人通常习惯于线性思维，顺序执行的代码容易理解，但上面的代码嵌套太多。这还不是嵌套最多的，我之前编写微信小程序，参考的代码有嵌套七八层的，看得令人绝望。这种多层嵌套容易出错，也不好调试。虽然我们可以采用一些模块化技术，改善代码的阅读性，但无法从根本上解决这一问题。

callbacks的另一个问题是“控制反转”，当你的代码调用另一个函数，如果这个函数并不是你编写的，你就失去了控制权。万一你调用的回调函数执行了非常耗时的操作，但又没有考虑异步，你也无法控制。如果你调用的是jQuery、lodash以及JavaScript内置库时，可以放心的假设它们会及时返回。但是，对于众多第三方库，你还会这么放心吗？ 第三方库可能有意或无意破坏了它们与回调的交互方式。

### Promise

为了解决callbacks的种种不足，一些聪明人提出了Promise的思路。为了理解这一方案，我们先从日常生活的一个场景出发，作为一名都市人，估计大家都有去餐馆等位置的经历吧！最傻的一种方式就是叫号，这也是大多数餐厅采用的方法，大家都排在餐厅的门口，有了空位再按先来后到的顺序就餐。后来有的商家做了改进，留下电话号码，快到有位子的时候，通过短信或者微信通知。在等待的这段时间，客户可以在附近逛逛，只要不是离得太远。仔细想想，第一种方式类似于编程中的同步模型，客户需要一直死等，第二种方式类似于前面的回调模型。回调模式的问题在哪？想想我们平常收到的推销电话，有没有可能就是你在一次不经意的留下电话号码招来的？我们无法保证每个餐厅都能按良心办事，只用于这次的餐厅等位通知。

两种方式都存在不足，于是有人想出了第三种方案，就是如下图所示的蜂鸣器：

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/js_async_01.png)

这种小装备在国内不多见，反正我是没见过。不过简单解释一下，很容易明白其工作原理。当蜂鸣器嗡嗡作响并发光时，表明已经有桌子空出来。实际上，蜂鸣器将处于三种不同状态之一：待处理、接受或拒绝。

* **待处理**是默认的初始状态。当他们给您蜂鸣器时，它就处于这种状态。

* 蜂鸣器闪烁表明您的桌子准备就绪，蜂鸣器处于**接受**状态。

* 出现问题时（也许餐厅快要关门了，或者他们忘了有人把餐厅租了一晚），蜂鸣器将处于**拒绝**状态。

在现实中，这种方案有很多细节需要考虑，蜂鸣器通讯范围多广（会不会走太远，收不到信号？）、客人拿了蜂鸣器不归还怎么办？但是将这种方案用在解决JavaScript中的异步问题，就不存在上述问题，又能很好的解决控制权反转问题，这就是JavaScript中的Promise。

Promise有三种状态：pending, fulfilled和rejected。如果异步请求仍在进行中，则Promise的状态将为pending。如果异步请求已成功完成，则Promise将变为fulfilled状态。如果异步请求失败，则Promise将变为rejected状态。是不是和前面用于解决餐厅等位问题的蜂鸣器很像？

了解Promise存在的原因以及它们可能处于的不同状态后，我们还需要回答三个问题：

* 如何创建Promise？
* 如何更改Promise的状态？
* 当Promise状态发生变化时，您该如何监听？

#### 创建Promise

第一个问题很好回答，直接new一个Promise的实例即可：

```javascript
const promise = new Promise()
```

注意并非所有浏览器都支持Promise对象，自 Chrome 32、Opera 19、Firefox 29、Safari 8 和 Microsoft Edge 起，promise 默认启用，所以使用前请确认你所使用的浏览器内核。

#### 修改Promise的状态

Promise构造函数接受一个参数，即（回调）函数。该函数将传递两个参数：resolve和reject。

* resolve: 将Promise状态修改为fulfilled的函数。

* reject: 将Promise状态修改为rejected的函数。

在下面的代码中，我们使用setTimeout等待2秒，然后调用resolve，Promise状态将变为fulfilled。

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve() // Change status to 'fulfilled'
  }, 2000)
})
```

我们可以通过在创建Promise后立即输出Promise值，然后在大约2秒钟后resolve被调用后再次输出Promise值，来观察到这种变化。

![image](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/201912/images/js_async_02.gif)

注意到没有，Promise从**pending**状态变为**resolved**。

#### 监听Promise状态变化

这是最重要的问题。如果状态更改后我们不知道如何做，那毫无用处。

创建新的Promise时，实际上只是在创建一个普通的JavaScript对象。该对象可以调用then和catch这两个方法，这两个方法都接受一个回调函数作为参数。当Promise的状态变为fulfilled时，传递给.then的函数将被调用。当一个Promise的状态更改为rejected时，将调用传递给.catch的函数。

让我们来看一个例子。我们将再次使用setTimeout两秒钟（2000毫秒）后将Promise状态变为fulfilled。

```javascript
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

尝试着运行上面的代码，大约2秒钟后，在浏览器控制台中可看到“Success！”。

这个过程发生了什么？首先，当我们创建Promise时，我们在约2000毫秒后调用了resolve，这将Promise的状态更改为fulfilled。其次，我们将onSuccess函数传递给promises的.then方法。这样做，我们告诉了Promise，当Promise的状态更改为fulfilled时调用onSuccess，它在大约2000毫秒后执行。

再来看看rejected情况下的代码：

```javascript
function onSuccess () {
  console.log('Success!')
}

function onError () {
  console.log('💩')
}

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject()
  }, 2000)
})

promise.then(onSuccess)
promise.catch(onError)
```

这种情况下，onError将会被调用，因为2000毫秒后，reject被调用了。

---

回头再看看前面的异步代码：

```javascript
function getUser(id, onSuccess, onFailure) {
  $.getJSON({
    url: `https://api.github.com/users/${id}`,
    success: onSuccess,
    error: onFailure
  })
}

function getWeather(user, onSuccess, onFailure) {
  $.getJSON({
    url: getLocationURL(user.location.split(',')),
    success: onSuccess,
    error: onFailure,
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis", (user) => {
    getWeather(user, (weather) => {
      updateUI({
        user,
        weather: weather.query.results
      })
    }, showError)
  }, showError)
})
```

用Promise的方式该如何改写呢？首先看看getUser这个函数的改写：

```javascript
function getUser(id) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: `https://api.github.com/users/${id}`,
      success: resolve,
      error: reject
    })
  })
}
```

注意到没有，getUser的参数有所变化，仅接收ID，不再需要其他两个回调函数，保证不会发生**控制反转**。如果请求成功，则将调用resolve；如果发生错误，则将调用reject。

同样的方式改写getWether函数：

```javascript
function getWeather(user) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: getLocationURL(user.location.split(',')),
      success: resolve,
      error: reject,
    })
  })
}
```

接着改写按钮点击处理：

```javascript
$("#btn").on("click", () => {
  const userPromise = getUser('tylermcginnis')

  userPromise.then((user) => {
    const weatherPromise = getWeather(user)
    weatherPromise.then((weather) => {
      updateUI({
        user,
        weather: weather.query.results
      })
    })

    weatherPromise.catch(showError)
  })

  userPromise.catch(showError)
})
```

代码的逻辑就是根据id获取用户信息，然后通过用户所在的地理位置获取天气信息，最后更新到用户界面上。

整条逻辑就像是一个线性处理过程，事实上，通过Promise的链式结构，我们可以将代码写得更紧凑一些。

```javascript
$("#btn").on("click", () => {
  getUser("tylermcginnis")
    .then(getWeather)
    .then((weather) => {
      // We need both the user and the weather here.
      // Right now we just have the weather
      updateUI() // ????
    })
    .catch(showError)
})
```

上面的代码看起来很简练，但实际上隐藏着一个问题。在第二个.then中，我们要调用updateUI。问题是我们需要同时给updateUI传递用户和天气。但上面的代码中，我们只传递了天气信息，而没有用户信息。我们需要以某种方式找到一种实现方法，以便在getWeather返回的Promise在resolve时，用户和天气都可以传递。

解决问题的关键在于，resolve只是一个函数，传递给它的任何参数都将传递给给.then的函数。这意味着在getWeather内部，如果我们调用自己的resolve方法，则可以将天气和用户传递给它。这样，链中的第二个.then方法将同时接收用户和天气作为参数。

```javascript
function getWeather(user) {
  return new Promise((resolve, reject) => {
    $.getJSON({
      url: getLocationURL(user.location.split(',')),
      success(weather) {
        resolve({ user, weather: weather.query.results })
      },
      error: reject,
    })
  })
}

$("#btn").on("click", () => {
  getUser("tylermcginnis")
    .then(getWeather)
    .then((data) => {
      // Now, data is an object with a
      // "weather" property and a "user" property.

      updateUI(data)
    })
    .catch(showError)
})
```

比较以下Callbacks和Promise的实现代码，是不是Promise更容易理解？

```javascript
// Callbacks 🚫
getUser("tylermcginnis", (user) => {
  getWeather(user, (weather) => {
    updateUI({
      user,
      weather: weather.query.results
    })
  }, showError)
}, showError)


// Promises ✅
getUser("tylermcginnis")
  .then(getWeather)
  .then((data) => updateUI(data))
  .catch(showError);
```

### async/await

上面的Promise方案解决了Callbacks的两大重要缺陷，但还存在不足，我们需要将用户数据从第一个异步请求一直传递到最后一个.then。这使得我们修改getWeather函数，使其可以传递用户。

有没有什么方法可以让我们以编写同步代码的方式编写异步代码呢？假如我们以同步方式实现上述的功能，大概写法如下：

```javascript
$("#btn").on("click", () => {
  const user = getUser('tylermcginnis')
  const weather = getWeather(user)

  updateUI({
    user,
    weather,
  })
})
```

如何让Javascript引擎知道这里getUser和getWeather实际上是一个异步方法呢？这时就该async/await登场了。

```javascript

$("#btn").on("click", async () => {
  const user = await getUser('tylermcginnis')
  const weather = await getWeather(user.location)

  updateUI({
    user,
    weather,
  })
})
```

首先，函数前的async修饰告诉引擎，该函数中存在异步调用。其次，代码中的await这表示这个调用是一个异步调用，将返回一个Promise。在await的地方，代码将等待，直到异步调用返回Promise。

函数前加上async，代表函数将返回一个Promise，即使像下面这样的空函数，也会隐式返回一个Promise：

```javascript
async function getPromise(){}

const promise = getPromise()
```

如果async函数返回了值呢？如以下代码所示，该值将封装到Promise中：

```javascript
async function add (x, y) {
  return x + y
}

add(2,3).then((result) => {
  console.log(result) // 5
})
```

需要注意的是，await只能用在async函数中，比如下面的代码，会出错：

```javascript
$("#btn").on("click", () => {
  const user = await getUser('tylermcginnis') // SyntaxError: await is a reserved word
  const weather = await getWeather(user.location) // SyntaxError: await is a reserved word

  updateUI({
    user,
    weather,
  })
})
```

也就是说，当async加到函数时，会产生两种结果：

* 使函数本身返回（或包装返回的内容）一个promise
* 可以在其中使用await。

### 小结

好了，关于JavaScript中的异步编程就探讨到这儿，是不是和我们平常采用的Python、Java或C++语言不太一样。有人说，学一门语言，实际上是学习一种编程思路，你没有想到JavaScript会用这种方式来解决异步编程吧！这篇文章你看了之后，是醍醐灌顶，还是更加迷糊呢？欢迎留言探讨。

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)