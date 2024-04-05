---
title: "Interview Spring"
date: 2024-02-28
draft: true
---

### Spring

Spring框架是一个开源的Java应用程序框架，旨在简化企业级应用的开发。它提供了全面的基础设施支持，包括依赖注入、面向切面编程、事务管理等功能，有助于提高应用程序的可维护性、可扩展性和灵活性。

使用Spring框架带来以下好处：

1. **简化开发：** Spring提供了丰富的功能和组件，可以减少样板代码，简化企业级应用的开发过程。
2. **依赖注入：** Spring支持依赖注入，通过IOC容器管理对象之间的依赖关系，降低了组件之间的耦合性。
3. **面向切面编程（AOP）：** 允许将横切关注点（如日志记录、事务管理）从业务逻辑中分离出来，提高代码的模块化和可维护性。
4. **事务管理：** 提供声明式事务管理，简化了处理数据库事务的复杂性，确保数据一致性。
5. **模块化开发：** 使用Spring可以构建模块化的应用，使得每个模块更易于测试、维护和替换。
6. **集成支持：** 提供对各种数据源、消息队列、缓存等的集成支持，使得整合外部服务变得更加容易。
7. **测试支持：** Spring框架鼓励编写可测试的代码，提供了对单元测试和集成测试的支持。



### 依赖注入（DI）、控制反转（IoC）

依赖注入（Dependency Injection，DI）是一种设计模式，在依赖注入中，一个对象并不负责获取或创建它依赖的对象，而是由外部容器负责将依赖的对象注入（传递）给客户端。这样，对象之间的关系由外部容器管理，而不是在对象内部直接硬编码。

控制反转（Inversion of Control，IoC）是一种更广泛的概念，依赖注入是IoC的一种具体实现方式。在IoC中，对象的控制权发生了反转，不再由对象自身负责管理其依赖和执行流程，而是由外部容器（通常是框架）来控制。这种反转让应用的架构更加灵活、可扩展，同时降低了组件之间的紧耦合。

> 除了依赖注入，IoC 还有另外一种主要的实现方式，即服务定位（Service Locator）。在服务定位中，对象通过向服务定位器（Service Locator）请求所需的服务或依赖，而不是直接持有对这些服务或依赖的引用。服务定位器充当中介，负责定位和提供所需的服务。这种方式在某些情况下可以实现控制反转，但相对于依赖注入，它通常更容易导致代码紧密耦合，难以进行单元测试，以及难以理解和维护。



### Spring的依赖注入

Spring框架实现依赖注入主要通过以下方式：

1. **构造器注入：** Spring通过构造函数将依赖注入到目标对象。在目标类的构造函数中声明需要的依赖，Spring容器会负责在实例化目标对象时提供这些依赖。
2. **Setter注入：** 通过setter方法将依赖注入到目标对象。Spring容器在实例化对象后，调用相应的setter方法为对象设置依赖。
3.  **接口注入：** 通过实现接口并在接口中定义注入方法，Spring容器调用这个方法进行依赖注入。



### Spring的IoC容器

`org.springframework.beans` 和 `org.springframework.context` 包是Spring Framework的IoC容器的基础：

1. `org.springframework.beans`： 这个包提供了关于bean的基本功能，包括`BeanWrapper`（用于包装bean实例）、`PropertyValue`（用于封装属性值）等用于 bean 处理的基础工具类。核心接口是 `BeanFactory`，定义了IoC容器的基本功能，包括获取bean、检查bean是否存在等。还包含了与 bean 定义和配置相关的核心功能。主要的接口是 `BeanDefinition`，它描述了 bean 的定义，包括类名、作用域、属性值等。其他与 bean 配置和定义相关的接口和类也位于这个包中，如 `ConfigurableBeanFactory`、`BeanPostProcessor` 等。
2. `org.springframework.context`： 这个包提供了应用程序级别的上下文支持，包括应用程序事件、国际化、资源加载等。`ApplicationContext` 接口是这个包的核心，它扩展了 `BeanFactory` 接口，并提供了更多的功能，如AOP集成、事件传播、国际化等。`WebApplicationContext` 是 `ApplicationContext` 的子接口，专门用于Web应用程序。



### Bean Scope

| Scope                                                        | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://springdoc.cn/spring/core.html#beans-factory-scopes-singleton) | 单例模式。这是Spring中Bean的默认Scope。每个Spring容器中只有一个Bean实例，所有对该Bean的请求都将返回同一个实例。 |
| [prototype](https://springdoc.cn/spring/core.html#beans-factory-scopes-prototype) | 原型模式。每次请求该Bean时都会创建一个新的实例，每个实例都是独立的。这意味着每次通过容器获取Bean时，都会得到一个新的对象。 |
| [request](https://springdoc.cn/spring/core.html#beans-factory-scopes-request) | 这个Scope类型只适用于Web应用程序，并且通常与 `XmlWebApplicationContext` 共同使用。它的作用域限定在一个HTTP请求内。也就是说，每个HTTP请求都会有一个新的Bean实例，这些实例在请求结束时会被销毁。 |
| [session](https://springdoc.cn/spring/core.html#beans-factory-scopes-session) | 这个Scope也仅适用于Web应用程序。它的作用域限定在一个HTTP会话内。这意味着每个用户会话都会有一个新的Bean实例，这些实例在会话结束时会被销毁。 |

##### 结合生命周期机制

从Spring 2.5开始，你有三个选项来控制Bean的生命周期行为。

- [`InitializingBean`](https://springdoc.cn/spring/core.html#beans-factory-lifecycle-initializingbean) 和 [`DisposableBean`](https://springdoc.cn/spring/core.html#beans-factory-lifecycle-disposablebean) callback 接口。
- 自定义 `init()` and `destroy()` 方法。
- [`@PostConstruct` 和 `@PreDestroy` 注解](https://springdoc.cn/spring/core.html#beans-postconstruct-and-predestroy-annotations)。你可以结合这些机制来控制一个特定的Bean。

为同一个Bean配置的多个生命周期机制，具有不同的初始化方法，其调用方式如下。

1. 注解了 `@PostConstruct` 的方法。
2. `afterPropertiesSet()`，如 `InitializingBean` 回调接口所定义。
3. 一个自定义配置的 `init()` 方法。

销毁方法的调用顺序是一样的。

1. 注解了 `@PreDestroy` 的方法。
2. `destroy()`，正如 `DisposableBean` 回调接口所定义的那样。
3. 一个自定义配置的 `destroy()` 方法。



### BeanFactory 和 ApplicationContext

`BeanFactory` 和 `ApplicationContext` 都是在Spring框架中用于管理和组织Bean（对象）的容器。`BeanFactory` 是 Spring 框架的基础设施，面向 Spring 本身；`ApplicationContext` 面向使用Spring 框架的开发者，几乎所有的应用场合我们都直接使用 `ApplicationContext` 而非底层的 `BeanFactory`。

主要区别在于功能和特性：

1. **加载时机：**`BeanFactory` 是延迟加载的，只有在获取Bean时才会进行实例化，节省内存。而`ApplicationContext` 在容器启动时就进行了实例化和初始化，提前检测和解析Bean。
   
2. **功能差异**：`ApplicationContext` 是 `BeanFactory` 的子接口，因此它继承了 `BeanFactory` 的所有功能，包括Bean的获取、类型判断等。但除此之外，`ApplicationContext` 还提供了许多 `BeanFactory` 没有的功能，如国际化支持、资源访问、事件传播等。这使得 `ApplicationContext` 更适合用于开发者的日常开发，因为它提供了更完整的框架功能。
   
3. **自动装配：**`ApplicationContext` 具有自动装配的特性，能够更方便地实现依赖注入。`BeanFactory` 需要手动配置依赖关系。

以下是几种较常见的 `ApplicationContext` 实现方式：

1. `ClassPathXmlApplicationContext`：从 类路径（classpath）中加载XML配置文件，并创建相应的ApplicationContext 实例。类路径通常指的是包含项目类文件的目录，如`src/main/resources`目录下的文件。

   ```java
   ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”); 
   ```

2. `FileSystemXmlApplicationContext`：由文件系统中的 XML 配置文件读取上下文。

   ```java
   ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
   ```

3. `XmlWebApplicationContext`：用于加载Web应用程序的上下文配置文件。在Web应用中，`XmlWebApplicationContext` 通常被配置为ServletContext监听器，在Web应用启动时自动加载上下文配置文件。配置文件通常位于Web应用的`/WEB-INF`目录下。

4. `AnnotationConfigApplicationContext`：用于加载基于Java注解的配置，通常包含了一个或多个`@Configuration`注解的类，并且可能还包含`@Bean`、`@ComponentScan`、`@PropertySource`等其他注解来进一步配置Spring容器。

[BeanFactory 还是 ApplicationContext?](https://springdoc.cn/spring/core.html#context-introduction-ctx-vs-beanfactory)



### ApplicationContextAware 和 BeanNameAware

接口 Aware，在 Spring 框架中它是一种感知标记性接口，具体的子类定义和实现能感知容器中的相关对象。

当 `ApplicationContext` 创建一个实现 `org.springframework.context.ApplicationContextAware` 接口的对象实例时，该实例被提供给该 `ApplicationContext` 的引用。下面的列表显示了 `ApplicationContextAware` 接口的定义。

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

当 `ApplicationContext` 创建一个实现 `org.springframework.beans.factory.BeanNameAware` 接口的类时，该类被提供给其相关对象定义中定义的名称的引用。下面的列表显示了 `BeanNameAware` 接口的定义。

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

这个回调是在正常的Bean属性之后，但在 `InitializingBean.afterPropertiesSet()` 或自定义 `init-method` 等初始化回调之前调用的。



### BeanPostProcessor 和 BeanFactoryPostProcessor

在Spring框架中，`BeanPostProcessor`和`BeanFactoryPostProcessor`是用于在Spring容器创建和初始化bean的过程中执行一些自定义的逻辑：

1. `BeanPostProcessor`
   `BeanPostProcessor` 实例对Bean（或对象）实例进行操作。`BeanPostProcessor`接口中的方法允许你在bean的初始化前后进行定制处理。`postProcessBeforeInitialization`方法在bean的属性设置完成之后、但在bean的自定义初始化方法（如果有的话）之前被调用。`postProcessAfterInitialization`方法则在bean的自定义初始化方法之后被调用。

   你可以实现这个接口来修改bean的属性、返回代理对象等。例如，Spring AOP就是通过`BeanPostProcessor`实现的，它在bean初始化后为其添加切面逻辑。 `@PostConstruct` 和  `@PreDestroy` 也是通过`BeanPostProcessor`实现的。

2. `BeanFactoryPostProcessor`
   要改变实际的Bean定义（即定义Bean的蓝图），你需要使用 `BeanFactoryPostProcessor`， `BeanFactoryPostProcessor`接口允许你在Spring容器中的所有bean定义加载完成，但bean实例还未创建之前进行后处理。它的`postProcessBeanFactory`方法会在这个阶段被调用。

   通常，`BeanFactoryPostProcessor`用于修改或添加bean定义，或者在bean定义加载之后执行一些初始化逻辑。例如，`PropertyPlaceholderConfigurer`就是一个`BeanFactoryPostProcessor`实现类，用于解析bean定义中的占位符。



### FactoryBean

你可以为那些本身就是工厂的对象实现 `org.springframework.beans.factory.FactoryBean` 接口。

`FactoryBean` 接口是Spring IoC容器实例化逻辑的一个可插入点。如果你有复杂的初始化代码，最好用Java来表达，而不是用（潜在的）冗长的XML来表达，你可以创建自己的 `FactoryBean`，将复杂的初始化写入该类中，然后将你的自定义 `FactoryBean` 插入容器中。

`FactoryBean<T>` 接口提供三个方法。

- `T getObject()`: 返回本工厂创建的对象的一个实例。该实例可能会被共享，这取决于该工厂是返回singleton还是prototype。
- `boolean isSingleton()`: 如果这个 `FactoryBean` 返回 singleton，则返回 `true`，否则返回 `false`。这个方法的默认实现会返回 `true`。
- `Class<?> getObjectType()`: 返回由 `getObject()` 方法返回的对象类型，如果事先不知道类型，则返回 `null`。



### Spring Bean的扩展点

当然可以，以下是按照Spring Bean生命周期的顺序整理的关于Bean的扩展点：

**Spring Bean生命周期扩展点（按生命周期顺序）：**

1. **BeanDefinitionRegistryPostProcessor**：
   - 这是一个在Bean定义被加载到Spring容器之前执行的扩展点。它允许开发者在Bean定义注册之前和之后进行自定义处理，例如动态添加、修改或删除Bean定义。

2. **BeanFactoryPostProcessor**：
   - 在Bean定义加载到BeanFactory后，但Bean实例创建之前执行。这允许开发者在Bean实例化之前修改Bean定义，例如添加属性、覆盖现有Bean定义等。

3. **MergedBeanDefinitionPostProcessor**：
   - 在Bean定义合并后执行，允许开发者在Bean定义被合并到最终的BeanDefinitionHolder后进行自定义处理。

4. **InstantiationAwareBeanPostProcessor**：
   - 在Bean实例化前后执行，可以控制Bean的实例化过程。开发者可以自定义Bean的实例化逻辑，例如使用特定的构造器或方法。

5. **BeanPostProcessor**：
   - 在Bean实例化后、初始化前后执行。允许开发者在Bean初始化之前或之后添加自定义逻辑，比如修改Bean的属性、调用额外的初始化方法等。

6. **InitializingBean** 或 `@PostConstruct`：
   - 在Bean的依赖注入完成后立即执行。这是Bean初始化回调的一个方式，允许开发者执行自定义的初始化逻辑。

7. **AOP（面向切面编程）**：
   - 在Bean初始化之后，通过AOP代理增强Bean的功能。Spring AOP允许开发者在不修改现有代码的情况下，在Bean的方法执行前后添加额外的逻辑。

8. **Bean的使用**：
   - 在Bean的生命周期中，Bean会被Spring容器管理，并提供给其他Bean使用。在这个阶段，Bean可以执行其业务逻辑。

9. **DestructionAwareBeanPostProcessor** 或 `@PreDestroy`：
   - 在Bean销毁前执行。这允许开发者在Bean的生命周期结束时执行清理操作或执行其他必要的逻辑。

10. **DisposableBean**：
    - 如果Bean实现了`DisposableBean`接口，那么在容器关闭时会调用其`destroy`方法，用于执行清理操作。

