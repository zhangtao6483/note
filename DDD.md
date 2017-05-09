领域驱动模型

# 入门

## 为什么需要DDD

** 贫血症**：
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


