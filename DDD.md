领域驱动模型

# 入门

## 为什么需要DDD

**贫血症**：<br>
缺少内在行为的领域对象

行为：

- 领域对象中由公有的getter对象和setter对象，几乎没有业务逻辑
- 经常使用的领域对象包含了系统主要的业务逻辑，并且多数情况下需要在服务层（Service Layer）或者应用层（Application Layer）调用getter和setter

劣势：

- 开发者需要将很多时间花在对象和数据存储之间的映射上，而只是将关系型数据库的模型映射到了对象上而已。
- 槽点就是有大量的getter，setter方法，像下面这个代码。。。

```java
@Transactional
public void saveCustomer(String customerId, String consumerFirstName ... String country) {
    Customer customer = customerDao.readCustomer(customerId);
    if (customer == null){
        customer = new Customer();
        customer.setCustomerId(customerId);
    }
    
    if (customerFirstName != null){
        customer.setCustomerFirstName(customerFirstName);
    }

    ....
    
    if (country != null){
        customer.setCountry(country);
    }
    customerDao.saveCustomer(customer);
}
```

这段代码存在的问题

1. saveCustomer()业务意图不明确
2. 方法的实现本身增加了潜在的复杂性
3. Customer领域对象根本就不是对象，而只是一个数据持有器（data holder）

## 如何DDD

两大支柱：

- 通用语言
- 限界上下文

**通用语言：**
通用语言是团队自己创建的公用语言，团队中同时包含领域专家和软件开发人员(就是方法命名、方法参数等等的达成一致)

```java
public interface Customer {
    public void changePersonalName(String firstName, String lastName);
    
    ....

}
```

```java
@Transactional
public void changeCustomerPersonalName(Stirng customerId, String customerFirstName, String customerLastName) {
    Customer customer = customerRepository.customerOfId(customerId);
    
    if (customer == null) {
        throw new IllegalStateExccption("Customer does not exist");
    }
    
    customer.changePersonalName(customerFirstName, customerLastName);

}
```

不用在只修改用户姓名参数后跟上10个null了 ：）

**界限上下文**：


---

再来个getter，setter的例子，将一个待定项（Blacklog Item）提交到冲刺（Sprint）中去
通常做法，使用属性访问的方式：

```java
public class BacklogItem extends Entity {
    private SprintId sprintId;
    private BacklogItemStatusType status;
    ...
    
    public void setSprintId(SprintId sprintId) {
        this.sprintId = sprintId;
    }
    public void setStatus(BacklogItemStatusType status) {
        this.status = status;
    }
    ... 
}

```

客户端代码：

```java
// 客户端通过设置sprintId和status将一个BackLogItem提交到Sprint中
backlogItem.setSprintId(sprintId);
backlogItem.setStatus(BacklogItemStatusType.COMMITTED);

```

使用领域对象的行为，这种行为表达出了领域中的通用语言

```java
public class BacklogItem extends Entity {
    
    private SprintId sprintId;
    private BacklogItemStatusType status;
    ...
    
    public void commitTo(Sprint aSprint) {
        if (!this.isScheduledForRelease()) {
            throw new IllegalStateException(
                "Must be scheduled for release to commit to sprint.");
        }
        
        if (this.isCommittedToSprint()) {
            if (!aSprint.sprintId().equals(this.sprintId())) {
                this.uncommitFromSprint();
            }
        }
        
        this.elevateStatusWith(BacklogItemStatus.COMMITTED);
        
        this.setSprintId(aSprint.sprintId());
        
        DomainEventPublisher.instance()
            .publish(new BacklogItemCommitted(this.tenant(), this.backlogItemId(), this.sprintId()));
    }
    ... 
}
```

```java
// 客户端通过特定于领域的行为将BackLogItem提交到Sprint中
backlogItem.commitTo(sprint);
```

# 领域，子域，限界上下文

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_1.png)

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_2.png)


领域（Domain）即是一个组织所做的事情以及其中所包含的一切

子域：

- 核心域 <br>整个业务领域的一部分，也是业务成功的主要促成因素
- 支撑子域 <br>限界上下文对应着业务的某些重要方面，但却不是核心
- 通用子域 <br>被用于整个业务的子域

限界上下文：
限界上下文是一个显示边界，领域模型便存在于边界之内。在边界内，通用语言中的所有术语和词组都有特定的语义，而模型需要准确地反应通用语言

# 上下文映射图

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_3.png)


# 架构

## 分层架构模型

DDD系统所采用的传统分层架构，其中核心域只位于架构中的其中一层，其上层为**用户界面层（User Interface）** 和 **应用（Application Layer）**，其下层是**基础设施层（Infrastructure layer）**

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_4.png)


严格分层架构（Strict Layers Architecture）,某层只能与直接位于下方的层发生耦合<br>
松散分层架构（Relaxed Layers Architecture）,允许任意上方层与任意下方层发生耦合

**依赖倒置原则**
> 高层模块不应该依赖于底层模块，两者都应该依赖于对象
> 抽象不应该依赖于细节，细节应该依赖于抽象

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_5.png)

## 六边形架构（端口和适配器）

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_6.png)

## 面向服务架构（Service-Oriented Architecture， SOA）

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_7.png)

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_8.png)

## REST

## 命令和查询职责分离-CQRS

CQRS（Command-Query Responsibility Segregation）<br>
CQRS是将紧缩（Stringent）对象设计原则和命令-查询分离（CQS）应用在架构模式中的结果<br>
一个方法要么是执行某种多做的命令，要么是返回数据的查询，而不能两者皆是

## 事件驱动架构

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/ddd/ddd_9.png)
