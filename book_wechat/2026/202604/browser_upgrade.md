---
title: 实战备忘录：Claude Code + superpowers进行浏览器内核升级
cover: ./images/browser_upgrade_title.png
---
> 纸上得来终觉浅，绝知此事要躬行。

不知道大家有没有这样的感受：一方面，互联网上充斥着各种炸裂的案例：一人公司、AI 改变生产力的故事层出不穷；另一方面，亲自上手时却发现，现实与这些叙事之间存在明显落差。有人不懂编程，借助 AI 做出了完整的应用，但当我们自己使用 AI 时，可能修复一个 bug，烧了无数 tokens 仍不能解决。又或者尝试实现一个新功能，粗看效果很经验，但落地发现问题很多。

归根结底，AI 目前仍然只是工具。它的上限，很大程度上取决于使用者本身的认知、经验与问题拆解能力。单纯阅读他人的经验，很难直接迁移到自己的场景中。比如某些基于 Claude Code 插件或 skill 实现的高光案例，即使按照同样的提示词，同样的流程，往往难以复现。每个人面对的业务背景、技术栈与约束条件各不相同，而 AI 也远非万能。因此，唯有通过不断实践、复盘与总结，才能逐步厘清哪些问题适合交给 AI，哪些仍需人工介入，以及如何将两者有效结合。

本文记录了一次在浏览器内核升级中引入 AI 的实践过程。谈不上经验总结，更多是对实际问题的还原：过程中遇到了哪些障碍，如何应对的。最终结果谈不上完美，也未必是最优解。将其记录出来，既是一次自我沉淀，也希望能和读者朋友一起探讨。

## 背景

做了十几年的浏览器产品开发，最让人头疼的，是内核升级。

绝大多数浏览器产品都是基于 Chromium 这个开源项目构建的，我们也不例外。问题在于它的迭代速度实在太快。从我最早接触 Chromium 时的 10 版本，到如今的 146，不仅版本号的在疯狂增加，更剧烈的是其内部代码的持续重构与功能演进。几乎每隔几个版本，整体结构就会发生明显变化。

另外一个，我们对 Chromium 的定制修改，通常无法合入上游，导致和上游代码存在分叉。出于稳定性考虑，实际项目中往往会选定一个当时最新的版本作为基线，在其之上进行功能开发，并维持一到两年的内核冻结周期。但问题在于，内核终究需要升级，否则一些依赖新 Web 技术的站点将无法正常支持，有些 bug 可能在新内核中修复。

而一旦开始升级，问题就来了：原有的定制修改，很可能对应的代码早已被重构甚至重写，难以直接迁移。如果只是行号偏移或函数签名变化，这类问题还算简单，手工调整一下就可以。真正棘手的是底层技术框架发生变化，导致原有实现逻辑在新体系中无处安放。这种情况下，几乎等同于重做一遍。

也正因如此，每一次内核升级都异常漫长且痛苦。少则数月，多则接近一年，还要叠加各种兼容性问题和回归 bug 的修复。等升级工作基本稳定，往往还没来得及推进新功能，就又要面对下一轮内核升级的压力。

回顾过去的工作内容，大致可以归为两类：功能定制与内核升级，而后者往往占据了更多时间和精力。

最近 AI 的进展令人惊喜，如是就思考，是否可以将它引入到内核升级这一过程之中？从直觉上看，AI 非常适合处理这类任务。例如简单的 patch 迁移，AI 在效率和一致性上肯定优于人工。但更关键的问题在于：当面对代码重构甚至框架演进时，AI 是否能够理解新的架构，同时还原旧版本中的修改意图，并在新代码体系中重新实现这套逻辑？

这次升级，我选择了 Claude Code + Superpowers 工具，AI大模型主要是智谱的 GLM 4.7。

## 使用Claude Code初始化项目

Chromium 的源码有上千万行，让 Claude Code 去扫描全部的源代码不现实，所以我选择使用 Chromium 的文档来初始化项目。

从 源码的 agents 目录下复制 skills 到 .claude 目录下，里面包含了 chromium-docs skill:

```bash
cp -r ./agents/skills .claude/
```

根据 docs 下的文档初始化，避免扫描庞大的 Chromium源码，来进行 Claude Code 的项目初始化：

![](./images/browser_upgrade_01.png)

会生成 CLAUDE.md 文件，包含如下信息：

```
  1. 构建命令 - Debian 打包构建和 GN/Ninja 手动构建
  2. 架构概览 - Chromium 的分层架构说明                                                                                                                                 
  3. 关键架构原则 - Content vs Chrome、多进程、Mojo IPC、沙箱
  4. Content 层进程结构 - browser/renderer/common 分离                                                                                                                  
  5. 组件指南 - 共享功能的结构和依赖规则                                                                                                                              
  6. 服务 - Mojo 服务及其组织方式                                                                                                                                       
  7. 测试 - 测试类型和运行方法                                                                                                                                        
  8. 重要目录 - 目录到用途的映射表
  9. GN 构建系统 - GN 和 Ninja 概述
  10. 文档访问 - 使用 chromium-docs 技能的方法
  11. 仓库上下文 - Deepin Chromium 衍生版的包装信息
```

## Superpowers工具

Superpowers 是一套面向开发者的 AI 能力增强框架，通过 Skills、Commands 和结构化工作流，让大模型具备工程化执行能力。它不仅能生成代码，还能参与完整的软件开发流程，并支持能力复用与自动化协作，显著提升复杂项目中的开发效率。

Superpowers 安装很简单，不需要额外的配置，也没有复杂的依赖。

整个过程在 Claude Code 里完成，只要三步。

### 步骤一：把插件加入市场

在 Claude Code 的终端中执行：

```bash
/plugin marketplace add obra/superpowers-marketplace
```

### 步骤二：安装插件

```bash
/plugin install superpowers@superpowers-marketplace
```

### 步骤三：重新加载插件

```bash
/reload-plugins
```

可以输入 /superpowers 确认一下是否加载成功。

![](./images/browser_upgrade_02.png)

## 制定升级计划

首先在 127 内核版本的浏览器代码上生成 patch 文件，每个提交生成一个 patch 文件。放到 146 源码的 patches 目录下。然后在 Claude Code 中进行头脑风暴。

> /superpowers:brainstorm 我之前将 chromium 127 内核基础上进行了功能定制，现在内核升级到 chromium 146，需要在新内核上将修改迁移过来，patch按照顺序放在 patches 目录下

接下来 Claude Code 就询问一些问题，经过确认后，会生成 spec 和 plan 文档。文档生成在 docs/superpowers 目录下。

其中 spec 文档内容如下：

![](./images/browser_upgrade_03.png)

从名称上可以看出，这个文档并不只是简单描述需求，而是将整个迁移过程**结构化为可执行流程**：

-  明确目标：218 个 patch 必须逐个迁移，保证 **可应用、可编译、可启动**
-  定义严格约束：每个 patch 都要经过 **apply → build → 启动验证 → commit**
-  提供完整流程：从预检查、执行步骤到错误处理、回滚机制，形成闭环 
-  细化到脚本级：直接给出可执行的 shell 流程，实现自动化迁移 
-  配套可追踪机制：通过 migration log 和失败记录，保证过程可审计 

这份 spec 也不只是设计文档，还把复杂工程任务（如 Chromium 内核升级）转化为 AI 和人都能执行的标准化流程。如果对这份 spec 文档有不满意的地方，可以进行修改，比如对于 patch 应用失败的流程，我觉得还有改进的地方。

![](./images/browser_upgrade_04.png)

我希望能够分析v127版本的上下文，查找 patch 作用位置，再对比 v146 版本源码上下文。比如如果是 API 变更，修改 patch 代码适配新接口。如果是代码逻辑发生了变化，AI 需要理解 patch 的作用，然后在新代码上做等效的实现。

所以我对上面的文档做了少量修改（通过 AI 辅助）：

![](./images/browser_upgrade_05.png)

在更新了 spec 文档后，需要同步到计划，在 Claude Code 中输入指示：

> Update the execution plan based on the latest design.
> Include patch adaptation strategy when patch apply fails.

![](./images/browser_upgrade_06.png)

Plan 用于将设计转化为可执行的流程，将整体任务拆解为一系列有序步骤（如应用补丁、构建、验证、提交等），并为每一步定义明确的验证标准以及失败时的处理路径。与侧重实现思路的 Design 不同，Plan 更关注实际执行过程，确保整个流程具备确定性、可验证性和可重复性。

## 执行升级计划

在 Plan 文档生成并确认无误后，就可以开始执行计划。前面 12 个任务是做升级前的准备工作，比如生成各种验证构建成功、验证启动成功的脚本：

![](./images/browser_upgrade_07.png)

还有就是对基线版本代码进行确认，确保在做移植前代码编译和启动无问题。这些任务加上**Task 13: Apply first patch (test run)**都执行得没有问题，但执行 **Task 14: Apply patch 0002-0050**，我发现问题了，执行结果是这样的：

![](./images/browser_upgrade_08.png)

1. 没有按照我的要求，按照顺序应用补丁。
2. 不是在应用一个补丁后等我的确认，再提交。执行过程相当于批量应用补丁，跳过应用不上的补丁。
3. 虽然提示处理剩余的补丁，但和我期望的按顺序应用补丁不一样。

我检查了一下 plan 内容：

![](./images/browser_upgrade_09.png)

发现这个任务的执行过程过于简略，于是指示 Claude Code 修改计划：

> 请修改Task 14的流程，对于每个patch，都需要编译成功，且能启动成功。如果patch应用失败，要尝试走简单适配或复杂适配流程，最后都需要我确认后再提交

进过这样修改，流程变成如下：

````
**For each patch from 0002 to 0050 (excluding already applied):**

**Step 1: Check patch status**

```bash
PATCH_NUM="xxxx"  # Replace with actual patch number
PATCH_FILE="patches/${PATCH_NUM}-*.patch"

echo "=========================================="
echo "Processing Patch: $PATCH_NAME"
echo "=========================================="

# Check if already applied
git apply --check "$PATCH_FILE" 2>&1
CHECK_STATUS=$?

if [ $CHECK_STATUS -eq 0 ]; then
    echo "✓ Patch applies cleanly"
else
    echo "✗ Patch does not apply cleanly - needs adaptation"
    # Proceed to Step 2
fi
```

**Step 2: If patch fails, attempt adaptation**

```bash
# Analyze failure
git apply --check "$PATCH_FILE" 2>&1 | tee patch_error.log

# Show user the failure and ask for action
echo "=========================================="
echo "Patch failed to apply. Options:"
echo "1. Check if change already exists in baseline"
echo "2. Adapt patch manually (line offset or API change)"
echo "3. Skip this patch (only if functionality not needed)"
echo "=========================================="
# WAIT FOR USER DECISION
```

**Adaptation Actions:**

```bash
# Option 1: Check if change already exists
# Example: grep -n "is_uos_browser" build/config/features.gni

# Option 2: View patch content
less "$PATCH_FILE"

# Option 3: Find target file in v146
# Example: grep -rn "function_name" .

# Option 4: Apply with 3way merge if safe
git apply --3way "$PATCH_FILE"
```

**Step 3: Apply patch (after adaptation if needed)**

```bash
# Apply the patch
if git apply "$PATCH_FILE"; then
    echo "✓ Patch applied successfully"
elif git apply --3way "$PATCH_FILE"; then
    echo "✓ Patch applied with 3way merge"
else
    echo "✗ Failed to apply patch"
    echo "Action required: Manual adaptation or skip"
    # WAIT FOR USER
fi
```

**Step 4: Build verification**

```bash
echo "Starting build verification..."
bash scripts/verify_build.sh
BUILD_STATUS=$?

if [ $BUILD_STATUS -eq 0 ]; then
    echo "✓ Build successful"
else
    echo "✗ Build failed"
    echo "Action: Fix build error or revert patch"
    # Rollback
    git diff --name-only | xargs git checkout --
    exit 2
fi
```

**Step 5: Startup verification**

```bash
echo "Starting startup verification..."
bash scripts/verify_startup.sh
STARTUP_STATUS=$?

if [ $STARTUP_STATUS -eq 0 ]; then
    echo "✓ Startup successful"
else
    echo "✗ Startup failed"
    # Rollback
    git diff --name-only | xargs git checkout --
    exit 3
fi
```

**Step 6: Show changes and request user confirmation**

```bash
echo "=========================================="
echo "Patch $PATCH_NAME Results:"
echo "=========================================="
echo "Files modified:"
git diff --stat
echo ""
echo "Detailed changes (press q to exit):"
git diff | less
echo ""
echo "Build: ✓ SUCCESS"
echo "Startup: ✓ SUCCESS"
echo ""
echo "Ready to commit? (yes/no)"
# WAIT FOR USER INPUT
```

**Step 7: Commit with user confirmation**

```bash
read -p "Commit this patch? (yes/no): " CONFIRM
if [ "$CONFIRM" = "yes" ]; then
    # Extract patch info
    PATCH_AUTHOR=$(grep "^From:" "$PATCH_FILE" | sed 's/^From: //g')
    PATCH_DATE=$(grep "^Date:" "$PATCH_FILE" | sed 's/^Date: //g')
    PATCH_SUBJECT=$(grep "^Subject:" "$PATCH_FILE" | sed 's/^Subject: //g')

    git add .
    git commit -m "Apply patch $PATCH_NAME

From: $PATCH_AUTHOR
Date: $PATCH_DATE

$PATCH_SUBJECT"

    # Update migration log
    BUILD_TIME=$(cat build.time 2>/dev/null || echo "0")
    echo "| $PATCH_NAME | ✓ | ${BUILD_TIME}s | ✓ | Applied with verification |" >> PATCH_MIGRATION_LOG.md
    git add PATCH_MIGRATION_LOG.md
    git commit --amend --no-edit

    echo "✓ Patch committed"
else
    echo "Commit cancelled. Reverting changes..."
    git diff --name-only | xargs git checkout --
fi
```

**Patch List to Process:**
- Remaining patches 0002-0050 to process individually

```bash
# List remaining patches to process
for i in $(seq -f "%04g" 2 50); do
    PATCH_FILE="patches/${i}-*.patch"
    if [ -f "$PATCH_FILE" ]; then
        printf "%s " "$i"
    fi
done
echo ""
```
````

也就是每次应用补丁，都经过七个步骤。在应用补丁失败时，AI 会根据上下文，进行调整，并保证程序编译通过，并且能够运行成功。

### 简单patch失败的处理

比如在应用补丁 0003 时因为上游代码发生了修改，导致补丁应用失败，我让 AI 自动修复了这个问题。

![](./images/browser_upgrade_10.png)

当然，这个是一个比较简单的问题，AI 能够比较好的处理，最后处理成功：

![](./images/browser_upgrade_11.png)

### 确保patch内容都打上了

有时 AI 也会偷懒，比如第 0004 个 patch，修改了 14 个文件，AI 处理了 3 个就没有处理剩下的。

![](./images/browser_upgrade_12.png)

这个时候仍然可以调整Plan中Task的执行步骤，添加一个检查步骤，确保补丁所涉及的文件都有修改。

![](./images/browser_upgrade_13.png)

![](./images/browser_upgrade_14.png)

### 不要轻易跳过 patch

感觉 AI 也是偷懒的一把好手，比如它给了几个选项，有跳过，有深入研究，结果我选了稍后深入研究，它就想跳过这个 patch，这个时候需要手动拉回来。

![](./images/browser_upgrade_15.png)

再比如：

![](./images/browser_upgrade_16.png)

### AI 自作聪明

有时，AI 有可能自作聪明，比如在浏览器定制化UI时，会将界面上的Chromium换成别的词，AI 就自作聪明的将所有的 Chromium 从资源文件中进行了替换。

![](./images/browser_upgrade_17.png)

所以这里需要更加精确的指令。

对于 xtb 中的 ID，我下达了几次指令，但 AI 就是不修改，还给出了理由：

![](./images/browser_upgrade_18.png)

但实际上，chromium中国际化字符串的方式非常怪异，翻译文件中的ID是根据英文字符串通过hash算法计算出来的。这样就导致，如果我们修改了英文字符串，不仅需要修改翻译，还需要修改 ID。碰到这种，只能再下达精确的指示。

还有这种，触发了clean build。

![](./images/browser_upgrade_19.png)

要知道重新编译一次 Chromium，少则几个小时。其实绝大多数情况下，增量编译足够，即使修改了 gn 文件，增量编译也会自动处理。更离谱的是，Claude Code 居然将 out 目录删除了，平常做一些 grep、git 操作，都要我确认，删除 out 文件夹却没有经过我的确认。需要在 plan 文件中添加：

![](./images/browser_upgrade_20.png)

### 对抗 AI “忘事”

每完成一个 patch 迁移，我希望能记录到 PATCH_MIGRATION_LOG.md 中，但在执行 Plan 的时候，发现 AI 经常没有写这个 Log，于是在 Plan 文档中，特意增加这一步骤。

### 继续执行任务

如果我们退出 Claude Code，或者机器重启，没有继续加载上一个会话，Claude会忘记之前的任务，这个时候请Claude 再读一遍计划文档：

> /superpowers:execute-plan 请按照docs/superpowers/plans/2026-03-30-patch-migration-plan.md严格执行

如果是继续执行任务，最好明确告知，如果仅仅告知继续执行任务，有可能会从头开始。

## 使用中碰到的问题

### 1. 上下文超出模型限制

如果经过多轮对话，积累了太长的上下文，可能会超出大模型的上下文长度限制：

![](./images/browser_upgrade_21.png)

这个时候可以在 Claude Code 中执行 `/compact`指令手动压缩上下文，或者使用更精确的指令：

> Update plan.md based on the new patch adaptation strategy.
>
> Constraints:
> \- Do NOT output full file
> \- Only show modified sections
> \- Keep response concise (<2000 tokens)

执行 /compact 指令后，Claude Code 可能会忘记之前的计划，可以明确下指令让其再读一遍计划文档。

> 请再读一遍plan文档

### 2. 很多操作都需要进行确认

包括 bash、grep、git操作等等，都需要手工确认，即使前面已经确认，后面还是会。

Claude Code 有 auto mode，由 AI 判断哪些是危险操作，避免很多不必要的确认，但仅能用于 opus 模型上。解决方法是使用 "bypassPermissions" 参数跳过所有确认，不过这个操作太危险，慎用。可以使用 docker 来减少风险。

## 小结

总的来看，浏览器内核升级之所以复杂，在于其不只仅仅是将 patch 正确应用上去，更是一次架构语义的重建过程。而 AI 的引入，或许并不能从根本上消除这种复杂性，但它有潜力在**理解差异、还原意图、辅助实现**这些关键环节中，显著降低人力投入。
