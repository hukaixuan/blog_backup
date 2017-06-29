---
title: "Flask Context Processors"
date: 2017.04.11 22:53
tags:
 - Flask
---

--------------------
我学习flask是跟着Miguel Grinberg的《Flask Web Development》上手的、书上都是选了一些更为方便实用的扩展包进行开发的，跟着书中的项目做下来，能有了一个比较好的flask架构方案，但对很多flask的机制还是很模糊。最近自己在写一个小项目，遇到许多坑，于是开始边看官方文档边继续写，以后也会记录一些个人觉得比较好，比较有用的知识点。              


  今天发现了一个很有用的装饰器 context_processor，刚好解决了小项目中困扰我的一个问题。主要有两个作用：
- 可以向模板上下文中自动注入变量，在模板中调用：
```
@app.context_processor 
def inject_user():
      return dict(user=g.user)
```
注意：返回值必须是一个dictionary
虽然这个例子的确没啥实际意义(g 本来就可以在模板中直接调用)，但做例子说明这点作用还是可以的
- 不但可以注入变量，还可以直接注入方法：
```
@app.context_processor 
def utility_processor():
       def format_price(amount, currency=u  ):
              return u {0:.2f}{1} .format(amount, currency)
       return dict(format_price=format_price)
```
在模板中的调用方法：`{ { format_price(0.33) } }` 
--------------------
