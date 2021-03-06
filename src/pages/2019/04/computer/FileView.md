# 查看文件
显示或查找文件里符合条件的字符串。

## CAT 命令
`cat` 命令可以用来显示文件中的内容。

### 命令格式

```bash
$ cat【选项】 【文件名】
```

### 常用选项介绍

|选项 | 描述 |
| :-- | :-- |
| -n 或 --number | 由 1 开始对所有输出的行数**编号**。 |
| -b 或 --number-nonblank | 和 -n 相似，只不过**对于空白行不编号**。 |
| -s 或 --squeeze-blank | 当遇到有连续两行以上的空白行，就代换为一行的空白行。 |

### 实例
把 `textfile1` 的文档内容加上行号后输入 `textfile2` 这个文档里：

```bash
$ cat -n textfile1 > textfile2
```

`cat` 也可以用来制作镜像文件。例如要制作软盘的镜像文件，将软盘放好后输入：

```bash
# OUTFILE 指输出的镜像文件名
$ cat /dev/fd0 > OUTFILE
```

## MORE
类似 `cat`，不过会以一页一页的形式显示，更方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 `b` 键就会往回（back）一页显示，按 `q` 退出。另外，按 `v` 调用 `vi` 编辑器，而且还有搜寻字串的功能。

### 语法

```bash
$ more [-dlfpcsu] [-num] [+/pattern] [+linenum] [fileNames..]
```

### 常用参数

|选项 | 描述 |
| :-- | :-- |
| -num | 一次显示 num 行。 |
| +num | 从第 num 行开始显示。 |
| -s | 当遇到有连续两行以上的空白行，就代换为一行的空白行。 |
| -f | 计算行数时，以实际上的行数，而非自动换行过后的行数（有些单行字数太长的会被扩展为两行或两行以上）。 |
| +/pattern | 在每个文档显示前搜寻该字串（pattern），然后从该字串之后开始显示。 |

### 实例
从第 20 行开始显示 `testfile` 之文档内容，一次显示 30 行。

```bash
$ more +20 -30 testfile
```

## LESS
`more` 仅能向前移动，却不能向后移动。使用 `less` 可以随意浏览文件，并且在查看之前不会加载整个文件。

### 语法

```bash
$ less [参数] 文件 
```

### 常用参数

|选项 | 描述 |
| :-- | :-- |
| -N | 显示每行的行号。 |
| -m | 显示类似 more 命令的百分比。 |
| -s | 显示连续空行为一行。 |
| -i | 忽略搜索时的大小写。 |
| -g | 只标志最后搜索的关键词。 |
| -f | 强迫打开特殊文件，例如外围设备代号、目录和二进制文件。 |

### 常用操作

|选项 | 描述 |
| :-- | :-- |
| /字符串 | 向下搜索"字符串"的功能。 |
| ?字符串 | 向上搜索"字符串"的功能。 |
| n | 重复前一个搜索（与 / 或 ? 有关）。 |
| N | 反向重复前一个搜索（与 / 或 ? 有关）。 |
| 回车键 | 滚动一行。 |
| y | 向前滚动一行。 |
| d | 向后翻半页。 |
| b | 向上翻一页。 |
| 空格键 | 向下翻一页。 |
| Q | 退出less 命令。 |

### 其他命令

|选项 | 描述 |
| :-- | :-- |
| v | 使用配置的编辑器编辑当前文件 |
| &pattern | 仅显示匹配模式的行，而不是整个文件 |
| ma | 使用 a 标记文本的当前位置 |
| 'a | 导航到标记 a 处 |

### 实例
查看 `log2019.log` 文件。

```bash
$ less log2019.log
```

## GREP
用于查找文件里符合条件的字符串。

`grep` 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 `grep` 指令会把含有范本样式的那一列显示出来。

若不指定任何文件名称，或是所给予的文件名为 "-"，则 `grep` 指令会从标准输入设备读取数据。

### 语法

```bash
$ grep [-abcEFGhHilLnqrsvVwxy][-A<显示列数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

### 常用参数

| 名称 | 描述 |
| :-- | :-- |
| -i 或 --ignore-case | 忽略字符大小写的差别。 |
| -c 或 --count | 计算符合样式的列数。 |
| -x --line-regexp | 只显示全列符合的列。 |
| -n 或 --line-number | 在显示符合样式的那一行之前，标示出该行的列数编号。 |
| -r 或 --recursive | 此参数的效果和指定"-d recurse"参数相同。 |
| -v 或 --revert-match | 显示不包含匹配文本的所有行。 |

### 实例
在当前目录中，查找后缀有 `file` 字样的文件中包含 `test` 字符串的文件，并打印出该字符串的行的内容。

```bash
$ grep test *file 
```

## 参考资料
 * [Linux 命令大全 | 菜鸟教程](http://www.runoob.com/linux/linux-command-manual.html)
 * [Linux达人养成计划 I](https://www.imooc.com/learn/175)