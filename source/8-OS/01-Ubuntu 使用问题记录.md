# Ubuntu 使用问题记录

## IDEA 转换大小写失效
- 现象： 选中单词，使用`Ctrl-Shift-U`尝试转换大小写，结果单词变为u
- 原因： 快捷键被 ibus 占用，参考 [这里解决](https://youtrack.jetbrains.com/issue/IDEA-112533/Toggle-Case-Ctrl-Shift-U-not-working-under-Gnome-Linux)

> - 运行 `ibus-setup` 命令
> - 切换到 Emoji 页面
> - 修改或删除 `Ctrl-Shift-U` 快捷键

![test](./img/images.jpeg)