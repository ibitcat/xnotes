- `-`，一般表示标准输出
- `--`，双短横线前面的都是选项（以及有的选项可能会需要接收一个值），后面的就是非选项相关的参数。

示例：
```bash
# 将标准输出的内容 123 写入到 1.txt
echo "123" | cat - > 1.txt
echo "456" | cat - >> 1.txt

# 输出文件1.txt 的行数
cat -n 1.txt

# 在 -n 前加入 --，会报：cat: -n: No such file or directory 的错误
# 即 -- 后面的内容不会被解析成cat命令的参数，而是解析成一个名为 "-n" 的文件
cat -- -n 1.txt

# 创建一个 -n 的文件
touch -- -n

# 无法删除 -n 文件，提示：rm: invalid option -- 'n'
rm -n

# 删除 -n 文件
rm -- -n
#或者
rm ./-n
```

参考知乎回答：[Linux中短横线“-”的用途有哪些？](https://www.zhihu.com/question/51941417)