# 光标操作

| 快捷键   | 说明                       |
| -------- | -------------------------- |
| ctrl + a | 光标到行首(Ahead)          |
| ctrl + e | 光标到行尾(End)            |
| ctrl + b | 光标左移一个字符(Backward) |
| ctrl + f | 光标后移一个字符(Forward)  |
| esc  + b | 光标左移一个单词           |
| esc  + f | 光标后移一个单词           |


# 字符操作

| 快捷键         | 说明                                 |
| -------------- | ------------------------------------ |
| ctrl + d       | 删除当前字符(Delete)                 |
| ctrl + h       | 删除前一个字符（Backspace)           |
| ctrl + k       | 删除光标至行尾                       |
| ctrl + u       | 删除光标至行首 (iterm中是删除当前行) |
| ctrl + w       | 删除光标前一个单词                   |
| ctrl + y       | 粘贴刚删除的内容                     |
| ctrl + t       | 交换光标前两个字符                   |
| esc  + c       | 当前字母大写，其后到单词末尾字母小写 |
| esc  + u       | 当前到单词末尾字母大写               |
| ctrl + v       | 插入特殊字符，如<tab>, <^M:Enter>    |
| ctrl + x + u   | 撤销                                 |
| ctrl + insert  | 复制                                 |
| shift + insert | 粘贴                                 |

# 命令操作

| 快捷键   | 说明                                   |
| -------- | -------------------------------------- |
| ctrl + i | 自动实全，<tab>的效果                  |
| ctrl + n | 显示下一个命令(next)                   |
| ctrl + p | 显示上一个命令(previos)                |
| ctrl + r | 搜索历史命令                           |
| ctrl + g | 退出历史命令搜索模式                   |
| esc  + . | 获取上一条命令最后的参数               |
| ctrl + j | 回车                                   |
| ctrl + m | 回车，方便与 ctrl + n 组合使用         |
| ctrl + o | 回车，方便与 ctrl + p 组合使用         |
| `!n`     | 显示history中的第n条命令               |
| `!!`     | 显示上一条命令和参数                   |
| `!:0`    | 显示上一条命令第一段，即命令本身，不含 |
| `!:n`    | 显示上一条命令第n段，即第n个参数       |
| `!$`     | 显示上一条命令最后一段，即最后的参数   |
| `!v`     | 显示最近一条以`v`开头的命令和参数      |


# 屏幕操作

| 快捷键           | 说明               |
| ---------------- | ------------------ |
| ctrl + l         | 清屏 (cLear)       |
| ctrl + s         | 锁屏，阻止屏幕输出 |
| ctrl + q         | 解屏，允许屏幕输出 |
| shift + pageup   | 向上滚屏           |
| shift + pagedown | 向下滚屏           |

# 进程操作

| 快捷键   | 说明                             |
| -------- | -------------------------------- |
| ctrl + c | 杀死当前进程                     |
| ctrl + z | 当前进程转后台，使用`fg`命令恢复 |