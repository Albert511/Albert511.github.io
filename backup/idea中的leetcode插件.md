# 一、下载插件

打开idea编译软件，点击右上角的File-->Setting-->Plugins。如下图所示：

![image](https://github.com/user-attachments/assets/596a6576-53db-46f0-adee-6f0c79896a6a)


# 二、插件配置

 插件安装之后，继续点击File-->Setting-->Tools->leetcode plugin，如下图所示：

![image](https://github.com/user-attachments/assets/b5e3e43d-4d5b-4b65-88d1-64da4df2c14e)

  打开这个窗口之后，我们接下来的工作就是进行完成对插件配置的工作了。具体的操作全部展示在下图中：

![image](https://github.com/user-attachments/assets/70b9b308-f078-41e4-85ae-1ce40bcde134)


### **注意** 

​     如果要是想专门刷算法题，我的建议是：重新建一个项目来专门存放算法源代码。

关于部分参数的定义：

- ${question.title}   题目标题   示例:两数之和
- ${question.titleSlug}   题目标记   示例:two-sum
- ${question.frontendQuestionId}   题目编号
- ${question.content}   题目描述
- ${question.code}   题目代码
- $!velocityTool.camelCaseName(str)   转换字符为大驼峰样式（开头字母大写）
- $!velocityTool.smallCamelCaseName(str)   转换字符为小驼峰样式（开头字母小写）
- $!velocityTool.snakeCaseName(str)   转换字符为蛇形样式
- $!velocityTool.leftPadZeros(str,n)   在字符串的左边填充0，使字符串的长度至少为n
- $!velocityTool.date()   获取当前时间 

# 三、插件的使用

​    点击leetcode，进入题目列表，并且点击登录按钮，进行账号登陆。如下图所示：

![image](https://github.com/user-attachments/assets/4f045242-5065-47fd-8f8b-e51e28afdc5d)


将自己的账户登陆上去之后， 就可以进行对算法题的编写和解答了。点击一个题目（如 **两数之和**），然后在右边编写代码，点击运行。具体流程如下图所示：

![image](https://github.com/user-attachments/assets/e91e98e7-cfa1-4af2-88aa-8a59df971e9b)


  根据题目提示，编写相应的代码：

![image](https://github.com/user-attachments/assets/8bb50212-5588-4b1b-aa09-8e4502193429)

点击运行按钮运行程序：

![image](https://github.com/user-attachments/assets/1ef95516-24b0-44c8-874b-454654e0960a)

 最后，下面的执行窗口就会显示代码的执行结果，来判断代码是否正确：

![image](https://github.com/user-attachments/assets/faedb9d6-7a24-44cb-86d7-cffd12ee0bd3)

