在 普通模式下，输入 `:` 进入命令模式。字符串替换命令格式如下：
```
:s/要替换的字符串/替换字符串/
```

可以使用其他分割符，例如使用 `#` 作为分隔符，下面的命令也是合法的。
```
:s#要替换的字符串#替换字符串#
```

## 测试环境
若有以下内容的文本文件：
```
aaa aaa /aa/bb/cc a\b\c bbb
ccc ccc /aa/bb/cc a\b\c ddd
eee eee /aa/bb/cc a\b\c fff
```

## 单次替换
替换第一行的 `aaa` 为 `xxx`，首先将光标移动到第一行，然后输入命令：
```
# 命令
:s/aaa/xxx/

# 结果
xxx aaa /aa/bb/cc a\b\c bbb
ccc ccc /aa/bb/cc a\b\c ddd
eee eee /aa/bb/cc a\b\c fff
```
**注意**：这样只会替换光标所在行的第一个匹配到的内容。

## 转义替换
有一些特殊字符在替换时需要进行转义，如 `.`、`\` 需要转义成 `\.`、`\\`。
替换第一行的 `a\b\c` 为 `a/b/c`，可以这样：
```
#命令
:s#a\\b\\c#a/b/c#

#结果
aaa aaa /aa/bb/cc a/b/c bbb
ccc ccc /aa/bb/cc a\b\c ddd
eee eee /aa/bb/cc a\b\c fff
```

## 整行替换
替换第一行的所有的 `aaa` 为 `xyz`，命令类似单次替换，只需要在后面加上`/g`。
首先将光标移动到第一行，然后输入命令：
```
# 命令
:s/aaa/xyz/g

# 结果
xyz xyz /aa/bb/cc a\b\c bbb
ccc ccc /aa/bb/cc a\b\c ddd
eee eee /aa/bb/cc a\b\c fff
```

## 多行替换
在 `s` 前面加入形如`n,m`的表达式，可以控制指定范围内的行的字符串替换。
1. `n` 表示从第 n 行开始
2. `m` 表示到第 m 行结束(包括第m行)，当 m 为 `$` 时，表示文件的最后一行

- 替换第1、2两行的 `/aa/bb/cc` 为 `/x/y/z`
    ```
    # 命令
    :1,2s#/aa/bb/cc#/x/y/z

    # 结果
    aaa aaa /x/y/z a\b\c bbb
    ccc ccc /x/y/z a\b\c ddd
    eee eee /aa/bb/cc a\b\c fff
    ```
- 替换第1行到最后行的 `/aa/bb/cc` 为 `hello`
    ```
    # 命令
    :1,$s#/aa/bb/cc#hello#

    # 结果
    aaa aaa hello a\b\c bbb
    ccc ccc hello a\b\c ddd
    eee eee hello a\b\c fff
    ```

- 替换第1、2两行中所有的 `a` 为 `A`（即在后面加上`g`）
    ```
    # 命令
    :1,2s/a/A/g

    # 结果
    AAA AAA /AA/bb/cc A\b\c bbb
    ccc ccc /AA/bb/cc A\b\c ddd
    eee eee /aa/bb/cc a\b\c fff
    ```

## 全文替换
替换全文的 `/` 为 `@`，可以这样：
```
# 命令
:%s#/#@#g

# 结果
aaa aaa @aa@bb@cc a\b\c bbb
ccc ccc @aa@bb@cc a\b\c ddd
eee eee @aa@bb@cc a\b\c fff
```

`%s` 等价于 `1,$s`。

## 正则替换
个人很少使用，参考[这篇文章](https://blog.csdn.net/RockHill_001/article/details/115460172)。