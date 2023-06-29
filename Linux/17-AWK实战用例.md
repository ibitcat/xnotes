[TOC]

## 匹配字符串
```
# 示例文本 test.txt 内容如下
[2023-02-28 12:00:00] a=1,b=2,c=xxxxx,d=4
[2023-02-28 12:00:00] a=1,b=2,c=xxxxx,d=4

# 匹配出字段 c 的字符串
awk '{match($0,/c=([^,]+)/,result); {print $1,$2 "---" result[1]}}' test.txt
```

## AWK将列用指定连接符拼接
示例文件 test.txt 内容如下:
```
AAAA 12345 xccvbn
BBBB 43431 fkodks
CCCC 51234 plafad
```

要获得 `AAAA,BBBB,CCCC`，即使用连接符 `,` 将每一行的第一列拼接起来。可以这样:
```bash
# 若不想输出的内容换行,则不加后面的 END{print ""}
# 方法1
awk '{printf "%s%s",sep,$1; sep=","} END{print ""}' test.txt

# 方法2
awk 'BEGIN{ORS=","}{print $1}' test.txt | sed 's/,$//'
```

## AWK中的分隔符
下面列出 `RS`、`ORS`、`FS`、`OFS` 的含义：
- `RS`，row separator，即**输入行分隔符**
- `ORS`，out row separator，即**输出行分隔符**
- `FS`，field separator，即**输入列分隔符**
- `OFS`，field separator，即**输出列分隔符**

行分隔符默认为 `\n`，即换行符；列分隔符默认为空格` `或制表符`\t`。

- 示例1：
    ```
    # 指定 输入行分隔符
    echo "111#222|333#444|555#666" | awk 'BEGIN{RS="|"} {print $0} END{print "---"}'

    # 输出
    111#222
    333#444
    555#666
            #注意，这里还有一个空行，因为 echo 命令输出的内容后面会带一个换行符
    ---
    ```

- 示例2：
    ```
    # 指定 行输入分隔符 和 行输出分隔符
    echo "111#222|333#444|555#666" | awk 'BEGIN{RS="|";ORS="$"} {print $0}'

    # 输出
    111#222$333#444$555#666
    ```

- 示例3：
    ```
    # 指定 输入行分割符 和 输入列分隔符
    echo "111#222|333#444|555#666" | awk 'BEGIN{RS="|";FS="#"} {print $1}'

    # 输出
    111
    333
    555
    ```

- 示例4：
    ```
    # 指定 输入行分割符、输入列分隔符 和 输出列分隔符
    echo "111#222|333#444|555#666" | awk 'BEGIN{RS="|";FS="#";OFS="-"} {print $1,$2}'

    # 输出
    111-222
    333-444
    555-666
     
    ```
