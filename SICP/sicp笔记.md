# SICP笔记

## 第一章 概念

### 1.1 代换模型
**组合式和复合过程确定的计算过程是（代换模型）：**
- 1. 求出各参数表达式（子表达式）的值
- 2. 找到要调用的过程的定义（根据第一个子表达式的求值结果）
- 3. 用求出的实际参数代换过程体里的形式参数
- 4. 求值过程体

### 1.2 计算进程

![enter image description here](https://mitpress.mit.edu/sicp/full-text/book/ch1-Z-G-7.gif)

-  先展开后收缩：展开中积累一些计算，收缩是完成这些计算
- 解释器要维护待执行计算的轨迹，轨迹长度与后续计算的次数成正比
- 积累长度为线性的，计算序列的长度也为线性，称为**线性递归进程**

![enter image description here](https://mitpress.mit.edu/sicp/full-text/book/ch1-Z-G-10.gif)

- 没有展开/收缩，直接计算
- 计算轨迹中的信息量为常量，只要维护几个变量的当前值
- 计算序列的长度为线性的具有这种性态的计算进程称为**线性迭代进程**

尾递归形式和尾递归优化
- 一个递归定义的过程称为是尾递归的，如果其中对本过程的递归调用都是过程执行的最后一个表达式
- 虽然是递归定义过程，计算所需的存储却不随递归深度增加。尾递归技术就是重复使用原过程在执行栈里的存储，不另行分配

![enter image description here](https://mitpress.mit.edu/sicp/full-text/book/ch1-Z-G-13.gif)

**树形递归**

**换硬币**
人民币的硬币有1元，5角，1角，5分，2分和1分
*问题*：给了一定的人民币，问有多少种不同方式将它换成硬币？
这个问题用递归方式解决比较简单和自然
首先要分析问题，规划出一种对问题的递归观点（算法）

  例如：确定一种硬币排列，币值 a 换为硬币的不同方式等于：
    -  将 a 换为不用第一种硬币的方式，加上
    - 用一个第一种硬币（设币值为 b）后将 a-b 换成各种硬币的方式

这里把用 k 种硬币得到币值 a 归结为两个更简单的情况
- 用 k - 1 种硬币得到 a （减少一种硬币）
- k 种硬币得到较少的币值（前面说的 a-b，减少了币值）

换硬币的几种基本情况：
- a = 0，计 1 种方式
- a < 0，计 0 种方式，因为不合法
- 货币种类 n = 0 但 a 不是 0，计 0 种方式，因为已无货币可用

**递归的观点（递归的分析）**
设法把解决原问题归结为在一定条件下解决一个/几个相对更简单的同类问题（或许还有另外的可以直接解决的问题）

###1.3 高阶函数（Higher-order function）

[高阶函数](https://zh.wikipedia.org/wiki/%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0)是**至少满足**下列一个条件的函数：
- 接受一个或多个函数作为输入
- 输出一个***函数***

**高阶过程：**以过程作为参数或返回值，操作过程的过程

**lambda 表达式**是把一段计算参数化，抽象为一个（匿名）过程

直接使用 lambda 表达式，主要作用是
- 不引入过程名，简单（如果只用一次）
-  直接描述过程，有可能使程序更清晰

在 OO 语言里提供类似函数式语言 lambda 表达式的功能，主要是提供匿名函数，简化程序书写；也想支持一些高级编程技术

## 第二章 构造数据抽象
### 2.1 抽象屏障
建立层次性抽象屏障的价值：
-  数据表示和使用隔离，两部分可以独立演化，容易维护和修改
- 实现好的数据抽象可以用于其他程序和系统，可能做成库
- 一些设计决策可以推迟，直到有了更多实际信息后再处理

## 第三章 模块化、对象和状态
### 3.1 赋值和局部状态
对象的观点是对世界的一种看法：
- 世界由一批事物(对象)组成，每个对象有其状态和行为方式
- 对象的状态随时间不断变化，其行为受到历史的影响

现实世界里真实对象随着时间而改变状态，要模拟它们
- 程序里就需要有随着运行不断改变状态的对象
- 为此需要改变程序对象状态的操作
	主要是赋值操作
- 赋值，就是修改对象的状态变量
	常规语言中的变量，OO 里的对象等等，都是对象

在复杂计算中，从其中一部分观察，其他部分都像在随着时间不断变化，它们通常都隐藏了一些变化的细节（内部状态）。如果想基于这种认识这样分解系统，最直接的方式就是用计算对象模拟系统随时间变化的行为，用局部变量模拟部分的内部状态，用赋值模拟状态变化

处理状态变化的两种方式：
- 通过显式计算
	其中通过额外的参数传递随时间变化的状态
- 采用局部状态变量和赋值
	自然地利用变化的状态

没有赋值的时候
- 以同样参数调用同一过程总得到同样结果
- 这种过程就像是在计算数学的函数
- 无赋值的编程称为**函数式编程**

有了赋值（set!）
- 丧失了引用透明性
- 代换模型不再适用
- “同一个”的问题也不再简单清晰

比较两个过程
```lisp
(define (make-decrementer
balance)
(lambda (amount)
(- balance amount)))
(define D (make-decrementer 25))
(D 20)
5
(D 10)
15
(D 10)
15
```

```lisp
(define (make-simp-withdraw balance)
(lambda (amount)
(set! balance (- balance amount))
balance))
(define W (make-simp-withdraw 25))
(W 20)
5
(W 10)
-5
(W 10)
-15
```

代换模型能解释 make-decrementer，但它无法解释 make-simplified-withdraw，不能解释为什么两个(W 10) 调用会得到不同结果

如果一种语言支持“同样的东西可以相互替换”，而且这种替换不会改变表达式的值（程序的意义），称这种语言具有引用透明性，*纯函数式语言具有引用透明性*，赋值打破了语言的引用透明性（基于赋值的程序设计称为命令式程序设计）

### 3.2 求值的环境模型
求值规则
 - 整数和实数的值直接取得，变量代换为约束值
 - 组合式求值将过程作用于一组参数，先求值所有参数，而后
	 - 对基本过程，直接得到它作用于实参的结果
	 - 对复合过程，用实参代换过程体的形参后求值得到的过程体

有了赋值后，代换模型就失效了
- 变量已不再是代表值的简单名字，而表示某种“存储位置”
- 有关位置保存着值，而保存的值可以随计算进展而改变
- 可行的求值模型必须能反映变量值的变化

**新求值模型**，组合表达式的基本求值规则仍是：
- 求出组合式的各子表达式的值
- 将运算符表达式的值作用于运算对象表达式的值
- 赋值 set! 的执行可能改变当前环境里已有的约束，define 的执行可能导致环境中增加新的约束
- 在基于新模型的求值过程中，过程定义，调用和退出导致的环境变化是最重要最需要关注的事项

**内部定义**
在建立过程对象时
- 内部过程的名字与相应过程对象的约束在一个局部框架里，与其他框架里的同名对象（变量或过程）无关
- 内部过程对象的环境指针指向外围过程调用时的环境，因此内部过程可以直接使用其外围过程的局部变量（形式参数等）

每次调用有内部过程定义的过程时，将新建一个框架
- 包括重新建立其中的各内部过程对象
- 代码的处理见前面说明，不同过程对象之间是否共享代码是系统的实现细节，不影响语义

###3.3 用变动数据做模拟
**约束传播**：
- 一组变量通过某些方式相互约束
- 一旦某变量有了值，其值可以通过相关约束传播到其他变量
- 如果已有的约束足够强，就可以确定某些未定变量的值
- 如果一个变量的值改变，这种改变也会通过约束传播出去

### 3.4 并发：时间是一个本质问题

参考文献：
http://www.math.pku.edu.cn/teachers/qiuzy/progtech/

