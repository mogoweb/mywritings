# Python环境管理新利器：UV工具详解

Python 包和环境管理最好的工具无疑是 Anaconda，但我在之前的一篇文章《[注意，使用这款 Python 软件可能会带来麻烦](https://mp.weixin.qq.com/s/YHNmjP2wfOm_eSbDlXjFzQ)》写过，个人使用完全没有问题，如果在公司内使用，就需要格外小心，可能会招来官司。在我们公司，Anaconda（包括 MiniConda） 就是禁止安装的软件之一。

但是在工作中，确实又存在需要切换不同 Python 版本的需求，比如编译 Chromium 需要 Python 3.8 以上的版本，而打开 D-feet 软件又只限定只能使用 Python 3.7。所以我在公司都是使用《[龙芯 UOS 系统升级 Python](https://mp.weixin.qq.com/s/-75AGX9y_cSH35rG59M3ug)》这篇文章中介绍的方法，通过 update-alternatives 命令切换系统 Python 版本。但这种方法有一个缺点，经常需要切换来切换去，有时忘记切换回去，运行一些系统软件就会出现莫名奇妙的问题。比如有一次运行 **统信Windows应用兼容引擎**，始终提示 deepin-wine 未安装，但实际上系统上已经安装了 deepin-wine。后来经过排查，是由于一个用 python 写的服务未能启动，这个 python 应用需要一些 python 包，而我切到 Python 3.8，这个 Python 版本没有安装所需的包。

所以这段时间我也一直在寻找 Python 环境管理工具，终于给我找到了，就是这款由 Astral 团队开发的下一代 Python 环境管理工具：UV。UV 使用 Rust 语言编写，旨在替代传统包和虚拟环境管理工具（如 pip、virtualenv、pyenv 等），成为一站式解决方案。自 2024 年推出以来，其 GitHub 星标已突破 40k，成为 Python 社区最受关注的环境管理工具。

由于使用了 Rust 底层架构，带来了革命性提升，依赖解析速度比 pip 快 10-100 倍。例如安装 torch 等大型包时，耗时从传统工具的十几分钟缩短到几秒。

关键是它还整合了六大核心功能：

* 包管理（替代pip）
* 虚拟环境管理（替代virtualenv）
* Python版本控制（替代pyenv）
* 项目依赖锁定（替代poetry）
* 工具安装（替代pipx）
* 包发布（替代twine）

UV 支持创建项目级虚拟环境，通过 .venv 目录实现完全隔离，避免依赖冲突。它还完全兼容 pip 语法（如 uv pip install ），支持 pyproject.toml 配置规范，并能生成 requirements.txt 文件。如果你之前使用 pip 管理包，可以很快就上手。

## deepin | UOS 安装指南

通过官方脚本快速部署：

```bash
# 一键安装命令（支持Ubuntu 20.04+）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 验证安装（输出版本号即成功）
uv --version  # 示例输出：uv 0.6.8
```

安装过程自动配置环境变量，无需手动设置 PATH

## 核心使用场景
* 包管理
```bash
# 安装单个包（--system表示全局安装）
uv pip install numpy --system

# 批量安装依赖文件
uv pip install -r requirements.txt
```
实测安装 pandas 等大型库耗时仅传统工具的 1/10 。

* 虚拟环境管理
```bash
# 创建Python 3.12虚拟环境
uv venv --python 3.12 .venv

# 激活环境（Linux）
source .venv/bin/activate

# 安装环境专属依赖
uv pip install torch
```
环境文件仅占用约 30MB 空间，比 conda 轻量 80% 。

* 项目管理
```bash
# 初始化项目（自动生成配置）
uv init -p 3.11 my_project

# 添加依赖
cd my_project
uv add requests  # 自动更新pyproject.toml

# 生成锁定文件
uv pip compile pyproject.toml -o requirements.lock
```
项目结构包含标准化的 .python-version 和 uv.lock 文件，确保跨平台一致性。

* 脚本运行
```bash
# 为单文件脚本管理依赖
uv add --script demo.py pandas

# 在隔离环境中执行脚本
uv run demo.py
```
该特性特别适合快速原型开发。

* ​Python 版本切换
```bash
uv python install 3.10  # 安装指定版本
uv python use 3.10     # 切换当前环境版本
```

## 小结

作为 Python 生态的新基建，UV 通过 Rust 的极致性能与一站式功能设计，可以完全取代 pip、venv 等工具。其每秒处理数千个依赖的解析能力，结合友好的 CLI 交互，甚至可以挑战一下 Conda。如果你也在苦于无法在公司内使用 Anaconda，不妨试试这款工具。
