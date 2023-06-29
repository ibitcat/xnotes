## 正则判断操作符
在 bash 中，操作符 `=~` 表示正则匹配判断操作符，参考官方文档 [Bash Reference Manual](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Conditional-Constructs)。
```
When you use ‘=~’, the string to the right of the operator is considered a POSIX extended regular expression pattern and matched accordingly (using the POSIX regcomp and regexec interfaces usually described in regex(3)).
```
它遵循 POSIX ERE，即 POSIX 扩展正则表达式。

## POSIX 正则表达式
POSIX 定义了以下两种正则表达式：
- BRE(Basic Regular Expression)，标准正则表达式
- ERE(Extended Regular Expression)，扩展正则表达式

linux 命令 `grep`、`sed` 都遵循 POSIX 正则表达式，模式使用标准正则，加`-E`参数则使用扩展正则。

**注意**：两种正则对于数字的匹配都没有支持 `\d`，可以用 `[0-9]` 来替代。

参考：
- [POSIX正则表达式 | BRE和ERE](https://www.cnblogs.com/wyzersblog/p/13709354.html)
- [正则表达式 - POSIX & PCRE](https://zhuanlan.zhihu.com/p/435815082)

## 示例
- 示例一
```bash
commit_msg=`cat $1`
commit_msg_pattern="^(revert: )?(feat|fix|docs|style|refactor|perf|test|workflow|build|ci|chore|types|wip|dep)(\(.+\))?: .{1,50}"
if [[ $commit_msg =~ $commit_msg_pattern ]]; then
    echo "The regex matches!"
    echo ${BASH_REMATCH[0]}
    echo ${BASH_REMATCH[1]}
    echo ${BASH_REMATCH[2]}
    echo ${BASH_REMATCH[3]}
fi
```
上面是一个匹配[Angular提交信息规范](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)([中文指南](https://zj-git-guide.readthedocs.io/zh_CN/latest/message/Angular%E6%8F%90%E4%BA%A4%E4%BF%A1%E6%81%AF%E8%A7%84%E8%8C%83/))提交消息hooks的正则判断。

- 示例二
```bash
if [[ "abcyyy13554221547HelloxxxWorld" =~ yyy([0-9]{11})(Hello)xxx(.*) ]]; then
    echo "The regex matches!"
    echo $BASH_REMATCH
    echo ${BASH_REMATCH[1]}
    echo ${BASH_REMATCH[2]}
    echo ${BASH_REMATCH[3]}
fi
```