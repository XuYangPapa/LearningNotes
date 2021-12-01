# Linux文件查看

Linux系统中用于查看文件内容的命令主要有vim、cat、more、less、head、tail。

## 一、vim

vi/vim是绝大部分Unix Like操作系统内建的文本编辑器，除了可查看文件内容外，还具有强大且方便的文本/代码编辑能力。vim包含**命令模式（Command mode）**，**输入模式（Insert mode）**和**底线命令模式（Last line mode）**。

如果是在生产环境中查看日志或配置文件，建议使用`view`命令通过Read Only模式来查看文件。

vi/vim的更多操作在本笔记中暂时不展开。

## 二、cat

`cat`（英文全拼：concatenate）命令常用于显示整个文件内容，或者合并多个文件进行显示。

命令格式：`cat [options] filename`

Options：

* `-n`：对输出内容显示行号
* `-b`：和`-n`类似，但是空白行不编号

```bash
# 将file1的内容全部显示在显示屏
cat file1
# 将file1和file2的内容合并显示
cat file1 file2
# 将file1的内容加上行号后覆盖写入到file2
cat -n file1 > file2
# 将file1的内容加上行号（空白行不加）后追加写入到file2
cat -b file1 >> file2
# 清空file1度内容
cat /dev/null > file1
```

`cat`命令对应的还有一个`tac`命令，用于倒序显示。

**注意：`cat`将会一次性加载全部文件内容，在生产上若文件过大可能会出现卡死，需谨慎使用**

## 三、more

`more`命令和`cat`命令类似，也是用于显示文件内容，但是支持分页显示，并且支持更加丰富的命令参数和操作。

命令格式：`more [options] filename`

Options:

* `-p`：不以卷动的方式显示每一页，而是先清除萤幕后再显示内容
* `-num`：指定每页显示的行数
* `+num`：指定从num行开始显示
* `+/pattern`：在每个文档显示前搜寻该字串（pattern），然后从该字串前两行开始显示

```shell
# 将file1的内容分页显示
more file1
# 按顺序显示file1和file2(不支持文件切换)
more file1 file2
# 从第100行开始以每页50行显示file1
more +100 -50 file1
# 在file1中查找test并从查找到的位置往上两行开始显示
more +/test file1
```

进入`more`模式后可以进行操作：

* `空格`：向下翻页
* `b`：向上翻页
* `num 回车`：向下移动num行
* `=`：输出当前行行号
* `:f`：输出文件名和当前行的行号
* `v`：调用vi编辑器，退出vi后还在more模式中
* `!命令`：调用shell并执行命令
* `q`：退出more模式

可配合管道分页查看，eg：`ps -ef | more`

## 四、less

`less`命令的用法和`more`类似，但是相比`more`命令更加灵活，支持与`vi`类似的反向搜索，并且不会像`more`一样加载整个文件，适合在生产环境查看日志。

命令格式：`less [options] filename`

Options:

* `-m`：像`more`一样显示百分比
* `-M`：显示读取文件的百分比、行号及总行数
* `-N`：显示行号
* `-F`：如果文件内容只有一屏，则显示后退出less模式
* `-p pattern`：从搜素到的pattern位置开始显示
* `-I`：打开less后的搜索完全忽略大小写

进入`less`模式后，可以进行以下操作:

（注：使用`less`时，进入less模式后还可以输入上述options来启用相应功能，more则不行）：

* `空格`：向下滚动翻页
* `b`：向上滚动翻页
* `PageUp` or `PageDown`：向上或向下翻动一页
* `num 回车`：向下移动num行
* `=` or `:f`：显示当前行详细信息
* `v`：调用vi编辑器，退出vi后还在more模式中
* `\`：向下搜索
* `?`：向上搜索
* `n` or `N`：搜索下一个\上一个（和搜索方向有关）
* `& patttern`：仅显示匹配pattern的行
* `g` or `G`：移动到文件头或文件尾
* `F`：进入类似tail的模式，可以动态刷新文件内容
* `:e filename`：奇切换其他文件
* `:n` or `:p`：查看下一个或上一个文件
* `q`：退出less模式

可配合管道分页查看，eg：`ps -ef | less`

## 五、head

`head`用来显示文件前面几行内容，可以指定行数或字节数。

命令格式：`head [options] filename`

* `-v`：带文件名显示
* `-n num`：显示文件前num行
* `-c num`：显示文件前num个字节

## 六、tail

`tail` 用来显示文件的最后几行内容，当文件内容有更新时，tail可以自己主动刷新，确保一直显示最新的文件内容。

命令格式：`tail [options] filename`

* `-f`：动态刷新文件
* `-n num`：显示文件倒数num行
* `-c num`：显示文件倒数num个字节

## 七、总结

* `vi/vim`：文本处理能力最强。在生产上尽量使用`view`避免对生产上的文件产生污染；若文件过大时，尽量避免使用`vi\vim\view`
* `cat`：文本编辑或查看能力都比较一般，常用于复制文件内容重定向到另一文件
* `more`：相较于`cat`可以分页显示，可以在文件中向下查找，可编辑
* `less`：`vim\cat\more`均需一次读取全部文件，不适用于大文件查看，而`less`不会在一开始就读取整个文件，适合大文件；`less`支持更为丰富的命令，甚至可实现`tail -F`的功能
* `head`：仅关注文件开头部分时使用
* `tail`：希望动态监控文件内容时使用，譬如：各类监控日志



