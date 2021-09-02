---
title: android奇技淫巧 21 code review
date: 2019-10-23 21:55:10
tags:
  - android
---

Code Review

<!--more-->

# 什么是Code Review
代码评审是指在软件开发过程中，通过对元代码进行系统检查的过程。通常目的是查找系统缺陷，确保软件总体质量和提升开发者自身水平。Code Review是轻量级代码精神，相对于正式代码评审，轻量级代码评审所需要的各种成本要明显的低，如果流程正确，他可以起到更加积极的效果。

# 具体Review事项

- 注意命名规范(类名，成员变量，接口等)
- Android Lint检查，借助Android Studio工具未完成
- 检查资源文件使用情况，strings.xml,dimen.xml等使用情况，图片资源大小，命名，点9格式使用问题
- 代码格式化等问题

# 代码Review的好处

1. 通过代码Review可以提高产品代码的质量
2. 通过代码Review可以增强团队成员之间的沟通
3. 通过代码Review能够有效的提前发现代码中存在的缺陷和BUG，降低线上出现事故的概率
4. 通过代码Review提供团队成员的编程能力，不同成员之间对功能设计思路的重构可以很好的提高团队成员 的跟人专业技能


# 代码Review实践

## 主要流程

1. 每个人介绍各自的功能需求，实现的主要逻辑，核心代码
2. 团队成员提出问题，其他实现思路等
3. 讨论不同实现思路的方式以及优劣势

# Android代码Lint检查
除了组员之间的代码review，我们还可以通过Lint检查。
通过Android studio编译工具执行Lint检查

1. 执行 android studio -->  Analyze  --> Inspect code操作
![打开代码检查框](/assets/tools/tools-review-01.png)

2. 在代码检查框中选择为整个工程执行lint检查？还是整个module或者是当前的源文件执行lint检查，这里为了简单起见，我们只为当前的源代码文件执行lint检查，然后执行确认即可
![](/assets/tools/tools-review-02.png)

3. 接下来就可以在我们的Android studio查看lint检查结果了
![](/assets/tools/tools-review-03.png)

可以发现我们lint检查之后出现了许多检查结果，其中在uuelectricrenter项目下存在着多条检查信息，下面我们就分析一下检测结果。