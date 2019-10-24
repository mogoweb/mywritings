
1. vscode + markdown all in one + markdown math + marp for vscode + plantuml。编辑和预览工具。
2. pandoc + Makefile + chrome-headless-render-pdf。编译系统，用于表现层 —— 任何我记录的内容可以被输出成 html，epub 和 pdf。
3. git + github。版本控制和存储系统。所有的内容我自己拥有，且随时随地可以在我的任何设备上访问。

### github做Markdown图床

以raw形式上传图片至github

1. 在github创建repository

2. clone至本地

3. 将需要上传的文件放入本地的文件夹

4. 利用github的windows客户端commit

5. 网页浏览复制图片URL，如

   ```script
   https://github.com/mogoweb/mywritings/blob/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png?raw=true
   ```

   在浏览器的输入框中粘帖上述地址，会得到如下真正的地址：

   ```script
   https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png
   ```

![images](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)