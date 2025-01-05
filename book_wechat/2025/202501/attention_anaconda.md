# 注意，使用这款 Python 包管理器可能会带来麻烦

在上一篇文章《[龙芯 UOS 系统升级 Python](https://mp.weixin.qq.com/s/-75AGX9y_cSH35rG59M3ug)》中，介绍了 UOS 系统上多种 Python 版本共存并切换的方法。有朋友问，为什么不使用 Anaconda 这款工具呢？的确，在 AI 时代，Anaconda 的确是非常优秀的 Python 包管理工具。它不仅支持多种 Python 版本，还可以支持虚拟环境，避免不同项目之间 Python 包出现冲突。

没有使用 Anaconda 的原因之一是，Anaconda 并没有龙芯架构的安装包，另外一个原因是 Anaconda 是一款商业软件，个人可以随便用，但如果在公司使用，可能会带来麻烦。

Anaconda 作为商业产品，对大于 300 人的组织有收费要求。Anaconda 除了提供 Conda 和开源包管理器外，还包含一些商业支持服务、额外的工具和功能，例如数据科学和机器学习的高级包管理服务、企业支持、安全合规等。

如果我们选择 Miniconda ，能否绕开这个授权问题？

**Miniconda** 本身是免费的，没有人数限制，即使在人数大于 300 的组织中也可以免费使用。Miniconda 是 **BSD 3-Clause License** 授权下的开源软件，因此对于人数、使用场景（包括商业用途）没有任何限制。

但需要注意的是，如果 Miniconda 使用到  Anaconda 的包，也是需要收费的。而 Miniconda 默认使用 `defaults` 仓库，这是 Anaconda 提供的官方仓库。也就是说，如果你直接下载 Miniconda 而不做任何配置，那肯定会用到 Anaconda 中的商业包。如果在公司使用，就会违反协议，给公司带来不必要的麻烦。

如何确保 Miniconda 不使用 Anaconda 包的步骤呢？

### 1. 禁用 `defaults` 仓库

Miniconda 默认使用 `defaults` 仓库，这是 Anaconda 提供的官方仓库。为了确保你不从这个仓库下载包，可以显式禁用它。

* 编辑 Conda 的配置文件（`~/.condarc`）来禁用 `defaults` 仓库：

  ```yaml
  channels:
    - conda-forge
  default_channels: []
  ```

  上述设置将 `defaults` 仓库移除，并将 `conda-forge` 仓库设置为首选来源。

### 2. 使用 `conda-forge` 仓库

`conda-forge` 是一个社区维护的 Conda 包仓库，提供了许多常用的软件包。`conda-forge` 是完全开源的，没有商业限制。

* 添加 `conda-forge` 作为首选渠道：

  ```bash
  conda config --add channels conda-forge
  conda config --set channel_priority strict
  ```

  这会确保所有的包都优先从 `conda-forge` 获取，并避免使用 Anaconda 的 `defaults` 仓库。

### 3. 验证包来源

在安装包之前，可以通过检查包的来源，确保它们来自开源社区的仓库而非 Anaconda 的商业渠道。

* 安装软件包时可以指定来源：

  ```bash
  conda install numpy -c conda-forge
  ```

  这样会强制 Conda 从 `conda-forge` 安装 `numpy`，而不会使用 `defaults` 或其他 Anaconda 提供的渠道。
* 安装包后，使用 `conda list` 来检查包的来源：

  ```bash
  conda list
  ```

  在输出中，你会看到每个包的来源渠道。例如，`conda-forge` 是来自社区的包源，`defaults` 是来自 Anaconda 的包源。

### 4. 禁用自动激活 `base` 环境

默认情况下，Miniconda 会自动激活 `base` 环境，而这个环境可能使用了 Anaconda 提供的某些工具和包。为了避免这种情况，可以禁用自动激活，并只在必要时手动激活特定环境。

* 禁用自动激活 `base` 环境：
  ```bash
  conda config --set auto_activate_base false
  ```

### 5. 移除 Anaconda 自带的元包

如果你不小心安装了 Anaconda 自带的元包（例如 `anaconda`），它可能会将 Anaconda 提供的包源引入。你可以卸载它，并确保安装的包都是来自其他仓库。

* 卸载 Anaconda 元包：
  ```bash
  conda remove anaconda
  ```

### 6. 确保不使用 Anaconda Cloud

确保 `conda` 的配置中没有使用 Anaconda Cloud 上托管的私人或商业仓库。你可以通过以下命令查看已配置的仓库来源：

```bash
conda config --show channels
```

### 7. 手动管理环境

如果你完全希望避免使用 Anaconda 提供的包，可以手动管理 Conda 环境。创建一个新环境并明确指定从 `conda-forge` 安装包：

```bash
conda create -n myenv -c conda-forge python=3.9
conda activate myenv
```

你看，是不是不小心就会中招？所以很多公司明令禁止在公司内部使用 Anaconda 和 Miniconda。

如果你在工作中确实需要 Anaconda，也可以尝试说服老板，毕竟相对于开源版本，商业版本的包更加齐全，还提供服务。至于龙芯架构下，还是老老实实用 Python + venv 的模式吧。
