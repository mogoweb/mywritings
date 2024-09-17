# 在 Deepin 系统中搭建 Node.js 开发环境

Node.js 是基于 Chrome V8 JavaScript 引擎的运行时环境。它使得 JavaScript 不再仅限于前端，而可以扩展到后端开发，从而在传统由 C/C++、Java、Go 等语言主导的领域中占据一席之地。JavaScript 语言本身简洁易用，加上 Node.js 提供的大量模块，使得开发者能够快速构建和部署应用程序。如今，Node.js 在开发可扩展的网络应用和实时应用方面，已经得到了广泛的应用和认可。

在 Deepin 系统中搭建 Node.js 环境非常简单，可以直接使用 apt 命令进行安装：

```bash
$ sudo apt install nodejs
```

在 Deepin V23 版本中，系统自带的 Node.js 版本为 18.19.1。

```bash
$ node -v
v18.19.1
```

如果项目中需要使用更新的版本，可以通过以下方式进行升级。以下示例使用最新的 LTS 版本 V20.17.0 进行说明。

首先，安装 nvm，这是一个命令行工具，用于管理和切换不同版本的 Node.js 环境。通过 nvm，你可以轻松地安装多个版本的 Node.js，并在它们之间切换，无需手动删除和重新安装。

```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

脚本下载并安装 nvm，完成后会提示你关闭并重新打开终端以使 nvm 生效，或者可以直接在当前终端中运行以下命令立即生效：

```bash
export NVM_DIR="$HOME/.config/nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # 加载 nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # 加载 nvm 的 bash 补全功能
```

我们选择前一种方式，关闭当前终端窗口，然后打开一个新的终端窗口，此时 nvm 应已生效。你可以使用以下命令安装 Node.js 的最新 LTS 版本：

```bash
$ nvm install 20
```

安装完成后，确认是否已切换到新版本：

```bash
$ node -v
v20.17.0
$ npm -v
10.8.2
```

你还可以列出系统中已安装的 Node.js 版本：

```bash
$ nvm ls
->     v20.17.0
system
default -> 20 (-> v20.17.0)
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v20.17.0) (default)
stable -> 20.17 (-> v20.17.0) (default)
lts/* -> lts/iron (-> v20.17.0)
```

若需要切换回系统自带的版本，可以使用以下命令：

```bash
$ nvm use system
```

此时，Node.js 版本将切换回系统自带的 v18.19.1：

```bash
$ node -v
v18.19.1
```

再切换回 20 版本：

```bash
$ nvm use 20.17.0
$ node -v
v20.17.0
```

通过以上步骤，你已经在 Deepin 系统中成功搭建了 Node.js 开发环境，是不是非常简单？通过 nvm 不仅可以灵活切换版本，也确保了项目能够使用最新的稳定版本。希望本文能帮助你更好地使用 Deepin 系统进行 Node.js 开发，享受高效、简洁的编程体验。
