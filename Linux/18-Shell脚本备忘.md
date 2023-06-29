[TOC]

## bash中的<()语法
`<(list)` 是 bash、zsh、ksh的特性，称之为“进程替换”。参考官网文档[Process Substitution](https://www.gnu.org/software/bash/manual/bash.html#Process-Substitution)。可以想象括号内的命令最终会返回一个特殊的文件，有点类似 mkfifo 创建的命名管道。

一个[使用场景](Skynet/01-Skynet开启ssl.md#自签名证书)，在 openssl 生成证书时，因为一些原因需要对默认的配置进行一些扩展，但是又不想再创建一个配置文件。
此时，就可以使用进程替换，例如：
```
-config <(cat /etc/ssl/openssl.cnf <(printf '[SAN]\nsubjectAltName=DNS.1:www.xxx.com,DNS.2:*.xxx.com,IP.1:192.168.1.101'))
```
即在 /etc/ssl/openssl.cnf 配置文件后追加 SubjectAltName 选项内容，然后将进程替换后的特殊文件传递给 -config 参数。

推荐[Named pipes, process substitution and tee](https://kaushikghose.wordpress.com/2016/10/27/named-pipes-process-substitution-and-tee/)这篇博文，它介绍了如何使用命名管道、进程替换以及与 tee 的结合使用。

需要注意的是，若是在 shell 脚本文件中使用了进程替换，使用 `sh xxx.sh` 会报语法错误，即使在脚本文件定义了 `#!/bin/bash`。
原因是在以 `sh xxx.sh` 的方式执行脚本时，系统使用的 POSIX 的 shell。正确的运行方式有以下两种：
- 指定使用 bash 来运行，如：`bash xxx.sh`
- 在脚本文件指定 bash 运行（即文件第一行为`#!/bin/bash`），并给脚本赋予可执行权限，然后 `./xxx.sh` 来运行脚本

## 检查命令是否存在
不推荐使用 `which`，可以使用 `command`、`type`、`hash` 等。

例如检查 jq 是否已经安装，可以使用如下检查:
```bash
# command
if [[ -x "$(command -v jq)" ]]; then
    echo "jq exist"
else
    echo "jq dose not exist"
fi

# type
type jq > /dev/null 2>&1
if [ $? == 0 ]; then
    echo "jq exist"
else
    echo "jq dose not exist"
fi
```

## 文件表达式
- e filename，如果 filename存在，则为真
- d filename，如果 filename为目录，则为真 
- f filename，如果 filename为常规文件，则为真
- L filename，如果 filename为符号链接，则为真
- r filename，如果 filename可读，则为真 
- w filename，如果 filename可写，则为真 
- x filename，如果 filename可执行，则为真
- s filename，如果文件长度不为0，则为真
- h filename，如果文件是软链接，则为真
- filename1 -nt filename2 如果 filename1比 filename2新，则为真。
- filename1 -ot filename2 如果 filename1比 filename2旧，则为真。

```bash
# 判断文件是否存在
if [[ ! -e text.txt ]]; then
    echo "text not exist!"
fi
# 判断 /etc是否为目录
if [[ -d "/etc" ]]; then
    echo "etc folder exist!"
fi
```

## 字符串表达式
字符串允许使用赋值号做等号。
- `if  [ $a = $b ] `，如果string1等于string2，则为真
- `if  [ $string1 !=  $string2 ]`，如果string1不等于string2，则为真
- `if  [ -n $string  ]`，如果string 非空(非0），返回0(true)
- `if  [ -z $string  ]`，如果string 为空，则为真
- `if  [ $sting ]`，如果string 非空，返回0 (和-n类似)

## 数值表达式
```
-eq //equals等于
-ne //no equals不等于
-gt //greater than 大于
-lt //less than小于
-ge //greater equals大于等于
-le //less equals小于等于
```
- 在shell中进行比较时，结果为0代表真，为1代表假
- `-eq`，`-ne`等比较符只能用于数字比较，有字符也会先转换成数字然后进行比较

## 逻辑表达式
 - `if [ ! 表达式 ]`，逻辑非 !
 - `if [ 表达式1 –a  表达式2 ]`，逻辑与 -a
 - `if [ 表达式1 –o  表达式2 ]`，逻辑或 -o


## 单中括号和双中括号区别
参考:[Shell test 单中括号[] 双中括号[[]] 的区别](https://www.cnblogs.com/zeweiwu/p/5485711.html)
```bash
# bash 终端下执行下面的命令
type [ [[ test
# 输出
[ 和test 是 Shell 的内部命令，而[[是Shell的关键字。[ is a shell builtin
[[ is a shell keyword
test is a shell builtin
```
`[` 和 `test` 是 Shell 的内部命令，而 `[[` 是Shell的关键字。



## 字符串变量处理
参考:[shell 字符串变量](https://blog.csdn.net/qq_34125999/article/details/125439015)
```bash
#!/bin/bash
string="hello world"

echo "字符串为: ${string}"
echo "字符串长度为: ${#string}"
echo '使用单引号打印 string: ${string}'
echo "使用双引号打印 string: ${string}"
echo  不使用引号打印 string: ${string}
echo "从左侧第0个开始,向左截取2个字符: ${string:0:2}"
echo "从左侧第5个开始,向左截取所有字符: ${string:5}"
echo "从右侧第5个开始,向右截取2个字符: ${string:0-5:2}"
echo "截取左边第一次出现字符l右边的所有字符: ${string#*l}"
echo "截取左边最后一次出现字符l右边的所有字符: ${string##*l}"
echo "截取右边第一次次出现字符l左边的所有字符: ${string%l*}"
echo "截取右边最后一次出现字符l左边的所有字符: ${string%%l*}"
```
输出：
```
字符串为: hello world
字符串长度为: 11
使用单引号打印 string: ${string}
使用双引号打印 string: hello world
不使用引号打印 string: hello world
从左侧第0个开始,向左截取2个字符: he
从左侧第5个开始,向左截取所有字符:  world
从右侧第5个开始,向右截取2个字符: wo
截取左边第一次出现字符l右边的所有字符: lo world
截取左边最后一次出现字符l右边的所有字符: d
截取右边第一次次出现字符l左边的所有字符: hello wor
截取右边最后一次出现字符l左边的所有字符: he
```

## %和%%的作用
用于截取字符串变量，支持通配符。
```bash
var="aaabbccbbxxxx"
echo ${var%bb*}  #输出 aaabbcc
echo ${var%%bb*} #输出 aaa
```
- `%`，表示从尾部最近的匹配删除
- `%%`，表示从尾部最远的匹配删除


## 获取当前执行的shell脚本路径
假设脚本在 /home/domi/test.sh。
```bash
#/bin/bash

echo "${BASH_SOURCE[0]}"
echo "${BASH_SOURCE}"
dir="$(dirname "${BASH_SOURCE[0]}")"
shelldir=$(cd $dir && pwd)
echo $dir
echo $shelldir
```

在任意目录执行 `sh /home/domi/test.sh`，输出如下：
```
/home/domi/test.sh
/home/domi/test.sh
/home/domi
/home/domi
```

在 `/home/domi` 执行 `sh test.sh`，输出如下：
```
test.sh
test.sh
.
/home/domi
```

## $符号的作用
- `$0`	    ，当前脚本的文件名
- `$1`到`$9`，传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 $1，第二个参数是 $2
- `$?`	    ，显示最后命令的执行情况
- `$#`	    ，传递到脚本的参数个数
- `$$`	    ，当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID
- `$*`	    ，以一个单字符串形式所有向脚本或函数传递的参数
- `$@`      ，传递给脚本或函数的所有参数
- `$!`	    ，后台运行的最后一个进程的 ID 号
- `$-`	    ，显示 Shell 使用的当前选项

参考 [Shell特殊变量](http://c.biancheng.net/view/806.html)。
**注意：** `$@` 和 `$*` 在不被双引号`" "`包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔，但是在被双引号包围时，`$*` 则会以一整行字符串的形式传递参数。

```bash
#!/bin/bash
echo "print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done

echo "print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```
执行 `sh test.sh a b c d`，输出结果如下：
```
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d
```


## for循环参数
```bash
argc=$#
argv=("$@")

for (( j=0; j<argc; j++ )); do
    echo "${argv[j]}"
done
```

## 分割字符串
```bash
myvar="string1 string2 string3"
myarray=($myvar)
echo "$myvar"
echo "${myarray[2]}"
echo "${#myvar[@]}"
echo "${#myarray[@]}"
for s in ${myvar[@]}; do
    echo "substr = $s"
done
```
输出：
```
string1 string2 string3
string3
1
3
substr = string1
substr = string2
substr = string3
```
- 参考[Bash split string into array using 4 simple methods](https://www.golinuxcloud.com/bash-split-string-into-array-linux/)
- 参考[Bash Loop Through a List of Strings](https://linuxhint.com/bash_loop_list_strings/)

## read提示字符串加颜色
```bash
red=$'\e[31m'
nocolor=$'\e[0m'
while true;do
    read -p "${red}是否确认?[Y/N]:${nocolor}" Input
	case $Input in
	[yY]|[yY][eE][sS])
		break
		;;
	[nN]|[nN][oO])
		exit 1
		;;
	*)
		continue
		;;
	esac
done
```

## shell参数截取
参数或数组截取形式为：`${parameter:offset:length}`，如果参数为`@`，则为截取可定位参数数组，从offset开始算，length为取的长度，如果不给，则取到最后。如果offset为负，则为从最后一位倒数，如offset为-1则为最后一个元素。如果offset和length计算得到的长度小于0则会报错。
```bash
#!/bin/bash
#推荐
function foo(){
    echo "${@:1}"
    echo "${@:1:2}"
    echo "${@:2:0}"
    echo "${@:0}"
    echo "${@: -1}" #注意负号前面要有一个空格
    echo "${@: -1:2}"
}
foo a b c d

#另一种方式
function bar(){
    args=("$@")
    unset args[0]
    unset args[1]
    echo ${args[@]}
}
bar x y z
```

## eval的用法
eval用来在执行命令时作二次解析：主要是每次执行一个shell命令它会先检察一次，看到有$标志就会把值替换一次，然后再执行一遍。eval不会唤起起另一个shell来执行，而是在本身这个shell内多解析一次，所以替换的结果可以保留下来使用。
```bash
#!/bin/bash
foo=10
x=foo
y='$'$x
echo $y #输出: $foo

#eval
eval y='$'$x
echo $y #输出：10
echo $(eval echo '$'$x) #输出：10
```

## shell参数解析
一般简单参数使用手工解析；**短选项**则推荐使用`getopts`，它是bash内置函数；**长选项**使用`getopt`，它是独立可执行程序。
一般用法如下：
```bash
#!/bin/bash
echo "$@"
while getopts ":a:bc:" opt; do #不打印错误信息, -a -c需要参数 -b 不需要传参 
  case $opt in
    a)
      echo "-a arg:$OPTARG index:$OPTIND" #$OPTIND指的下一个选项的index
      ;;
    b)
      echo "-b arg:$OPTARG index:$OPTIND"
      ;;
    c) 
      echo "-c arg:$OPTARG index:$OPTIND"
      ;;
    :)
      echo "Option -$OPTARG requires an argument." 
      exit 1
      ;;
    ?) #当有不认识的选项的时候arg为?
      echo "Invalid option: -$OPTARG index:$OPTIND"
      ;;
  esac
done
```

参考:
- [shell - 参数解析三种方式(手工, getopts, getopt)](https://www.cnblogs.com/kevingrace/p/11753294.html)
- [Shell脚本中的while getopts用法小结](https://www.cnblogs.com/kevingrace/p/11753294.html)