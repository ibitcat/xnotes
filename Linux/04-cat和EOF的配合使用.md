## 写入或追加文件

输入格式如下：

```
# 写入(覆盖原来的内容)
cat > 文件名 << EOF
#input something
EOF
```

或者

```
# 追加
cat >> 文件名 << EOF
#input something
EOF
```

用来创建文件，在这之后输入的任何东西，都会写入或追加到文件里，输入完成之后以 EOF 结尾代表结束。需要注意的是，`EOF`在这里没有特殊的含义，你可以使用 FOE 或 OOO 等（当然也不限制在三个字符或大写字符）。

> 注意，不支持在 fish shell 中执行，应该是 fish 不兼容这种输入语法。

-   示例 1（使用 EOF 作为输入结束符）：

```
cat > test.txt <<EOF
aaaa
bbbb
cccc
EOF
```

查看文件内容，显示如下：

```
domi@surface-book2:~$ cat test.txt
aaaa
bbbb
cccc
```

-   示例 2（使用 ab 作为输入结束符）：

```
cat > test1.txt <<ab
aaaa1
bbbb2
cccc3
ab
```

查看文件内容，显示如下：

```
domi@surface-book2:~$ cat test1.txt
aaaa1
bbbb2
cccc3
```

## 多行字符串变量

```bash
var=$(cat <<- EOF
this is line 1
this is line 2
EOF
)

# 注意：一定要用 "" 包起来，否则输出会变成一行
echo "$var"
```

## 忽略制表符

在 `<<` 后面添加 `-` ，即`<<-` 可以将每一行头部的制表符都删除（**只能是制表符，空格不行**），这在 shell 脚本的 if 代码块中也能使用 cat 追加内容。例如：

```bash
if true; then
    cat <<-EOF
    a
    EOF
fi
```

## 禁止变量展开

在 EOF 前面加 `\` 可以禁止变量的展开，例如：

```bash
FOO="bar"

cat << \EOT > foobar.txt
echo $FOO
EOT

# 输出为：echo $FOO
# 若不加\，则输出为：echo bar
```

## 认知误区

cat 不是只能用来看文件，使用 man cat 命令，查看官方对 cat 的描述：将[文件]或标准输入(即键盘输入)，输出到标准输出。

```
Concatenate FILE(s) to standard output.

With no FILE, or when FILE is -, read standard input.
```

不加选项参数，直接使用 cat，就是直接将标准输入(即键盘输入)输出到标准输出，如下所示：

```
domi@surface-book2:~$ cat
a
a
b
b


^C
```

输入 `a`，cat 会立即输出 `a`。
