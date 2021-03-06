# 工厂方法模式(Factory)

工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类事哪一个。工厂方法让类把实例化推迟到子类。

工厂方法让子类决定要实例化的类是那一个，所谓的“决定”并不是指模式允许子类本身在运行时决定，而是指在编写创建者类时，不需要知道实际创建的产品是哪一个

-------------------

**简单工厂:**

简单工厂模式的优点如下：

- 工厂类含有必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例，客户端可以免除直接创建产品对象的责任，而仅仅“消费”产品；简单工厂模式通过这种做法实现了对责任的分割，它提供了专门的工厂类用于创建对象。
- 客户端无需知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可，对于一些复杂的类名，通过简单工厂模式可以减少使用者的记忆量。
- 通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类，在一定程度上提高了系统的灵活性。

简单工厂模式的缺点如下：

- 工厂类中包括了创建产品类的业务逻辑，一旦工厂类不能正常工作，整个系统都要受到影响。
- 系统扩展困难，一旦添加新产品就需要修改工厂逻辑，在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护。
- 简单工厂模式由于使用了静态工厂方法，造成工厂角色无法形成基于继承的等级结构。

```java 
// 简单工厂
public class SimplePizzaFactory{
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("chesse")) {
            pizza = new ChessePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        }
        return pizza;
    }
}

public class PizzaStore {
    SimpalePizzaFactory factory;

    public PizzaStore (SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public PIzza orderPizza(String type) {
        Pizza pizza;

        pizza = factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        return pizza;
    }
    // 其他方法
}
```

-------------------

**抽象工厂模式:**

提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类

```java
public abstract class Pizza {
    String name;
    Dough dough;
    Chesse chesse;
    Pepperoni pepperoni;
    Clams clam;

    abstract void prepare();

    void setName(String name) {
        this.name = name;
    }
    String getName() {
        return name;
    }
}

public class ChessePizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;

    public ChessePizza (PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }

    void prepare() {
        System.out.println("Prepareing " + name);
    }
}

public abstract class PizzaStore {
    public abstract Pizza createPizza(String item);
    public Pizza orderPizza(String item) {
        Pizza pizza = createPizza(item);
        pizza.Prepare();
        pizza.Bake();
        pizza.Cut();
        pizza.Box();
        return pizza;
    }
}

public class NYPizzaStore extends PizzaStore {
    protected Pizza = createPizza (String item) {
        Pizza pizza = null;
        PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory;

        if (item.equals("chesse")) {
            pizza = new ChessePizza(ingredientFactory);
            pizza.setName("New York Chesse Pizza");
        }

        return pizza;
    }
}
```

通过抽象工厂所提供的接口，可以创建产品的家族，利用这个接口书写代码，我们的代码将从实际工厂中解耦，以便在不同的上下文中实现各式各样的产品




