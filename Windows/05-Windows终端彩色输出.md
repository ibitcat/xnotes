Windows 终端彩色输出和 linux 类似，基本格式如下：

```
echo ^[[32mhello^[[0m
```

其中 `^[` 并不是 `^` 和 `[` 这两个字符，而是`\x1b`(十六进制)或 `\27`(十进制)或`\033`(八进制)，即`ESC`的 ASCII 码。

Windows 中 `ESC` 输入不能和linux那样使用 -e 参数转义，输入方式如下：

- 在命令提示符窗口下，同时按下 `ctrl + [` 组合键；
- 在文本编辑器下，按下 alt 键，再依次按小键盘的 2、7 数字键，最后松开 alt 键，最终得到``；
- 使用以下脚本：
    ```bash
    @echo off
    ::src: https://stackoverflow.com/a/5344911
    set ESC_CHAR=
    @for /f "delims=#" %%i in ('prompt #$E#^&echo on^&for %%a in ^(1^) do rem') do set ESC_CHAR=%%i

    echo %ESC_CHAR%[32mhello%ESC_CHAR%[0m

    pause
    ```