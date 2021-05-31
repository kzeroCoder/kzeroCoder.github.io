---
title: Java杂记——设计模式
author: Kzero Coder
date: 2021-05-28 11:00:00 +0800
categories: [Blogging, JavaNotes]
tags: [writing]
math: true
---

<head>
    <script type="text/x-mathjax-config"> 
   		MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); 
   	</script>
    <script type="text/x-mathjax-config">
    	MathJax.Hub.Config({tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
    </script>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" 	type="text/javascript">
	</script>
</head>


# 设计模式 Design Pattern

> 本文参考https://www.liaoxuefeng.com/wiki/1252599548343744/1264742167474528
>
> 仅作个人摘记，学习建议前往原网站

## 创建型模式

### 工厂方法 Factory Method

**定义一个用于创建对象的接口，让子类决定实例化哪一个类**

目的是使得创建对象和使用对象分离

以廖雪峰的教程为例：

```java
public interface NumberFactory {
    // 创建方法:
    Number parse(String s);

    // 获取工厂实例:
    static NumberFactory getFactory() {
        return impl;
    }

    static NumberFactory impl = new NumberFactoryImpl();
}

public class NumberFactoryImpl implements NumberFactory {
    public Number parse(String s) {
        return new BigDecimal(s);
    }
}
```

这样调用方就可以忽略真正的工厂NumberFactory和实际产品BigDecimal



实际上使用的更多的是静态工厂方法 Static Factory Method，即**提供静态方法创建产品**

如Java源码Integer

```java
public final class Integer {
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    ...
}
```

这样可以隐藏创建产品的细节，从而降低调用方对被调用者的可知性



### 抽象工厂 Abstract Factory

**抽象工厂提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类**

抽象工厂模式中工厂是抽象的，产品也是抽象的

```java
public interface AbstractFactory {
    public static AbstractFactory createFactory(String name) {
        ...
    }
    
     // 创建Html文档:
    HtmlDocument createHtml(String md);
    // 创建Word文档:
    WordDocument createWord(String md);
}

// Html文档接口:
public interface HtmlDocument {
    String toHtml();
    void save(Path path) throws IOException;
}

// Word文档接口:
public interface WordDocument {
    void save(Path path) throws IOException;
}


```

这样只要实现抽象工厂和产品，就能够实现生成不同产品的需求，而不需要更改调用者的代码；调用者只需要更改调用静态工厂方法时输入的参数，就可以选择不同的工厂生成特定的产品。



### 生成器 Builder

**生成器将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。**

- 使用多个小型工厂来最终创建出一个完整对象

更确切的说，是将要转换的源数据分为一部分，通过每一部分的特征选择不同工厂进行转换，最终返回一个转换后的完整对象

```java
public class HtmlBuilder {
    private HeadingBuilder headingBuilder = new HeadingBuilder();
    private HrBuilder hrBuilder = new HrBuilder();
    private ParagraphBuilder paragraphBuilder = new ParagraphBuilder();
    private QuoteBuilder quoteBuilder = new QuoteBuilder();

    public String toHtml(String markdown) {
        StringBuilder buffer = new StringBuilder();
        markdown.lines().forEach(line -> {
            if (line.startsWith("#")) {
                buffer.append(headingBuilder.buildHeading(line)).append('\n');
            } else if (line.startsWith(">")) {
                buffer.append(quoteBuilder.buildQuote(line)).append('\n');
            } else if (line.startsWith("---")) {
                buffer.append(hrBuilder.buildHr(line)).append('\n');
            } else {
                buffer.append(paragraphBuilder.buildParagraph(line)).append('\n');
            }
        });
        return buffer.toString();
    }
}
```

这样就可以将整个Markdown转换成HTML了



### 原型 Prototype

**用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象**

原型类似Object提供的clone()方法，但是由于clone()返回的是Object类，还需要强转，所以一般通过定义一个copy()方法进行实现

```java
public class Student {
    private int id;
    private String name;
    private int score;

    public Student copy() {
        Student std = new Student();
        std.id = this.id;
        std.name = this.name;
        std.score = this.score;
        return std;
    }
}
```

需要注意的是，很多实例持有文件、Socket这样的资源，无法复制给另一个对象，只有存储简单类型的值对象可以复制



### 单例 Singleton

**一个类只有一个实例，且该类能自行创建这个实例，单例类对外提供一个访问该单例的全局访问点**

- 目的是保证在一个进程中，某个类仅有一个实例

```java
public class Singleton {
    // 静态字段引用唯一实例:
    private static final Singleton INSTANCE = new Singleton();

    // 通过静态方法返回实例:
    public static Singleton getInstance() {
        return INSTANCE;
    }

    // private构造方法保证外部无法实例化:
    private Singleton() {
    }
}
```

所以单例模式实现方法很简单：

- 只有```private```方法，确保外部无法实例化；
- 通过```private static```变量持有唯一实例，保证全局唯一性；
- 通过```public static```方法返回唯一实例，使调用方能够获取实例；



## 结构型模式

### 适配器 Adapter

**将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作**

- 也就是传入A类，经过一系列处理得到B类，然后返回B类

```java
public class RunnableAdapter implements Runnable {
    // 引用待转换接口:
    private Callable<?> callable;

    public RunnableAdapter(Callable<?> callable) {
        this.callable = callable;
    }

    // 实现指定接口:
    public void run() {
        // 将指定接口调用委托给转换接口调用:
        try {
            callable.call();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

这样就能够将Callable接口转换为Runable接口，执行多线程操作



### 桥接 Bridge

**将抽象部分与它的实现部分分离，使它们都可以独立地变化**

- 主要用来解决组合爆炸问题

一个整体可能具有多种零件，这些零件又有多种选择，如果直接写子类，很快就会出现组合爆炸的情况

通过将零件抽象，在零件下分为各个子类，在生成具体整体的时候，注入特定零件，就可以解决该问题

```java
public abstract class Car {
    // 引用Engine:
    protected Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public abstract void drive();
}

public interface Engine {
    void start();
}
```



### 组合 Composite

**将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性**

- 经常用于树形结构

```java
public interface Node {
    // 添加一个节点为子节点:
    Node add(Node node);
    // 获取子节点:
    List<Node> children();
    // 输出为XML:
    String toXml();
}
```

之后实现各种Node，即可完成对父结点和子结点的统一管理

此处插入个人的lowB数据库实验中的使用

```java
package temp;

import method.join;
import method.projection;
import method.select;

import java.lang.reflect.Field;
import java.util.*;

public class TreeNode {
    static int blocknum = 8;
    static int blocksize = 16;
    TreeNode prev;	//父结点
    Object value;
    List<TreeNode> next;	//子结点

    public TreeNode(){
        prev = null;
        next = new ArrayList<>();
        value = "root";
    }

    public TreeNode(Object o){
        prev = null;
        next = new ArrayList<>();
        value = o;
    }

    public void addNode(TreeNode treeNode){
        next.add(treeNode);
        treeNode.setPrev(this);
    }

    public void removeNode(TreeNode treeNode){
        next.remove(treeNode);
    }

    public Object getValue(){
        return this.value;
    }

    public void setValue(Object value){
        this.value = value;
    }

    public TreeNode getPrev(){
        return this.prev;
    }

    public void setPrev(TreeNode prev){
        this.prev = prev;
    }

    //对结点中的value进行操作
    public void handle() throws ClassNotFoundException {
        for(TreeNode treeNode: next){
            treeNode.handle();
        }
        if(value instanceof join){
            ...
        }
        else if(value instanceof select){
           ...
        }
    }

    public List<TreeNode> getNext() {
        return new ArrayList<TreeNode>(next);
    }

    @Override
    public String toString() {...}
}

```



### 装饰器 Decorator

**动态地给一个对象添加一些额外的职责**

- 将核心功能和附加功能区分开来

```java
public interface TextNode {
    // 设置text:
    void setText(String text);
    // 获取text:
    String getText();
}

//核心结点
public class SpanNode implements TextNode {
    private String text;

    public void setText(String text) {
        this.text = text;
    }

    public String getText() {
        return "<span>" + text + "</span>";
    }
}

public abstract class NodeDecorator implements TextNode {
    protected final TextNode target;

    protected NodeDecorator(TextNode target) {
        this.target = target;
    }

    public void setText(String text) {
        this.target.setText(text);
    }
}

public class BoldDecorator extends NodeDecorator {
    public BoldDecorator(TextNode target) {
        super(target);
    }

    public String getText() {
        return "<b>" + target.getText() + "</b>";
    }
}
```

如此，即可通过BoldDecorator来修饰SpanNode；

由于NodeDecorator本身是继承自TextNode的，所以其也可以作为NodeDecorator的输入对象，从而实现Decorator嵌套



### 外观 Facade

**为子系统中的一组接口提供一个一致的界面**

- **基本思想：**创建统一中介，使得客户端直接和中介对话，通过中介完成一系列对子系统的操作

```java
// 工商注册:
public class AdminOfIndustry {
    public Company register(String name) {
        ...
    }
}

// 银行开户:
public class Bank {
    public String openAccount(String companyId) {
        ...
    }
}

// 纳税登记:
public class Taxation {
    public String applyTaxCode(String companyId) {
        ...
    }
}

public class Facade {
    public Company openCompany(String name) {
        Company c = this.admin.register(name);
        String bankAccount = this.bank.openAccount(c.getId());
        c.setBankAccount(bankAccount);
        String taxCode = this.taxation.applyTaxCode(c.getId());
        c.setTaxCode(taxCode);
        return c;
    }
}
```

如此，客户端只需要和Facade进行对话，即可完成所有事务



### 享元 Flyweight

**运用共享技术有效地支持大量细粒度的对象**

- **思想：**如果一个对象实例一经创建就不可变，那么反复创建相同的实例就没有必要，直接向调用方返回一个共享的实例就行

```java
public class Student {
    // 持有缓存:
    private static final Map<String, Student> cache = new HashMap<>();

    // 静态工厂方法:
    public static Student create(int id, String name) {
        String key = id + "\n" + name;
        // 先查找缓存:
        Student std = cache.get(key);
        if (std == null) {
            // 未找到,创建新对象:
            System.out.println(String.format("create new Student(%s, %s)", id, name));
            std = new Student(id, name);
            // 放入缓存:
            cache.put(key, std);
        } else {
            // 缓存中存在:
            System.out.println(String.format("return cached Student(%s, %s)", std.id, std.name));
        }
        return std;
    }

    private final int id;
    private final String name;

    public Student(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

主要应用在缓存中，减少客户端读取、查询的时间



### 代理 Proxy

**为其他对象提供一种代理以控制对这个对象的访问**

- 把A接口转换为A接口，中间进行一系列操作

```java
public AProxy implements A {
    private A a;
    public AProxy(A a) {
        this.a = a;
    }
    public void a() {
        //某些权限检测、缓存查找等操作
        this.a.a();
    }
}
```

这样就不需要对原来的A接口进行修改，即可实现某些功能，使得A接口功能更纯粹



## 行为型模式

### 责任链

**使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系**

- 将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止

```java
public class HandlerChain {
    // 持有所有Handler:
    private List<Handler> handlers = new ArrayList<>();

    public void addHandler(Handler handler) {
        this.handlers.add(handler);
    }

    public boolean process(Request request) {
        // 依次调用每个Handler:
        for (Handler handler : handlers) {
            Boolean r = handler.process(request);
            if (r != null) {
                // 如果返回TRUE或FALSE，处理结束:
                System.out.println(request + " " + (r ? "Approved by " : "Denied by ") + handler.getClass().getSimpleName());
                return r;
            }
        }
        throw new RuntimeException("Could not handle request: " + request);
    }
}
```

虽然用List更简单易懂，但是这样也就严格要求了add的顺序，否则可能导致权限大的排在权限小的前，所有后续权限小的handler都无法处理操作

所以对handler的设计，个人更倾向于树形结构，这样只需要维护树形结构满足父结点权限大于子结点权限，即可保证权限的最大利用



### 命令 Command

**将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作**

```java
public interface Command {
    void execute();
    void undo();
}

public class CopyCommand implements Command {
    // 持有执行者对象:
    private TextEditor receiver;

    public CopyCommand(TextEditor receiver) {
        this.receiver = receiver;
    }

    public void execute() {
        receiver.copy();
    }
    
    public void undo(){...}
}

public class PasteCommand implements Command {
    private TextEditor receiver;

    public PasteCommand(TextEditor receiver) {
        this.receiver = receiver;
    }

    public void execute() {
        receiver.paste();
    }
    
    public void undo(){...}
}
```

这样就可以把一系列命令用List保存起来，支持Undo，Redo，同时也可以通过Invoker对象执行命令



### 解释器 Interpreter

**给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子**

- 实际上就是使用正则表达式匹配



### 迭代器 Iterator

**提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示**

```java
public class ReverseArrayCollection<T> implements Iterable<T> {
    private T[] array;

    public ReverseArrayCollection(T... objs) {
        this.array = Arrays.copyOfRange(objs, 0, objs.length);
    }

    public Iterator<T> iterator() {
        return new ReverseIterator();
    }

    class ReverseIterator implements Iterator<T> {
        // 索引位置:
        int index;

        public ReverseIterator() {
            // 创建Iterator时,索引在数组末尾:
            this.index = ReverseArrayCollection.this.array.length;
        }

        public boolean hasNext() {
            // 如果索引大于0,那么可以移动到下一个元素(倒序往前移动):
            return index > 0;
        }

        public T next() {
            // 将索引移动到下一个元素并返回(倒序往前移动):
            index--;
            return array[index];
        }
    }
}
```

在多线程访问的时候，可能会引起ConcurrentModificationException，此时需要好好设计Iterator



### 中介 Mediator

**用一个中介对象来封装一系列的对象交互**

```java
public class Mediator {
    // 引用UI组件:
    private List<JCheckBox> checkBoxList;
    private JButton selectAll;
    private JButton selectNone;
    private JButton selectInverse;

    public Mediator(List<JCheckBox> checkBoxList, JButton selectAll, JButton selectNone, JButton selectInverse) {
        this.checkBoxList = checkBoxList;
        this.selectAll = selectAll;
        this.selectNone = selectNone;
        this.selectInverse = selectInverse;
        // 绑定事件:
        this.checkBoxList.forEach(checkBox -> {
            checkBox.addChangeListener(this::onCheckBoxChanged);
        });
        this.selectAll.addActionListener(this::onSelectAllClicked);
        this.selectNone.addActionListener(this::onSelectNoneClicked);
        this.selectInverse.addActionListener(this::onSelectInverseClicked);
    }

    // 当checkbox有变化时:
    public void onCheckBoxChanged(ChangeEvent event) {
        boolean allChecked = true;
        boolean allUnchecked = true;
        for (var checkBox : checkBoxList) {
            if (checkBox.isSelected()) {
                allUnchecked = false;
            } else {
                allChecked = false;
            }
        }
        selectAll.setEnabled(!allChecked);
        selectNone.setEnabled(!allUnchecked);
    }

    // 当点击select all:
    public void onSelectAllClicked(ActionEvent event) {
        checkBoxList.forEach(checkBox -> checkBox.setSelected(true));
        selectAll.setEnabled(false);
        selectNone.setEnabled(true);
    }

    // 当点击select none:
    public void onSelectNoneClicked(ActionEvent event) {
        checkBoxList.forEach(checkBox -> checkBox.setSelected(false));
        selectAll.setEnabled(true);
        selectNone.setEnabled(false);
    }

    // 当点击select inverse:
    public void onSelectInverseClicked(ActionEvent event) {
        checkBoxList.forEach(checkBox -> checkBox.setSelected(!checkBox.isSelected()));
        onCheckBoxChanged(null);
    }
}
```

通过中介解除组件之间的互相调用，将其变为对中介的调用，在后续后修改的情况下，只需要更改中介即可



### 备忘录 Memento

标准的备忘录模式有以下角色：

- Memonto：存储的内部状态；
- Originator：创建一个备忘录并设置其状态
- Caretaker：负责保存备忘录

实际上平时使用的时候并没有那么复杂

```java
public class TextEditor {
    ...

    // 获取状态:
    public String getState() {
        return buffer.toString();
    }

    // 恢复状态:
    public void setState(String state) {
        this.buffer.delete(0, this.buffer.length());
        this.buffer.append(state);
    }
}
```



### 观察者 Observer

**定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新**

```java
public class Store {
    private List<ProductObserver> observers = new ArrayList<>();
    private Map<String, Product> products = new HashMap<>();

    // 注册观察者:
    public void addObserver(ProductObserver observer) {
        this.observers.add(observer);
    }

    // 取消注册:
    public void removeObserver(ProductObserver observer) {
        this.observers.remove(observer);
    }

    public void addNewProduct(String name, double price) {
        Product p = new Product(name, price);
        products.put(p.getName(), p);
        // 通知观察者:
        observers.forEach(o -> o.onPublished(p));
    }

    public void setProductPrice(String name, double price) {
        Product p = products.get(name);
        p.setPrice(price);
        // 通知观察者:
        observers.forEach(o -> o.onPriceChanged(p));
    }
}
```

为了能够使得观察者类型进行无限扩充，使用该模式

在消息中间件中常用观察者模式/发布订阅模式



### 状态 State

```java
public class BotContext {
	private State state = new DisconnectedState();

	public String chat(String input) {
		if ("hello".equalsIgnoreCase(input)) {
            // 收到hello切换到在线状态:
			state = new ConnectedState();
			return state.init();
		} else if ("bye".equalsIgnoreCase(input)) {
            /  收到bye切换到离线状态:
			state = new DisconnectedState();
			return state.init();
		}
		return state.reply(input);
	}
}
```

实际上状态转换可以交给State的具体类来进行，State内定义状态转换方法，对特定输入转换到另一State，返回转换后的State



### 策略 Stragety

**定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换**

```java
public class DiscountContext {
    // 持有某个策略:
    private DiscountStrategy strategy = new UserDiscountStrategy();

    // 允许客户端设置新策略:
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    public BigDecimal calculatePrice(BigDecimal total) {
        return total.subtract(this.strategy.getDiscount(total)).setScale(2);
    }
}

public interface DiscountStrategy {
    // 计算折扣额度:
    BigDecimal getDiscount(BigDecimal total);
}

public class UserDiscountStrategy implements DiscountStrategy {
    public BigDecimal getDiscount(BigDecimal total) {
        // 普通会员打九折:
        return total.multiply(new BigDecimal("0.1")).setScale(2, RoundingMode.DOWN);
    }
}

public class OverDiscountStrategy implements DiscountStrategy {
    public BigDecimal getDiscount(BigDecimal total) {
        // 满100减20优惠:
        return total.compareTo(BigDecimal.valueOf(100)) >= 0 ? BigDecimal.valueOf(20) : BigDecimal.ZERO;
    }
}
```

基本思想就是通过动态设置stragety，将各种操作交给具体stragety实现，完成操作的特异性



### 模板方法 Template Method

**定义一个操作中的算法的骨架，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤**

```java
public abstract class AbstractSetting {
    public final String getSetting(String key) {
        String value = lookupCache(key);
        if (value == null) {
            value = readFromDatabase(key);
            putIntoCache(key, value);
        }
        return value;
    }

    protected abstract String lookupCache(String key);

    protected abstract void putIntoCache(String key, String value);
}
```

通过子类实现对lookupCache和putIntoCache方法的实现，即可完成整体实现

**核心思想：**父类定义骨架，子类实现各种细节

- 或者说父类定义流程，子类实现流程细节



### 访问者 Visitor

**表示一个作用于某对象结构中的各元素的操作**

- **核心思想：**为了访问比较复杂的数据结构，不去改变数据结构，而是把对数据的操作抽象出来，在“访问”的过程中以回调形式在访问者中处理操作逻辑
- 也就是将操作交给具体Visitor进行，而不是让调用者自行进行

