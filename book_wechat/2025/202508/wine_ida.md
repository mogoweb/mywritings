# Wine 应用调试的重器：IDA pro

在上一篇文章《介绍几款 Windows 应用调试的小工具》中介绍了在调试 Wine 应用中的一些小工具，但在面临复杂问题时，就需要特别的手段和工具。比如在分析闭源软件时，有的时候程序并没有崩溃或者明显的报错，但程序逻辑并没有按照预期进行，或者程序发生了崩溃，但崩溃得比较深，也许是其它模块没有按照预想的逻辑运行，而导致本模块的崩溃。在没有源代码的情况下，要分析应用程序的逻辑，这个时候就需要搬出逆向工程工具来。在逆向工程领域，IDA Pro（Interactive Disassembler Professional）是最重要的工具之一。

IDA Pro（Interactive Disassembler Professional）被誉为逆向工程领域的瑞士军刀，是分析恶意软件、研究软件漏洞和理解闭源软件内部逻辑的终极工具，也是破解软件的必要工具之一。我们的工作和破解、分析软件漏洞没有关系，主要是用来写着理解软件的内部逻辑。

下面以分析一个简单的 "CrackMe" 程序为例，介绍 IDA Pro 软件和使用。

首先，我编写了一个简单的 easy_crackme.exe 的程序。运行时，它会提示“请输入密码”，输入错误的密码会提示“密码错误！”，效果如下：

```bash
$ deepin-wine8-stable easy_crackme.exe 
wine version: 8.16
Please input password: hh
mistake!
```

接下来，打开 IDA Rro 8.0。IDA Pro 有 32 位版本和 64 位版本，根据需要分析的软件是 32 位还是 64 位选择对应的版本。将 easy_crackme.exe 拖入IDA Pro，使用默认设置进行分析。可以看到如下界面：

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_01.png)

默认界面包含几个核心窗口：

1. IDA View (反汇编窗口)：这是最主要的窗口，以汇编代码的形式展示程序逻辑。
   
  默认视图是图形视图，将函数内的代码块（以跳转指令分割）可视化为流程图。绿色的边代表条件跳转为真，红色的边代表为假，蓝色的边代表无条件跳转。这对于理解复杂的if-else和循环结构极为有用。
  
  ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_02.png)

  在该视图下，按空格键可以切换到文本视图。此视图线性地展示代码，左侧是地址，右侧是汇编指令和注释。

  ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_03.png)

  要切回图形视图，再按一次空格键即可。

2. Hex View (十六进制窗口)：以十六进制和ASCII码的形式展示文件的原始字节数据，与反汇编窗口同步。
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_04.png)

3. Structures (结构窗口)：以树状结构展示结构体各成员的​​名称、偏移量（Offset）、数据类型（如 db、dw、dd）及大小​​。
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_05.png)

4. Enums（枚举窗口）：用于管理和显示二进制文件中枚举类型（enum）定义的视图。它通过将数字常量转换为有意义的符号名称，显著提升反汇编代码的可读性。
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_06.png)

5. Imports/Exports Window (导入/导出窗口)：分别显示程序调用了哪些外部库函数（Imports）以及它向外提供了哪些函数（Exports）。通过导入表可以快速了解程序的大致功能（如网络、文件、加密等）。
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_07.png)

6. Functions Window (函数窗口)：位于最左侧，列出IDA分析识别出的所有函数。可以快速跳转到任何一个函数。非常有用的函数通常有main, start或包含关键API调用的函数。
   
   ![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_08.png)

在默认设置中没有显示的窗口视图，我个人觉得比较重要的还有**Strings Window (字符串窗口)**，在这里，IDA会自动提取程序中所有的可疑字符串。这是分析的绝佳起点，因为错误信息、文件名、URL等关键线索常常在这里暴露。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_09.png)

从这个字符串窗口，我们一下子发现可疑线索:

```
.rdata:0040A044	00000018	C	Please input password: 
.rdata:0040A05E	00000018	C	I am the real password!
.rdata:0040A076	00000013	C	correct! welcome!\n
.rdata:0040A089	0000000A	C	mistake!\n
```

再运行一下 easy_crasckme.exe 程序：

```
$ deepin-wine8-stable easy_crackme.exe 
wine version: 8.16
Please input password: I am the real password!
correct! welcome!
```

当然，由于这个程序过于简单，所以猜测起来就比较简单。但是我们在实际项目中，还是不少人会在代码中写入连接数据库的字符串，这其中可能就包括用户名和密码。如果是明文，那就更容易被分析出来。还有不少程序员为了方便，将密钥写在代码中，且不说密钥泄露后只能通过升级程序来解决，更为可怕的是很容易被黑客通过上面简单的方法就可以分析出来。

回到实际项目中，我们不能指望程序员会将密码写到程序中，但是我们基本可以确定，如果密码输入正确后，会提示 "correct! welcome!"。因为这个是最终用户会看到的文本提示，双击该字符串，IDA会跳转到它在数据段中的地址。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_10.png)

选中该字符串所在的行，按下快捷键 X。IDA会弹出一个对话框，显示所有引用了这个字符串地址的代码位置。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_11.png)

点击确定，IDA会立即带我们跳转到使用该字符串的关键代码处。

![](https://raw.githubusercontent.com/mogoweb/mywritings/master/book_wechat/2025/202508/images/wine_ida_12.png)

这里可以看到一个典型的条件判断结构：一个代码块进行某些比较，然后根据结果，分别走向打印“密码正确”和“密码错误”的分支。

在这个函数内，按下 F5 键。IDA的反编译器会将复杂的汇编代码转换成类似C的伪代码，逻辑瞬间清晰：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *v3; // eax
  char Buffer[100]; // [esp+1Ch] [ebp-68h] BYREF

  __main();
  rpl_printf("Please input password: ");
  v3 = ___acrt_iob_func(0);
  if ( fgets(Buffer, 100, v3) )
    Buffer[strcspn(Buffer, "\n")] = 0;
  if ( !strcmp(Buffer, "I am the real password!") )
    rpl_printf("correct! welcome!\n");
  else
    rpl_printf("mistake!\n");
  return 0;
}
```

我们甚至可以将上述伪代码，交给 AI，写出对应的 C 语言代码：

```c
#include <stdio.h>
#include <string.h>  // 提供字符串操作函数（strcspn, strcmp）

int main(int argc, char *argv[]) 
{
    char userInput[100];  // 存储用户输入的缓冲区（原伪代码中的Buffer）
    
    // 初始化环境（原伪代码中的__main，通常编译器自动处理）
    
    // 提示用户输入
    printf("Please input password: ");
    fflush(stdout);  // 确保提示信息立即显示
    
    // 从标准输入读取用户输入（原伪代码中的fgets+___acrt_iob_func(0)）
    if (fgets(userInput, sizeof(userInput), stdin)) 
    {
        // 移除输入末尾的换行符（fgets会保留换行符）
        userInput[strcspn(userInput, "\n")] = '\0';
    }
    
    // 验证密码（原伪代码中的硬编码字符串比较）
    const char *correctPassword = "I am the real password!";
    if (strcmp(userInput, correctPassword) == 0) 
    {
        printf("correct! welcome!\n");
    } 
    else 
    {
        printf("mistake!\n");
    }
    
    return 0;
}
```

* 从伪代码中，我们可以一目了然地看到，程序的核心逻辑就是使用strcmp函数，将用户的输入 (userInput) 与一个固定的字符串 "I am the real password!" 进行比较。
* 当且仅当两个字符串完全相同时，strcmp返回0，程序才会执行打印“密码正确”的逻辑。
* 因此，我们成功找到了正确的密码：I am the real password!

IDA Pro的功能远不止于此，它还包括强大的调试器、脚本支持（IDAPython）、类型库和插件系统等高级功能。刚开始使用时主要用到了如下核心操作与快捷键，能够极大的提高分析的效率：

* G (Go)：跳转到指定的地址或函数名。
* X (Cross-References)：查看一个函数、变量或字符串在何处被引用。这是最重要的功能之一，可以帮您构建起代码的调用关系。
* N (Rename)：对函数、变量、地址等进行重命名。好的命名是成功的一半，例如，您可以将sub_401000重命名为ValidatePassword。
* ; (分号)：添加行注释。记录您的分析思路。
* : (冒号)：添加可重复的注释，当同一段代码在多处出现时，这个注释也会同步出现。
* Y (Yank/Set Type)：更改变量或函数的类型。例如，将一个DWORD指针的类型声明为char*，IDA就会自动将其引用的数据解析为字符串。
* F5 (Decompiler)：此键可将当前函数的汇编代码一键反编译成更易读的C伪代码。这是IDA的“杀手级”功能。

对于复杂的应用来说，可能会进行多天的连续分析，这个时候加入注释、为函数和变量重命名为一个用户友好的名字，无疑可以方便后续的理解，分析的结果可以保存起来，后续再加载。

## 小结

上面的示例展现了我们平常分析 wine 应用问题的经典流程：

1. 信息收集：通过字符串等静态特征寻找突破口。
2. 关联分析：利用交叉引用 (X) 找到关键特征在何处被使用。
3. 逻辑理解：借助图形视图和反编译器 (F5) 理解程序的判断逻辑和算法。
4. 得出结论：找到问题的答案。

实际的分析过程当然比这复杂得多，限于篇幅原因，这里就不继续展开，在后续的文章中，我将继续分析一些调试技巧。欢迎关注。