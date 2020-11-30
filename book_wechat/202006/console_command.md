# 掌握基本的命令行，迈向 Linux 第一步

当年我接触计算机时，微软的 Windows 系统还没有出现，所使用的操作系统是一张 5.25 吋软盘（容量 1.2MB）就能装下的 DOS 系统，自然没什么图形界面。所以，我是从学习各种命令，开始进入计算机世界的。拜技术的飞速发展所赐，Windows 很快取代 DOS，人们再也不用记那些繁杂的计算机命令，通过点点鼠标就可以操作电脑。Windows系统其实也带了一个命令行程序（包括最新的 Windows 10 系统），估计绝大多数人从来都没有打开过。

然而，在这个世界上，Windows 系统并没有主宰整个电脑世界，特别是对于程序员群体来说，Linux 系统和苹果的 Mac OS 系统也得到广泛使用。特别是人工智能、系统开发、云计算等领域，Linux 系统下环境的配置反而更简单。我经常会建议程序员尝试用 Linux 系统作为软件开发环境。现代 Linux 系统其实也有着非常友好的 GUI（图形用户界面），甚至有的 Linux 发行版本借鉴了 Mac OS，有着非常炫酷的用户界面。但对于程序员而言，不断改进并提高生产力是第一要务，这个时候使用命令行反而更加高效。

说起来记住各种命令，然后通过简陋的控制台用户接口输入命令，似乎有点反人性。但这是 Linux 系统的精髓。其实 Mac OS 虽然最早拥抱 GUI，但它也有强大的命令行系统。Windows 系统似乎也意识到这个需求，最近发布了一款全新的命令行应用程序： **Windows Terminal**。有数据表明，相对于使用鼠标在 GUI 中点击，在终端中完成完全相同的工作，速度更快。江湖中一直有传说，一流的程序员只使用 Vim 或 Emacs，编程完全不用鼠标，只听见啪啪啪的键盘声。我们也不要把自己当作大神，只用记一些简单的命令，为日常工作提升一点效率。下面我就总结一下最基础的 Linux 命令，助你进入 Linux 世界。

#### 1. pwd

注意，这个命令并不是 password 的缩写，而是 **print working directory** 的缩写。pwd命令输出当前工作目录的完整系统路径。

```bash
alex@alex-MS-7C22:~$ pwd
/home/alex
```

这个命令如此简单，也是最常用的命令之一，通常情况下我们并不需要传递命令行参数。如果你要查看完整的命令行参数，请输入：

```bash
alex@alex-MS-7C22:~$ man pwd
```

**注：**对后面的命令也是如此，如果你要了解更详细的命令用法，都可以使用 **man + <命令>** 查看帮助。

#### 2. cd

这也是最常用的命令之一。cd 是 **change directory** 的缩写，顾名思义，此命令用来更改当前工作目录。

```bash
alex@alex-MS-7C22:~$ cd Downloads/
alex@alex-MS-7C22:~/Downloads$
```

当然我们也可以进入到其它的工作目录，而且可以进入多个层级里面。

在以下示例中，我们将当前目录切换至 **Pictures** 文件夹中的 **myimages** 子文件夹。

```bash
alex@alex-MS-7C22:~/Downloads$ cd ~/Pictures/myimages
alex@alex-MS-7C22:~/Pictures/myimages$
```

在目录操作中，有几个特定的符号代表某个目录。

* **~** : 用户主目录，通常为 **/home/<username>**，比如在第一个命令示例中，pwd显示 **~** 的全路径为 **/home/alex**
* **..** : 父目录，比如执行如下命令，将当前目录切换到父目录的父目录。

```bash
alex@alex-MS-7C22:~/Pictures/myimages$ cd ../..
alex@alex-MS-7C22:~$
```

* **.** : 当前目录。

#### 3. ls

ls 是 **list** 的缩写，此命令列出目录中的所有文件。您可以指定要列出文件的目录。如果未指定目录，则使用当前工作目录。

```bash
alex@alex-MS-7C22:~/Pictures$ ls
myimages  Selection_001.png  Selection_008.png  Selection_009.png  Selection_010.png  tmp
```

该命令有几个非常有用的选项。例如 **-a** 选项，可以列出隐藏文件。**-l** 选项则可以显示更多的文件信息，其中包含文件大小和权限等。更酷的是，这些选项可以组合起来使用：

```bash
alex@alex-MS-7C22:~/Pictures$ ls -la
total 244
drwxr-xr-x  4 alex alex  4096 5月  11 09:31 .
drwxr-xr-x 65 alex alex  4096 5月  30 06:35 ..
drwxrwxr-x  2 alex alex 24576 12月 11 09:26 myimages
-rw-rw-r--  1 alex alex 30180 11月 11  2019 Selection_001.png
-rw-rw-r--  1 alex alex 64302 11月 20  2019 Selection_008.png
-rw-rw-r--  1 alex alex 92592 11月 25  2019 Selection_009.png
-rw-rw-r--  1 alex alex 19035 11月 25  2019 Selection_010.png
drwxrwxr-x  2 alex alex  4096 1月   2 10:08 tmp
```

#### 4. cp 和 mv

cp 命令是 **copy** 的缩写，用来复制文件和目录。第一个参数（文件或目录）是源，第二个参数是目标。 在以下示例中，我们将图像复制到 **Downloads** 文件夹。

```bash
alex@alex-MS-7C22:~/Pictures$ cp Selection_001.png ~/Downloads/
```

如果是复制目录，请加上 **-r** 选项进行递归复制。

复制文件和目录时有很多选项。例如，我们可以将具有特定扩展名的所有文件复制到目录中。以下示例将所有带有 png 扩展名的文件复制到 **Downloads** 文件夹。

```bash
alex@alex-MS-7C22:~/Pictures$ cp *.png ~/Downloads/
```

mv 命令是 move 的缩写，与 cp 命令的工作原理类似，只是用于移动（而不是复制）文件和目录。需要注意的是，在移动目录及子目录时，不带有 **-r** 选项。要想浏览可用于 mv 命令的所有选项，只需键入：

```bash
man mv
```

#### 5. mkdir 和 touch

mkdir 是 **make directory** 的缩写，该命令用于创建目录。 此命令需要一个参数：新目录的名称。要验证命令是否成功执行，可以使用 ls 命令查看。

```bash
alex@alex-MS-7C22:~/Pictures$ mkdir my-new-folder
alex@alex-MS-7C22:~/Pictures$ ls
myimages  my-new-folder  Selection_001.png  Selection_008.png  Selection_009.png  Selection_010.png  tmp
```

创建文件与创建目录一样容易。这个时候需要用到 touch 命令来创建一个新文件。

需要注意，所创建的文件为空。如果要验证命令是否已成功执行，请使用 ls 命令。

```bash
alex@alex-MS-7C22:~/Pictures$ touch myfile.txt
alex@alex-MS-7C22:~/Pictures$ ls -la
total 248
drwxr-xr-x  5 alex alex  4096 6月   3 15:24 .
drwxr-xr-x 65 alex alex  4096 5月  30 06:35 ..
-rw-r--r--  1 alex alex     0 6月   3 15:24 myfile.txt
drwxrwxr-x  2 alex alex 24576 12月 11 09:26 myimages
drwxr-xr-x  2 alex alex  4096 6月   3 15:23 my-new-folder
-rw-rw-r--  1 alex alex 30180 11月 11  2019 Selection_001.png
-rw-rw-r--  1 alex alex 64302 11月 20  2019 Selection_008.png
-rw-rw-r--  1 alex alex 92592 11月 25  2019 Selection_009.png
-rw-rw-r--  1 alex alex 19035 11月 25  2019 Selection_010.png
drwxrwxr-x  2 alex alex  4096 1月   2 10:08 tmp
```

#### 6. rmdir 和 rm

就像有单独的用于创建文件和目录的命令，删除文件和目录也有两个单独的命令。

要删除目录，可以使用 rmdir 命令，该命令是 **remove directory** 的缩写。但是，此命令只能删除空目录。

```bash
alex@alex-MS-7C22:~/Pictures$ rmdir my-new-folder
```

rm 命令更强大。您可能已经猜到它是 **remove** 的简写。 rm 命令删除指定的每个文件。

```bash
alex@alex-MS-7C22:~/Pictures$ rm Selection_001.png
```

它也可以用来删除目录，但需要带 **-r** 选项，它将递归删除匹配的目录、它们的子目录以及它们包含的所有文件。

```bash
alex@alex-MS-7C22:~/Pictures$ ls tmp/
IMG_20191205_084245-01.jpeg  IMG_20191211_081050-01.jpeg  IMG_20191222_082026-01.jpeg  IMG_20200101_090356-01.jpeg  mmexport1573967495194-01.jpeg
IMG_20191208_084017-01.jpeg  IMG_20191212_215942-01.jpeg  IMG_20191231_212811-01.jpeg  IMG_20200101_091823-01.jpeg  mmexport1573967502382_mr1573967801990-01.jpeg
IMG_20191208_084409-01.jpeg  IMG_20191213_203823-01.jpeg  IMG_20191231_213325-01.jpeg  IMG_20200101_092057-01.jpeg  mmexport1576987906954-01.jpeg
IMG_20191208_173142-01.jpeg  IMG_20191213_204007-01.jpeg  IMG_20191231_215049-01.jpeg  IMG_20200101_092254-01.jpeg
alex@alex-MS-7C22:~/Pictures$ rm -r tmp/
```

为了忽略不存在的文件并且在删除之前永远不会提示您，可以使用 **-f** 选项。

#### 7. cat， tail 和 head

在读取文件内容时，有几种选择。第一个选项是cat命令（concatenate的缩写），其主要作用是显示文件的内容。

```bash
alex@alex-MS-7C22:~/Pictures$ cat myfile.txt 
hello world!
```

注意，cat命令会显示整个文件的内容。如果文件内容特别多，可能只需要显示文件的前 n 行或最后 n 行，这时可用上 tail 和 head 命令。tail 命令输出文件的最后 10 行，而 head 命令输出文件的前 10 行。

```bash
alex@alex-MS-7C22:~/Pictures$ tail helloworld.txt 
hello world 4!
hello world 5!
hello world 6!
hello world 7!
hello world 8!
hello world 9!
hello world 10!
hello world 11!
hello world 12!
hello world 13!
```

也可以使用 -n 选项指定要输出的行数。

```bash
alex@alex-MS-7C22:~/Pictures$ head -5 helloworld.txt 
hello world 1!
hello world 2!
hello world 3!
hello world 4!
hello world 5!
```

#### 8. grep

grep 命令是 **global regular expression print** 的缩写，用于搜索文本。它将在文件中检索指定的字符串，并以特定的格式显示结果。

这个命令非常强大，前提是你要懂得正则表达式。如果不懂得也没有关系，可以先从基本的查找开始。

假定我们有一个包含所有国家/地区名称的文件。我们要检查 Netherlands 一词是否在该文件中。请注意，默认情况下，grep区分大小写。

我们传递的第一个参数是要查找的单词，第二个参数是我们要搜索的文件。

```bash
alex@alex-MS-7C22:~/Pictures$ grep Netherlands countries.txt
Netherlands
```

如果不想区分大小写，可以使用 **-i** 选项。这样，无论您要查找的是 BeL、bel还是 BEL，都一样对待。

```bash
alex@alex-MS-7C22:~/Pictures$ grep -i BeL countries.txt
Belarus
Belgium
Belize
```

注意，在上面的示例中，我们看到 grep 将整个匹配行输出到终端。使用 **-c** 选项可以打印匹配的行数。

```bash
alex@alex-MS-7C22:~/Pictures$ grep -ic BeL countries.txt
3
```

#### 9. find

find 命令用于快速查找文件或目录。假设我们需要找到当前目录中的所有CSS文件，就可以使用 find 命令执行此操作。

```bash
alex@alex-MS-7C22:~/Pictures$ find . -name "*.css"
./style.css
```

请注意，find 命令还会搜索所有子目录。

### 小结

命令行往往是 Windows 程序员转向 Linux 的一个拦路虎，其实只要掌握了一些基础的命令，使用起来并不是那么难。况且现在 Linux 的 GUI 已经非常易用。Linux 系统唯一的缺点是有很多娱乐、游戏方面的应用程序没有开发 Linux 版本，转过来想，没有那些 app，我们是不是更能专注于软件开发上面呢？

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/common_images/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7_%E5%85%B3%E6%B3%A8%E4%BA%8C%E7%BB%B4%E7%A0%81.png)
