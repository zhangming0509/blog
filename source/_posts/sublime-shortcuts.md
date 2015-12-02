title: sublime快捷键
date: 2015-12-02 12:08:00
tags: sublime
---

使用sublime也有一年多了,今天总结下比较常用的快捷键.

跳转类
------------

  * Ctrl+P: 搜索项目中的文件
      * 输入’#’: 在当前文件匹配结果
      * 输入’@’: 同Ctrl+R
      * 输入’:’: 同Ctrl+G
  * Ctrl+R: 搜索函数名或类名
  * Ctrl+G: 跳转到指定行
  F12: 跳转到定义处

文件窗口切换
-----------------

  * Ctrl+Tab: 在当前窗口和前一个窗口之间切换
  * Ctrl+PgDown: 切换到右侧标签
  * Ctrl+PgUp: 切换到左侧标签
  * Alt+数字: 切换到指定序号的标签(从左往右从1开始递增)

<!--more-->

查找类
----------

  * Ctrl+F: 在当前文件中查找
      * F3: 向下查找
      * Shift+F3: 向上查找
  * Shift+Ctrl+F: 在多个文件中查找(可查找指定目录下的文件)
      * F4: 向下查找
      * Shift+F4: 向上查找
  * Ctrl+F3: 快速查找(先选中要查找的内容,然后按键,可定位到下一处)
  * Alt+F3: 全选当前选中内容,适用于多行编辑,或全部

缩进类
--------------

  * Ctrl+[: 整行向右缩进
  * Ctrl+]: 整行向左缩进
  * Tab: 光标处向右缩进(选中块后,可使块代码向右缩进)
  * Shift+Tab: 光标处向左缩进(选中块后,可使块代码向左缩进)

行编辑类
-------------

  * Ctrl+Shift+K: 删除整行
  * Ctrl+Shift+Down: 当前行与上行互换
  * Ctrl+Shift+Up: 当前行与下行互换
  * Ctrl+Enter: 当前行后插入新行
  * Ctrl+Shift+Enter: 当前行前插入新行
  * Ctrl+J: 将选中的多行合并为一行
  * Crtl+Del: 从当前光标处删除至当前词组结尾
  * Crtl+Backspace: 从当前光标处删除至当前词组开头
  * Crtl+Shift+Del: 从当前光标处删除至当前行结尾
  * Crtl+Shift+Backspace: 从当前光标处删除至当前行开头

选择类
----------------

  * Ctrl+A: 全选当前文件
  * Ctrl+D: 选中光标所在处词组, 再按一次D, 选中下一处
  * Ctrl+L: 选中光标所在行, 再按一次L, 继续向下选中下一行
  * Ctrl+Shift+M: 选中括号内的内容, 再按一次M, 连同括号选中
  * Ctrl+Shift+J: 选中光标所在的缩进区块

其它
--------------

  * Ctrl+N: 新建
  * Ctrl+W: 关闭当前文件
  * Ctrl+Shifr+W: 关闭所有文件(退出sublime)
  * Ctrl+K+B: 关闭/打开侧边栏
  * Ctrl+K+U: 当前单词转换为大写
  * Ctrl+K+L: 当前单词转换为小写
  * Ctrl+M:
      * 光标位于括号内侧时切换光标至括号内侧的尾部, 再按M, 切换至括号头部内侧
      * 光标紧邻于括号外侧时, 切换至另一侧的外边缘(此种情况优先级高于上种情况, 例如:((test1) test2), 光标位于两个左括号中间, 此种情况下按键Ctrl+M会在(test1)外侧切换光标)
  * Ctrl+/：注释当前行
  * Ctrl+Shift+/: 当前位置插入注释
  * Ctrl+Shift+V: 粘贴并匹配当前缩进
  * 按Ctrl，依次点击多个需要编辑的位置, 可同时编辑多行
  * Shift+右键拖动: 多行编辑
  * Alt+Shift+数字：分屏显示
  * Alt+Down: 数值减0.1
  * Alt+Up: 数值加0.1
  * Ctrl+Down: 数值减1
  * Ctrl+Up: 数值加1
  * Alt+Shift+Down: 数值减10
  * Alt+Shift+Up: 数值加10
