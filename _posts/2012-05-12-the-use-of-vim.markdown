---
layout: post
title: "vim的使用和个人设置"
date: 2012-05-12 21:50
comments: true
categories: [Ubuntu,Vim]
---

进入与离开
----------
要"进入"VIM可以直接在系统提示下键入"VIM 文件名称"，VIM可以自动帮你载入所要编辑的文件或是开启一个新文件。进入VIM后屏幕左方会出现波浪符号，凡是行首有该符号就代表此列目前是空的。要离开VIM可以在指令模式下键入":q"，,":wq"指令则是存档後再离开r(注意冒号r)。要切换到指令模式下则是用"ESC"键，如果不晓得现在是处於什麽模式，可以多按几次"ESC"，系统会发出哔哔声以确定进入指令模式。

<!-- more -->

档案指令
--------
档案指令多以 : 开头，跟编辑指令有点区别。

:q 结束编辑(quit)
> 如果不想存档而要放弃编 辑过的档案则用 :q! 强制离开。

:w 存档(write)
> 其后可加所要存档的档名。
> 可以将档案指令合在一起，例如 :wq 即存档后离开。
> zz 功能与 :wq 相同。
> 另外 值得一提的是 VIM 的部份存档功能。可以用 :n,mw filename 将第 n 行到第 m 行的文字存放的所指定的 filename 里去哩。

## 1. 查找
    /xxx(?xxx)
    表示在整篇文档中搜索匹配xxx的字符串, / 表示向下查找, ? 表示向上查找.
    其中xxx可以是正规表达式,关于正规式就不多说了.一般来说是区分大小写的, 
    要想不区分大小写, 那得先输入":set ignorecase"查找到以后, 再输入'n'查找下一个匹配处，输入'N'反方向查找。

    *(#)
    当光标停留在某个单词上时, 输入这条命令表示查找与该单词匹配的下(上)一个单词. 
    同样, 再输入'n'查找下一个匹配处, 输入'N'反方向查找。

    g*(g#)
    此命令与上条命令相似, 只不过它不完全匹配光标所在处的单词, 而是匹配包含该单词的所有字符串。

    gd
    本命令查找与光标所在单词相匹配的单词, 并将光标停留在文档的非注释段中第一次出现这个单词的地方。

    %
    本命令查找与光标所在处相匹配的反括号, 包括 () [] { }

    f(F)x
    本命令表示在光标所在行进行查找, 查找光标右(左)方第一个x字符。找到后;输入';'表示继续往下找输入','表示反方向查找

## 2. 快速移动光标
在 vi 中, 移动光标和编辑是两件事, 正因为区分开来, 所以可以很方便的进行光标定位和编辑. 因此能更快一点移动光标是很有用的.

    w(e)  移动光标到下一个单词.
    b     移动光标到上一个单词.
    0     移动光标到本行最开头.
    ^     移动光标到本行最开头的字符处.
    $     移动光标到本行结尾处.
    H     移动光标到屏幕的首行.
    M     移动光标到屏幕的中间一行.
    L     移动光标到屏幕的尾行.
    gg    移动光标到文档首行.
    G     移动光标到文档尾行.
    c-f   (即 ctrl 键与 f 键一同按下) 本命令即 page down.
    c-b   (即 ctrl 键与 b 键一同按下, 后同) 本命令即 page up.
    '     此命令相当有用, 它移动光标到上一个标记处, 比如用 gd, \* 等查;找到某个单词后, 再输入此命令则回到上次停留的位置.
    '.    此命令相当好使, 它移动光标到上一次的修改行.
    `.    此命令相当强大, 它移动光标到上一次的修改点.

## 3. 拷贝, 删除与粘贴
在 vi 中 y 表示拷贝, d 表示删除, p 表示粘贴. 其中拷贝与删除是与光标移动命令结合的。

    yw    表示拷贝从当前光标到光标所在单词结尾的内容.
    dw    表示删除从当前光标到光标所在单词结尾的内容.
    y0    表示拷贝从当前光标到光标所在行首的内容.
    d0    表示删除从当前光标到光标所在行首的内容.
    y$    表示拷贝从当前光标到光标所在行尾的内容.
    d$    表示删除从当前光标到光标所在行尾的内容.
    yfa   表示拷贝从当前光标到光标后面的第一个a字符之间的内容.
    dfa   表示删除从当前光标到光标后面的第一个a字符之间的内容.
    yy    表示拷贝光标所在行.
    dd    表示删除光标所在行.
    D     表示删除从当前光标到光标所在行尾的内容.

## 4. 数字与命令
在 vi 中数字与命令结合往往表示重复进行此命令, 若在扩展模式的开头出现则表示行号定位. 如:

    5fx     表示查找光标后第 5 个 x 字符.
    5w(e)   移动光标到下五个单词.
    5yy     表示拷贝光标以下 5 行.
    5dd     表示删除光标以下 5 行.
    y2fa    表示拷贝从当前光标到光标后面的第二个a字符之间的内容.
    :12,24y 表示拷贝第12行到第24行之间的内容.
    :12,y   表示拷贝第12行到光标所在行之间的内容.
    :,24y   表示拷贝光标所在行到第24行之间的内容. 删除类似.

## 5. 快速输入字符
在 vi 中, 不要求你输入每一个字符, 可以有很多种方法快速输入一些字符.
使用 linux/unix 的同学一定有一个经验, 在命令行下输入命令时敲入头几个字符再按TAB 系统就会自动将剩下的字符补齐, 假如有多个匹配则会打印出来. 这就是著名的命令补齐(其实windows中也有文件名补齐功能). vi 中有许多的字符串补齐命令, 非常方便.

在编辑模式中, 输入几个字符后再输入此命令则 vi 开始向上(下)搜索开头与其匹配的单词并补齐, 不断输入此命令则循环查找. 此命令会在所有在这个 vim 程序中打开的文件中进行匹配.

    c-x-l
    在编辑模式中, 此命令快速补齐整行内容, 但是仅在本窗口中出现的文档中进行匹配.

    c-x-f
    在编辑模式中, 这个命令表示补齐文件名. 如输入:
      /usr/local/tom 后再输入此命令则它会自动匹配出:
      /usr/local/tomcat/

    abbr
    即缩写. 这是一个宏操作, 可以在编辑模式中用一个缩写代替另一个字符串. 
    比如编写java文件的常常输入 System.out.println, 这很是麻烦, 所以应该用缩写来减少敲字. 
    可以这么做:":abbr sprt System.out.println"以后在输入sprt后再输入其他非字母符号, 它就会自动扩展为System.out.println

## 6. 替换
替换是 vi 的强项, 因为可以用正规表达式来匹配字符串.以下提供几个例子.

    :s/aa/bb/g          将光标所在行出现的所有包含 aa 的字符串中的 aa 替换为 bb
    :s/<aa>/bb/g        将光标所在行出现的所有 aa 替换为 bb, 仅替换 aa 这个单词
    :%s/aa/bb/g         将文档中出现的所有包含 aa 的字符串中的 aa 替换为 bb
    :12,23s/aa/bb/g     将从12行到23行中出现的所有包含 aa 的字符串中的 aa 替换为 bb
    :12,23s/^/#/        将从12行到23行的行首加入 # 字符
    :%s= *$==           将所有行尾多余的空格删除
    :g/^s*$/d          将所有不包含字符(空格也不包含)的空行删除.

## 7. 多文件编辑
在一个 vim 程序中打开很多文件进行编辑是挺方便的.

> :sp(:vsp) 文件名
> > vim 将分割出一个横(纵)向窗口, 并在该窗口中打开新文件.从 vim6.0 开始, 文件名可以是一个目录的名称, 这样, vim 会把该目录打开并显示文件列表, 在文件名上按回车则在本窗口打开该文件, 若输入'O'则在新窗口中打开该文件, 输入'?'可以看到帮助信息.

> :e 文件名
> > vim 将在原窗口中打开新的文件, 若旧文件编辑过, 会要求保存.

> c-w-w
> > vim 分割了好几个窗口怎么办? 输入此命令可以将光标循环定位到各个窗口之中.

> :ls
> > 此命令查看本 vim 程序已经打开了多少个文件, 在屏幕的最下方会显示出如下数据:
         1   %a      "usevim.html"         行 162
         2   #       "xxxxxx.html"         行 0
         其中:
         1               表示打开的文件序号, 这个序号很有用处.
         %a              表示文件代号, % 表示当前编辑的文件,
         #               表示上次编辑的文件
         "usevim.html"   表示文件名.
         行 162          表示光标位置.

> :b 序号(代号)
> > 此命令将指定序号(代号)的文件在本窗口打开, 其中的序号(代号)就是用 :ls 命令看到的.

> :set diff
> > 此命令用于比较两个文件, 可以用":vsp filename"命令打开另一个文件, 然后在每个文件窗口中输入此命令,就能看到效果了.

## 8. 宏替换
vi 不仅可以用 abbr 来替换文字, 也可以进行命令的宏定义. 有些命令输起来很费劲,因此我把它们定义到 <F1>-<F12> 上, 这样就很方便了.这些配置可以预先写到 ~/.vimrc(windows 下为 $VIM/_vimrc) 中, 写进去的时候不用写前面的冒号.

    :nmap <F2> :nohls<cr>       取消被搜索字串的高亮
    :nmap <F9> <C-W>w           命令模式下转移光标到不同窗口
    :imap <F9> <ESC><F9>        输入模式下运行<F9>
    :nmap <F12> :%s= *$==<cr>   删除所有行尾多余的空格.
    :imap <F12> <ESC><F12>      同上

## 9. TAB
TAB 就是制表符, 单独拿出来做一节是因为这个东西确实很有用.

    <<        输入此命令则光标所在行向左移动一个 tab.
    >>        输入此命令则光标所在行向右移动一个 tab.
    5>>       输入此命令则光标后 5 行向右移动一个 tab.
    :12,24>   此命令将12行到14行的数据都向右移动一个 tab.
    :12,24>>  此命令将12行到14行的数据都向右移动两个 tab.

那么如何定义 tab 的大小呢? 有人愿意使用 8 个空格位, 有人用4个, 有的用2个.有的人希望 tab 完全用空格代替, 也有的人希望 tab 就是 tab. 没关系, vim 能帮助你.以下的设置一般也都先写入配置文件中, 免得老敲.
     :set shiftwidth=4   设置自动缩进 4 个空格, 当然要设自动缩进先.
     :set sts=4          即设置 softtabstop 为 4. 输入 tab 后就跳了 4 格.
     :set tabstop=4      实际的 tab 即为 4 个空格, 而不是缺省的 8 个.
     :set expandtab      在输入 tab 后, vim 用恰当的空格来填充这个 tab.

----
## vi配置
https://github.com/wongyouth/vimfiles

A handful of plugins for vim all maintained in one bundle subdirectory, useful vim configuration, espacially for Rails coding. All plugins are included as submodules, so you can get plugins updated in one command that makes life easier.

One Line Installation:
    bash < <(curl -s https://raw.github.com/wongyouth/vimfiles/master/install.sh)

Old School Installation:
    # Checkout configuration files
    git clone git://github.com/wongyouth/vimfiles ~/vimfiles

    # Create symlinks
    ln -s ~/vimfiles ~/.vim
    echo "source ~/.vim/vimrc" > ~/.vimrc

    # Switch to the `~/.vim` directory, and fetch submodules
    cd ~/.vim
    git submodule init
    git submodule update

## Usage
     :Helptags for build vim plugin doc
     F7        for NERDTree toggle
     F4        for paste toggle
     CTRL-B    for showing BufExplorer
     \cc       for comment out
     \c<SPACE> for removing comment mark
     \a=       for tabular =
     \a:       for tabular :
     \a>       for tabular =>
     CTRL-y,   for zencoding
     yss-      for <% -%>
     yss=      for <%= %>

## 其他

---
* :1R user.rb或:find user.rb 快速文件切换
* :Rfunctionaltest 在controller中可以直接跳转至对应的spec文件
* :Rcontroller        在spec文件中跳转至对应的controller文件
* ：RVfunctionaltest 将spec文件和controller文件多屏显示（横屏），竖屏维RS
* 当光标停留在某个函数上时使用gf可直接跳转至函数定义的文件中
* Ctrl + v   选择多个字符
* Shift + v  选择多行
* \a&gt;          对含有“=&gt;”的表达式自动对齐
* e!            重新载入文件
* ysiW)       对光标停留的单词加括号“（）”
* ：sp 文件名       将该文件在一个新的窗口中打开
* ：Ggrep 字段名        在当前文件夹下递归搜索含有该字段的文件 ：cnext 下一个，：copen 打开所有进行选择
* 在一段文字（若干行）前插入文字或空格，（1）Ctrl + v;  (2)shift + i (在最前面插入)
* :%s= \*$==      将所有行尾多余的空格删除
* :g/^s*$/d        将所有不包含字符（空格也没有）的空行删除
* :%s/old/new/g 全文替换将old替换为new（:%s/old/new/gc 代表“confirm”每次替换要求确认）
* :n1,n2s/old/new/g   在n1行到n2行范围内进行替换
* :s/old/new/g             替换当前行的所有匹配
* zz 到屏幕中央

