title: IDEA设置DashBoard
date: 2018/03/31  20:46:25
tags: 路漫漫兮
categories: 小试牛刀
---

最近学习SpringBoot时，运行时要用到DashBoard这个功能，使用的是IDEA，找了半天没找到，花了点时间看了下IDEA的帮助文档，文档果然是个好东西，分分钟解决<!--more-->
文档上有一段话关于DashBoard：
```
To enable the dashboard:
1.Click Edit Configurations from the run/debug configurations selector.
2.Select Defaults from the list in the left-hand section.
3.Under the Run Dashboard Types section, click   and select the necessary run configuration type. You can add multiple configuration types one by one.
4.Apply the changes and close the dialog.
```
翻译如下：
```
启用仪表板：
1. 点击运行/调试配置选择器中的编辑配置
2. 选择默认在左边的部分从列表
3. 运行仪表类型 栏目下，点击 和选择必要的运行配置类型。可以一个接一个地添加多种配置类型
4. 应用更改并关闭对话框
```

1. 工具栏上选择Run
2. 选择Edit Configurations
3. 选择左边列表Defaults
4. 右边下半部有一个Run DashBoard Type是折叠的，单击展开，单击加号选择需要运行的项目类型
因为我用的是SpringBoot，所以选择SpringBoot，应用之后，视图窗口会出现Run DashBoard的选项