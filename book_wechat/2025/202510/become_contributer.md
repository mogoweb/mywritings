# 成为一名开源贡献者

> 💡开源不是一场炫技，而是一场修行。
>
> 从「用开源」到「参与开源」，是每个开发者成长路上必经的一步。

作为一名软件开发者，这些年来我使用过无数开源软件，也曾多次修改过其中的源码，但真正向上游社区提交代码的经历却屈指可数。这并不是因为想把成果据为己有，而是出于以下几个现实原因：

* **提交门槛高。** 很多大型开源项目对代码提交者有严格限制。以我最常接触的 Chromium 和 Android 为例，这两个项目都由 Google 主导，只有内部员工才有直接提交代码的权限。
* **贡献流程复杂。** GitHub 上的项目虽然开放度高，但也对代码质量要求严格。通常需要提前阅读项目的贡献指南（Wiki），严格遵守编码规范。即便提交了 PR，如果格式、风格或逻辑不符合要求，也可能被驳回。若修改了功能，还需附上修改理由和测试用例。这通常需要来回拉扯，往往让人消磨耐心。

如今，越来越多的开源项目来自中国团队，也有越来越多的中国程序员投身其中。也许你也想为开源世界贡献一份力量。

不过，如果只是把源码上传到 GitHub，就像许多个人开发者那样，这其实只是“托管源码”，而非“参与开源”。真正的开源意味着协作与共创。如果你能发起一个让他人愿意参与的项目，当然是件令人自豪的事。但对于大多数人来说，最好的起点是——**参与已有的开源项目。**

那么，**如何成为一名开源贡献者？**

得益于 AI，如今撰写 commit 信息或 PR 说明已不再是难事。但整个流程依然需要耐心和方法。下面，我将以 Wine 项目为例，介绍从入门到提交代码的过程。

## 阅读项目 Wiki —— 从理解规则开始

开源项目的健康发展，离不开全球开发者的协作。参与者来自不同国家，背景各异，交流方式也与公司内部不同。

在公司里，改完代码，大喊一声“帮我 review 一下！”就能解决问题；

而在开源社区，沟通主要依赖邮件列表或论坛，实时聊天反而少见。

因此，每个项目都会制定详细的开发准则，以便大家协同工作。

**参与开源项目的第一步，就是认真阅读项目的 Wiki。**

但 Wiki 往往内容繁杂、英文居多，对中国开发者来说阅读门槛不低。

比如下图所示，是 Wine 的 Wiki：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/become_contributer_01.png)

一开始我看得眼花缭乱，不知道该从哪部分入手。于是我尝试求助 ChatGPT，没想到却掉进了一个“小坑”。

我问它：

> 如何向 Wine 项目提交代码？

ChatGPT 的回答开头还算靠谱，但到第三步时居然写着：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202510/images/become_contributer_02.png)

它建议我**通过邮件发送补丁**。我当时虽然了解 GitHub 的 PR 流程，但想着 ChatGPT 能联网，应该不会错吧，于是照做了。

结果注册邮件列表、发送补丁后等了几个小时毫无反应，连邮件主题都没出现在列表中。

后来仔细阅读 Wiki 才发现，Wine 早已改用 **GitLab 的 MR（Merge Request）流程**。

这次经历让我深刻体会到一个道理：

👉 **再聪明的 AI 也不能替代你自己读 Wiki。**

特别是 “Contribute” 相关章节，一定要亲自确认。

## PR / MR 流程实操

下面以 GitLab 上的 Wine 项目为例，介绍几个常见操作。

### fork wine 仓库

打开 Wine 官方 GitLab ：
https://gitlab.winehq.org/wine/wine

点击 Fork，把仓库 fork 到自己的账号下。

与 github 有些不同，wine gitlab 不允许随便创建软件仓库。Wine 项目在 GitLab 上对贡献者身份有明确要求，所以需要注意以下几点：

1. **必须使用真实姓名。**
   这不仅体现开发者的认真与承诺，也有助于社区建立信任，同时防止有人提交不合规代码（例如反编译的微软组件）。

2. **GitLab 账号资料也必须使用真实姓名。**
   姓名需要与提交记录中的一致，以便正确识别贡献者身份。

3. **Fork 与克隆代码仓库。**
   开发者需要先在 GitLab 上 fork 自己的个人副本，然后再克隆到本地进行开发。

4. **账号验证。**
   如果尚未完成账户验证，将无法创建 fork，需要在 GitLab 上提交 **User Verification（用户验证）** 申请。


### 同步上游代码

1. **查看远程仓库：**

```bash
git remote -v
```

你应能看到：

* `origin` 指向你自己在 GitLab 上的 fork；
* 若没有 `upstream`，需要手动添加：

```bash
git remote add upstream https://gitlab.winehq.org/wine/wine.git
```

---

2. **获取上游更新：**

```bash
git fetch upstream
```

---

3. **切换到本地 master 并同步：**

```bash
git checkout master
git reset --hard upstream/master
```

现在你的本地 `master` 已与上游保持一致。

---

4. **推送回自己的 fork：**

```bash
git push origin master --force
```

这样，GitLab 上的 fork 仓库也同步更新了。


### 开发与提交

1. 创建新分支

git checkout -b fix-ime-context


分支名应简洁明了，例如：

* fix-imm32-focus

* add-winex11-debug-log

* cleanup-dinput-header

2. 修改代码并提交

保持每个提交（commit）只完成一件事。

提交信息（commit message）需清晰、简短、有意义。

格式示例：

> imm32: Avoid overwriting IME context window handle across threads.


正文说明（空一行后可选写）：

> Directly assigning GetFocus() to ctx->hWnd breaks the association between
> HIMC and the correct window when running across threads.

3. 运行本地测试（可选但建议）

Wine 提供自测框架，可执行：

```
make test
```

或者仅运行与修改模块相关的测试（例如 imm32）：

```
make dlls/imm32/tests
```

### 提交 Merge Request (MR)

将分支推送到你自己的仓库：

```
git push origin fix-ime-context
```

在自己仓库的 GitLab 页面，创建 MR。

提交 MR 后，GitLab CI 会自动运行构建与测试。

⚠️ 注意：Wine 不允许多个无关提交合并在同一个 MR 中。

如果修改较多，请拆分成多个独立的 MR。

### 代码审核（Review）

审核者通常是维护对应模块的 Wine 开发者。

通常代码 review 不是那么顺利，运气比较好的话，可能只是一些代码格式上的问题。更多的是需要向 reviewer 解释为什么需要这么修改，是否需要补充测试用例。

经过几个来回，也许 master 代码已经有新的提交，而你之前基于 master 创建的修改分支已经落后 master 分支，这时需要 rebase 后再进行修改。

先根据前面 **同步上游代码** 的步骤更新 master 分支代码，然后 rebase:

```
git rebase master
```

然后再根据反馈修改代码并使用：

```
git commit --amend
git push origin fix-ime-context --force
```

⚠️ 注意：不需要关闭原来的 MR，以上操作会自动更新 MR，reviewer 会根据新的修改决定是否允许合并。

### 🧹 删除分支

删除远程分支：

```bash
git push origin --delete fix-bug
```

执行后，GitLab 上的 `fix-bug` 分支会被移除。

如果你本地也有该分支，可一并删除：

```bash
git branch -d fix-bug        # 已合并可安全删除
git branch -D fix-bug        # 强制删除（未合并也可删）
```

参与开源项目，既是技术历练，也是心态修行。

阅读文档、遵循规范、沟通协作，这些看似繁琐的步骤，其实正是开源精神的体现。

也许第一次提交会让你焦虑，但当你的名字出现在项目的贡献者列表中，那种成就感，足以让一切努力都值得。
