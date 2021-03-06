---
layout:     post
title:      "【Linux】重要快捷键和常用通配符"
date:       2017-01-18 12:00:00
author:     "Yuki"

---
## 重要快捷键

1. tab 使用tab键来补齐命令、目录、参数
2. Ctrl+c 强行终止当前程序
3. Ctrl+d 键盘输入结束或强行终止终端
4. Ctrl+s 暂停当前程序
5. Ctrl+z 将当前程序放至后台运行，要恢复到前台使用fg(frontground)
6. Ctrl+a 将光标移至行头
7. Ctrl+e 将光标移至行尾
8. Ctrl+k 删除从光标所在位置到行末
9. Alt+backspace 向前删除一个单词
10. Shift+PgUp(PgDn) 将终端显示向上（下）滑动
11. 方向键上下可以恢复上一个/下一个命令

***

## 常用通配符
1. *匹配0个或多个字符
2. ？ 匹配任意一个字符
3. [list] 匹配list中任一字符
4. [!list] 匹配除list中任一字符外的字符
5. [c1-c2] 匹配c1-c2范围内的任一字符
6. {string1，,string2...} 匹配string1或string2或...
7. {c1-c2} 匹配c1-c2范围中的所有字符