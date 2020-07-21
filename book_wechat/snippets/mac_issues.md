今天升级macOS High Sierra，终端里使用git的时候，弹出一行莫名其妙的错误：

> xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun

解决方法，重装xcode command line：

```bash
xcode-select --install
```

如果没有解决问题，执行以下命令

```bash
sudo xcode-select -switch /
```

