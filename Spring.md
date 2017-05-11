Spring

依赖注入：把对象之间的依赖关系转而用配置文件管理
Ioc容器：被Bean包裹的对象，Spring把对象包装在Bean中从而达到管理这些对象以及一系列额外操作的目的

# Bean

## Bean的创建

Spring Bean创建是典型的工厂模式，它的顶级接口是BeanFactory

- ListableBeanFactory: Bean是可列表的
- HierarchicalBeanFactory: Bean是有继承关系的
- AutowireCapableBeanFactory：定义Bean的自动装配规则

## Bean的定义

Bean的定义主要是由BeanDefinition描述

Bean的定义完整描述了在Spring配置文件中你定义的<bean/>节点所有信息，包括各种子节点。

## Bean的解析

Bean的解析主要是对Spring配置文件的解析

# Context

给Spring提供一种运行时环境，用以保存各个对象的状态

ApplicationContext是Context的顶级父类

- ConfigurableApplicationContext表示该Context是可修改的
- WebApplicationContext为Web准备的Context

ApplicationContext完成以下几件事：

1. 表示一个应用环境
2. 利用BeanFactory创建Bean对象
3. 保存对象关系表
4. 能够捕获各种事件

# 创建BeanFactory工厂

AbstractApplicationContext.refresh

1. 构建BeanFactory，以便产生所需的演员
2. 注册可能感兴趣的事件
3. 创建Bean实例对象
4. 触发被监听的事件

FactoryBean生产Bean的Bean
