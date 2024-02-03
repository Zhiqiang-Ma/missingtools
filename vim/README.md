### 1，光标移动

在vim命令行模式下光标移动方法

* gg 移动到文件头部
* G 移动到文件尾部
* n- 向上跳n行
* n+ 向下跳n行
* Enter 移动到上一行首

* 打开文件，即跳转到第n行。适用于debug时候定位错误行。

~~~shell
vim +n filename
~~~

打开文件时候匹配特定格式

~~~shell
vim +/pattern filename：打开文件，并将光标置于第一个与pattern匹配的串处
~~~

### 2，yank 复制

在vim中, 除了在编辑模式下修改文件，命令模式的时候可以删除和复制。yank（提起

* yy 复制当前行。
* nyy 复制当前和向下n行
* y^ 复制当前到行头的内容
* y$  复制当前到行尾的内容
* yG  复制至文档尾

### 3，delete 剪切 

* d   剪切选定块到缓冲区
* dd 剪切整行
* ndd向下删除当前行在内的n行
* d^ 剪切到行首
* d$ 剪切到行尾
* dG 剪切至文档尾
* x 删除当前字符
* nx 删除从光标开始的n个字符

### 4，paste 粘贴

* p 粘贴至光标后，字符后面，（当前行下）
* P 粘贴在光标的前，字符前面，（当前行上）

### 5，search 搜索

在命令行模式下

* / 斜杠 

  ~~~ shell
  / keyword 向光标下搜索keyword字符串
  ~~~

* ? 问好

  ~~~shell
  ?keyword   //向光标上搜索keyword字符串
  ~~~

* \* 和#

  当光标停留在某个单词上，表示查找与该单词匹配的下上一个单词。

  n      //向下搜索前一个搜索的单词

  N      //向上搜索前一个搜索的单词

ps：g*(g#)    //此命令与上条命令相似, 只不过它不完全匹配光标所在处的单词, 而是匹配包含该单词的所有字符串

### 6，sub替换

* 当前行替换

  ~~~shell
  :s/old/new        //用new替换行中首次出现的old
  ~~~

* 全局替换

  ~~~
  :s/old/new/g     //用new替换行中所有的old
  ~~~

* 指定范围替换

  ~~~
  :n,m s/old/new/g   //用new替换从n到m行里所有的old
  ~~~

* 文件中所有的

  ~~~
  :%s/old/new/g      //用new替换当前文件里所有的old
  ~~~

* 交互式替换

  ~~~
  :%s/old/new/gc
  ~~~

  可以选择 y 和n 

* 快速注释和取消注释

  * 注释步骤
    * 按 `ESC`进入命令行模式
    * `ctrl+v`进入块选择模式；
    * 操作 `k`和 `j`进行上下行选择；即上下键
    * 按大写 `I`进入插入模式，输入注释符 `//`或 `#`(shell脚本注释符)；
    * 最后按下 `ESC`即可完成注释。
  * 取消注释
    * 同上三步骤，第四步时候，按d 键删除注释，然后ESC 退出即可。

### 8，vim 中文显示乱码

方式一：

在制作容器的时候加上参数

~~~shell
-e LANG="C.UTF-8"
~~~

方式二：

在 /etc/vim/vimrc 或是 ~/.vimrc 中添加以下命令

~~~shell
set termencoding=utf-8
set encoding=utf8
set fileencodings=utf8,ucs-bom,gbk,cp936,gb2312,gb18030
~~~

解析：

* termencoding 是所有终端设置的编码方式
* encoding 是vim 内部的编码方式
* fileencodings 是文件打开的编码方式，启动时候会按照列表的顺序进行打开

### 7，大小写转换

* 整篇文章大写转化为小写

> 打开文件后，无须进入命令行模式。
>
> 键入:ggguG

> 解释一下：ggguG分作三段gg gu G
>
> gg=光标到文件第一个字符
>
> gu=把选定范围全部小写
>
> G=到文件结束

* 只转化某个单词

> guw 、gue、gUw、gUe
>
> 这样，光标后面的单词便会进行大小写转换
>
> 想转换5个单词的命令如下：
>
> gu5w、gu5e、gU5w、gU5e

* 转换几行的大小写

> 将光标定位到想转换的行上，键入：1gU
>
> 从光标所在行往下一行都进行小写到大写的转换
>
> 10gU，则进行11行小写到大写的转换
>
> 以此类推，就出现其他的大小写转换命令
>
> gU0 ：从光标所在位置到行首，都变为大写
>
> gU$ ：从光标所在位置到行尾，都变为大写
>
> gUG ：从光标所在位置到文章最后一个字符，都变为大写
>
> gU1G ：从光标所在位置到文章第一个字符，都变为大写

注意输入命令后，回车执行。

### 9，vim 中显示隐藏符号

在vim 中输入以下命令显示所有的隐藏符号。

（在使用grep 查找需要的sentence 时候，又特殊的字符可能失效）

~~~~shell
:set list
:set nolist
~~~~

其中的 $ 表示结束符号，^I 为tab 制表符号。使用以下的命令可以删除

~~~
:%s/\t//g 		#删除所有的制表符号
~~~

替换掉^M

~~~
:%s/\r//g
~~~

同理：如果出现^V则用:%s/^V//g替换

### 10，设置table 空格

~~~shell
set tabstop=4       " Tab键替换的空格长度，默认8
set softtabstop=4   " 退格键退回缩进空格的长度
~~~

写代码时，按换行键，如果需要让代码自动缩进一个Tab，可以增加以下配置

~~~shell
set autoindent      " 设置自动缩进
~~~

### 11，自动设置行号和改变配色方案

在文件 /etc/vim/vimrc 后追加两行

~~~bash
echo "colorscheme industry" >> /etc/vim/vimrc
echo "set nu" >> /etc/vim/vimrc
~~~

 可以在 vim的命令模式下，输入colorscheme + tabel 看下自己喜欢那种配色

### 12，同时滚动分屏幕

需要在两个vim 窗口中同时的设置 

~~~shell
set scrollbind
~~~

* “.” 是重复上一个命令，

* “jj.” 下移动两次并重复上个命令

### 13，vimdiff 比较不同的文件

~~~shell
vimdiff file1 file2
~~~

## 