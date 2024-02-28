---
title: linuxOrder
date: 2023-09-12 10:25:56
tags: 
 - Linux常用命令
categories: 
 - Linux
---

## 1．Linux管理文件和目录的命令

| 命令  | 功能               |
| ----- | ------------------ |
| pwd   | 显当前目录         |
| ls    | 查看目录下的内容   |
| cd    | 改变所在目录       |
| cat   | 显示文件的内容     |
| grep  | 在文件中查找某字符 |
| cp    | 复制文件           |
| touch | 创建文件           |
| mv    | 移动文件           |
| rm    | 删除文件           |
| rmdir | 删除目录           |
| vi    | 编辑文件           |

### 1.1 pwd命令

该命令的英文解释为print working directory(打印工作目录)。输入pwd命令，Linux会输出当前目录。

### 1.2 cd命令 用来改变所在目录。

cd / 转到根目录中
		cd ~ 转到/home/user用户目录下
		cd /usr 转到根目录下的usr目录中-------------绝对路径
		cd test 转到当前目录下的test子目录中-------相对路径

### 1.3 ls命令 用来查看目录的内容。

选项 含义
		-a 列举目录中的全部文件，包括隐藏文件
		-l 列举目录中的细节，包括权限、所有者、组群、大小、创建日期、文件是否是链接等
		-f 列举的文件显示文件类型
		-r 逆向，从后向前地列举目录中内容
		-R 递归，该选项递归地列举当前目录下所有子目录内的内容
		-s 大小，按文件大小排序
		-h 以人类可读的方式显示文件的大小，如用K、M、G作单位

ls -l examples.doc 列举文件examples.doc的所有信息

### 1.4 cat命令

cat命令可以用来合并文件，也可以用来在屏幕上显示整个文件的内容。
		cat snow.txt 该命令显示文件snow.txt的内容，ctrl+D退出cat。

### 1.5 grep命令 最大功能是在一堆文件中查找一个特定的字符串。

grep money test.txt
		以上命令在test.txt中查找money这个字符串，grep查找是区分大小写的。

### 1.6 touch命令

touch命令用来创建新文件，他可以创建一个空白的文件，可以在其中添加文本和数据。
		touch newfile 该命令创建一个名为newfile的空白文件。

### 1.7 cp命令 用来拷贝文件，要复制文件，输入命令：

cp
		cp t.txt Document/t 该命令将把文件t.txt复制到Document目录下，并命名为t。
		选项 含义
		-I 互动：如果文件将覆盖目标中的文件，他会提示确认
		-r 递归：这个选项会复制整个目录树、子目录以及其他
		-v 详细：显示文件的复制进度

### 1.8 mv命令 用来移动文件。

选项 说明
		-I 互动：如果选择的文件会覆盖目标中的文件，他会提示确认
		-f 强制：它会超越互动模式，不提示地移动文件，属于很危险的选项
		-v 详细：显示文件的移动进度
		mv t.txt Document 把文件t.txt 移动到目录Document中。

### 1.9 rm命令 用来删除文件。

选项 说明
		-I 互动：提示确认删除
		-f 强制：代替互动模式，不提示确认删除
		-v 详细：显示文件的删除进度
		-r 递归：将删除某个目录以及其中所有的文件和子目录
		rm t.txt 该命令删除文件t.txt

### 1.10 rmdir命令 用来删除目录。

### 1.11 find命令  用来查询文件

**按照文件名搜索**

选项：
		-name    按照文件名搜索
		-iname   按照文件名搜索，不区分文件名大小写
		-inum    按照inode号搜索

如要在`/home/linuxize`目录下搜索一个名为`document.pdf`的文件，你可以使用下面的命令。

> find /home/linuxize -type f -name document.pdf

**按照文件类型搜索**

选项：
		-type d  查找目录
		-type f  查找普通文件
		-type l  查找软链接文件

例如，要在`/var/log/nginx`目录下找到所有以`.log.gz`结尾的文件，你可以使用一下命令

> ​	find /var/log/nginx -type f -name '*.log.gz'

**使用案例**

> find -name java*                     # 在当前目录下查找以java开始的文件
> 		find -name java* fprint file         # 在当前目录下查找以java开始的文件，并把结果输出到file中
> 		find -name ap* -o -name may*         # 查找以ap或may开头的文件

### 1.12 vi 是UNIX操作系统和类UNIX操作系统中最通用的全屏幕纯文本编辑器。Linux中的vi编辑器叫vim，它是vi的增强版（vi Improved），与vi编辑器完全兼容，而且实现了很多增强功能。进入vi的命令

vi filename :打开或新建文件,并将光标置于第一行首
		vi n filename ：打开文件,并将光标置于第n行首
		vi filename ：打开文件,并将光标置于一行首
		vi /pattern filename：打开文件,并将光标置于第一个与pattern匹配的串处
		vi -r filename ：在上次正用vi编辑时发生系统崩溃,恢复filename
		vi filename....filename ：打开多个文件,依次进行编辑

屏幕翻滚类命令
		Ctrl u：向文件首翻半屏
		Ctrl d：向文件尾翻半屏
		Ctrl f：向文件尾翻一屏
		Ctrl＋b；向文件首翻一屏
		nz：将第n行滚至屏幕顶部,不指定n时将当前行滚至屏幕顶部.
		插入文本类命令
		i ：在光标前
		I ：在当前行首
		a：光标后
		A：在当前行尾
		o：在当前行之下新开一行
		O：在当前行之上新开一行
		r：替换当前字符
		R：替换当前字符及其后的字符,直至按ESC键
		s：从当前光标位置处开始,以输入的文本替代指定数目的字符

保存命令
		按ESC键 跳到命令模式，然后：
		:w 保存文件但不退出vi
		:w file 将修改另外保存到file中，不退出vi
		:w! 强制保存，不推出vi
		:wq 保存文件并退出vi
		:wq! 强制保存文件，并退出vi
		:q 不保存文件，退出vi
		:q! 不保存文件，强制退出vi
		:e! 放弃所有修改，从上次保存文件开始再编辑