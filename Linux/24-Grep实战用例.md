## 截断 grep 返回的长匹配行

当使用 grep 返回的匹配行过长，影响关键数据的查询，可以对匹配行进行截断，有两种方法：

-   方法一，使用 `-E` 参数，配合 `-o`参数。

```
#表示截取匹配到的行的关键词 BattleReport 前后10个字符，而不显示整行
grep -oE '.{0,10}BattleReport.{10}' xxx.log
```

截取方式入下图（图片引用字 stackoverflow）：
![grep truncate](../images/20220909-01-greptruncate.png)

-   方法二，使用 `-P` 参数，即使用 perl 正则匹配，配合 `-o` 来达到截取效果。

```
grep -oP '.*(?=BattleReport)' xxx.log
```

perl 的四种表达式：

-   `(?=...)`：表示从左向右的顺序环视。例如(?=\d)表示当前字符的右边是一个数字时就满足条件
-   `(?!...)`：表示顺序环视的取反。如(?!\d)表示当前字符的右边不是一个数字时就满足条件
-   `(?<=...)`：表示从右向左的逆序环视。例如(?<=\d)表示当前字符的左边是一个数字时就满足条件
-   `(?<!...)`：表示逆序环视的取反。如(?<!\d)表示当前字符的左边不是一个数字时就满足条件

示例文件 test.log 内容如下：

```
email1=aaa@163.com
email2=bbb@qq.com
```

匹配输出结果：

```
grep -oP '.*(?=@163\.com)' test.log
# 输出：email1=aaa

grep -oP '(?=aaa).*' test.log
# 输出：aaa@163.com

grep -oP '.*(?<=aaa)' test.log
# 输出：email1=aaa

grep -oP '(?<=\=)(.+?)(?=@\d+\.com)'
# 输出：aaa
```

更多示例：

```
# test.log 内容如下
# op=send,cmd=xxx,args={"id":1,"name":"cat"},response=false
# op=call,cmd=xxx,args={"id":1,"name":"cat"},response=false
# 取出日志中 args 的 json
grep -oP '(?<=args=)(.*)(?=,response)' test.log
```

> 小知识点，perl 默认使用贪婪匹配(greedy match)，若要使用懒惰匹配(lazy match)，则在 \*，+，？等表示匹配次数的后面加上？就表示以懒惰模式进行匹配。

参考：
[How to truncate long matching lines returned by grep or ack](https://stackoverflow.com/questions/2034799/how-to-truncate-long-matching-lines-returned-by-grep-or-ack)、[grep 与 perl 正则 AWK](https://www.jianshu.com/p/dd5b97f9385a)、[linux grep 命令的-P 和-o 选项的作用](https://www.cnblogs.com/onelikeone/p/16415452.html)

## 查找指定目录指定后缀的文件

下面的命令可以查找 `src` 目录下所有 `.lua` 文件中，包含 `class("xxx")` 的字符串，并输出匹配到的字符串。

```bash
grep -rh --include=\*.lua -oP "(?<=class\(\")(.*)(?=\")" /data/apps/server/src
```

-   `-r`：递归查找
-   `-h`：不显示文件名
-   `-o`：只显示匹配到的字符串
-   `-P`：使用 perl 正则
-   `--include=\*.lua`：只查找 `.lua` 文件
-   `(?<=class\(\")(.*)(?=\")`：使用 `(?<=)` 和 `(?=)` 来表示环视，匹配 `class("` 和 `")` 之间的字符串。
