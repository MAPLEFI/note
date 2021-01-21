IOC:(Inversion Of Control)控制反转

控制：资源的获取方式
      
```
主动式：（要什么资源都自己创建即可）
BookServlet{
    BookService bs=new BookService();
    比如AirPlane ap=new AirPlane();//其无参构造功能补全，复杂对象是比较庞大的工程
}
被动式：资源的获取不是我们自己创建，而是交给一个容器来创建和设置

管理所有的组件（有功能的类）：假设BookServlet受容器管理，BookService也受容器管理；容器可以自动探查出哪些组件需要用到 另一些组件，容器帮我创建BookService对象并把BookService对象赋值过去

容器：由主动的new资源变为被动 的接受资源比如
@Autowired
BookService bk;
只要BookService已经注入容器就可以这么搞
```

DI:(Dependency Injection)依赖注入：容器能知道哪个组件（类）运行的时候，需要另外一个组件；容器通过反射的形式，将容器中准备好的BookService注入对其赋值

Spring框架写配置是使用xml来写的，即写入容器注册需要在xml中进行注册，Bean标签可以注册一个组件(对象类)
```
<bean>标签：
class：写要注册的组件的全类名比如class="com.atguigu.bean,Person"
id：这个对象的唯一标识，比如<bean id="person01"></bean>

使用<property>标签为Person对象的属性赋值，写在<bean></bean>之间即
<bean>
<property name="lastname" value="zhangsan">
</bean>
其中name指定属性名，value指定属性的值，value的值都必须用""括起来
```

ApplicationContext:代表ioc容器

ClassPathXmlApplicationContext:当前应用的XMl文件在ClassPath下的那个位置

ApplicationContext ioc=new ClassPathXmlApplicationContext("ioc.xml")//根据spring的配置文件得到ioc容器对象

得到ioc容器对象之后我们就可以使用ioc.getBean("id")得到对应xml里面的对应的id代表的类对象比如 Person person=(Person)ioc.getBean("pserson01")



创建带有 生命周期方法的bean:在对应的xml里面的bean标签里面添加属性destroy-method="" init-method=""分别表示销毁方法和初始化方法
单实例Bean的生命周期:容器启动-》自动调用初始化方法-》关闭容器-》自动调用销毁方法

多实例Bean的生命周期：容器启动-》运行到需要获得对象时调用初始化方法-》关闭容器-》什么时候获取完什么时候调用销毁方法

后置处理器BeanPostProcessor，可以在bean初始化前和 初始化后自动调用
注意在初始化前和初始化后都一定返回bean不要返回null否则初始化后或者使用结束后的结果就被偷梁换柱成Null了

1.编写后置处理器的实现类 
2.将后置处理器 注册在配置文件中

无论bean是否有初始化方法后置处理器都会默认其有初始化并工作

有关context:component-scan标签
其有一个属性为base-package=""表示其起作用的包

在其中可以有context:exclude-filter表示扫描的时候可以排除一些不去扫描，其中有属性为type="annotation"即指定有规定注解的类不去扫描，后面跟expression=""表示具体哪一个注解的全类名，还有属性为type="assignable"表示指定 某个类不去扫描，后面跟expression具体类的全类名

还可以有context:include-filter标签标识只扫描哪些，其中有属性type="annotation"标识只扫描加了哪些注解的类，后面的expression写注解的全类名，也有type="assignable"表示只扫描哪一些类，后面的expression写具体哪一个类的全类名

@Autowired自动赋是去容器中找对应的组件，@Autowired原理 ：

```
第一步先看属性类型是什么，先按照类型去容器中找到对应的组件，三种情况：找到一个就赋值，没有找到就抛出异常，
如果找到多个（即比如找BookService但是又有一个类是BookService的子类比如BookServiceImpt）按照对象名作为id继续匹配,比如自动填充BookService bookService那么即使找到BookService(匹配bookService)

BookServiceImpt(匹配bookServiceImpt)也会按照id bookService来进行匹配还是会匹配到BookService类的

当然可以在@Autowired的注解上加上一个注解@Qualifier("")即指定一个名字作为id来进行匹配而不是使用对象名作为id来匹配
```
就作用上来说@Autowired和@Resource是差不多得，但是区别在于@Resource是java标准而@Autowired是spring框架自带也就是说只有spring才能用，而@Resource扩展性更强如果切换成另一个容器框架也能使用而@Autowired就不行了

就算是注入泛型父类一样会很智能的注入该有的类型



总结：ioc是一个容器帮我们管理所有的组件

1.依赖注入：@Autowired自动注入

2.某个组件要使用Spring提供的更多功能（IOC,AOP）必须加入到容器中

容器启动。创建所有单实例bean

@Autowired自动装配的时候是从容器中找到这些符合要求的bean

ioc.getBean("")也是从容器中找到这个bean

容器中包括了所有的bean

调试spring源码，容器到底是什么？其实就是一个map,这个map保存所有创建好的bean并提供外界获取功能

探索单实例的bean都保存到那个map中

源码调试思路从helloworld开始：给helloworld每一个关键步骤打上断点进去看里面都做了些什么，怎么知道哪些方法都是干什么的，翻译这个方法是干什么的，放行这个方法，看控制台怎么打印的或者看方法注释


