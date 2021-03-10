# 动态代理
Java 动态代理作用是什么？ - bravo1988的回答 - 知乎
https://www.zhihu.com/question/20794107/answer/658139129

# spring 启动过程
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5dcb4ca2d084864a6a90e0099688fe2~tplv-k3u1fbpfcp-zoom-1.image)

## beanDifinition
用于描述一个 bean 实例, 包装一个对象, ==使其能被 beanFactory 接收并处理==
![](https://pic2.zhimg.com/80/v2-7ea61ed98aede9409e1ffec392a4f893_1440w.jpg?source=1940ef5c)
![](https://pic4.zhimg.com/80/v2-f54704d1f6b9ef1ee47e37e0040a9b16_1440w.jpg?source=1940ef5c)
**举例:** 
1. 该 bean 是否是单例的
2. 是否懒加载
3. 需要调用哪个初始化方法/销毁方法

## 后置处理器
![](https://pic4.zhimg.com/80/v2-9bd6efe7c86130553896c3744c338778_1440w.jpg?source=1940ef5c)
![](https://pic4.zhimg.com/80/v2-c37f32ba1b6562e05772dcad2262880a_1440w.jpg?source=1940ef5c)

## beanFactoryPostProcessor 作用
可以对 bean 进行定制化的操作, 比如对 bean 的属性进行修改或添加.
发生在 bean 被读取进来, 但还没有初始化的时候, 此时传入的参数是一个 beanFactory, 我们可以从中获取到特定的 beanDefinition
```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
 
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("调用MyBeanFactoryPostProcessor的postProcessBeanFactory");
        BeanDefinition bd = beanFactory.getBeanDefinition("myJavaBean");
        System.out.println("属性值============" + bd.getPropertyValues().toString());
        MutablePropertyValues pv =  bd.getPropertyValues();  
        if (pv.contains("remark")) {  
            pv.addPropertyValue("remark", "把备注信息修改一下");  
        }  
        bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);
    }
}
```

## BeanPostProcessor 作用
支持在 bean 的初始化前后, 对 bean 进行定制化
**举例:**
我们希望在普通Bean中注入 ApplicationContext (不使用 autowired)
1. 可以让Bean实现ApplicationContextAware接口
![](https://pic1.zhimg.com/80/v2-4a629f6860a9230a64b248d70c0b34c6_1440w.jpg?source=1940ef5c)
2. 在初始化 bean 时, 遍历所有的 beanPostProcessor, 此时有一个 ApplicationContextProcessor, 他会检查类是否继承了 ApplicationContextAware, 如果是, 则调用 setApplicationContext 方法
![](https://pic2.zhimg.com/80/v2-dc8000e96551247ddb182edc8f875e1f_1440w.jpg?source=1940ef5c)


## bean 的生命周期
![](https://cdn.nlark.com/yuque/0/2019/png/181910/1549528833203-01b9a061-1535-454c-8e9c-e31cf3c8e06a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_eXVsb25nc3Vu%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![](https://pic2.zhimg.com/v2-c7469843ef8142962a42df0a9ba21505_r.jpg)
## beanFactory 和 factoryBean
### beanFactory
beanFactory 本质上是一个容器, sping IOC 容器的核心接口. 职责是==实例化、定位、配置应用程序中的对象及建立这些对象间的依赖==. applicationContext 就是继承了 beanFactory
**使用场景:**
1. 从 IOC 容器中获取 bean `getBean()`
2. 检查是否包含指定的 bean `containsBean()`
3. 判断是否单例 `isSingleton()` 

### factoryBean
1. 首先是一个 bean, 因此也是在 beanFactory 容器中, 同时又是一个工厂, 并且支持泛型, 能够在需要时返回任何一个对象, ==创建的对象是否单例由 isSingleton() 方法决定==
2. 如果一个 bean A 实现了 factoryBean 接口, 那么这个 bean A 就变成了一个工厂, 它有一个 target 的属性, 代表着他实际生产的对象
3. 当使用容器的 getBean() 方法, 实际上是调用了 factoryBean 的getObject() 方法, 因此返回并不是 bean A 本身, 如果想返回 beanA, 那么须在getBean 的参数名字前面 `'&'`

**使用场景:**
一般情况下，Spring通过反射机制利用`<bean>`的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在`<bean>`中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案