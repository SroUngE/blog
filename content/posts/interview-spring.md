---
title: "Interview Spring"
date: 2024-02-28
draft: false
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

依赖注入（Dependency Injection，DI）是一种设计模式，它的目标是减少组件之间的耦合度。在依赖注入中，一个对象并不负责获取或创建它依赖的对象，而是由外部容器负责将依赖的对象注入（传递）给客户端。这样，对象之间的关系由外部容器管理，而不是在对象内部直接硬编码。

控制反转（Inversion of Control，IoC）是一种更广泛的概念，依赖注入是IoC的一种具体实现方式。在IoC中，对象的控制权发生了反转，不再由对象自身负责管理其依赖和执行流程，而是由外部容器（通常是框架）来控制。这种反转让应用的架构更加灵活、可扩展，同时降低了组件之间的紧耦合。

> 除了依赖注入，IoC 还有另外一种主要的实现方式，即服务定位（Service Locator）。在服务定位中，对象通过向服务定位器（Service Locator）请求所需的服务或依赖，而不是直接持有对这些服务或依赖的引用。服务定位器充当中介，负责定位和提供所需的服务。这种方式在某些情况下可以实现控制反转，但相对于依赖注入，它通常更容易导致代码紧密耦合，难以进行单元测试，以及难以理解和维护。

### Spring的依赖注入

Spring框架实现依赖注入主要通过以下方式：

1. **构造器注入：** Spring通过构造函数将依赖注入到目标对象。在目标类的构造函数中声明需要的依赖，Spring容器会负责在实例化目标对象时提供这些依赖。
2. **Setter注入：** 通过setter方法将依赖注入到目标对象。Spring容器在实例化对象后，调用相应的setter方法为对象设置依赖。
3.  **接口注入：** 通过实现接口并在接口中定义注入方法，Spring容器调用这个方法进行依赖注入。

### Spring的IoC容器

Spring的IoC容器实现了通过IoC容器管理对象的生命周期和依赖关系，将对象的创建、初始化、销毁等控制权交给容器，使得开发者能够更专注于业务逻辑，而不必过多关注对象的创建和管理。IoC容器根据配置信息，实例化和管理bean，同时负责注入依赖关系，从而实现了控制反转。

1. **`org.springframework.beans` 包：** 这个包提供了关于bean的基本功能，包括`BeanWrapper`（用于包装bean实例）、`PropertyValue`（用于封装属性值）等用于 bean 处理的基础工具类。
2. **`org.springframework.beans.factory` 包：** 在Spring框架中，这个包扩展了`org.springframework.beans` 包，提供了关于bean工厂的更多功能。核心接口是 `BeanFactory`，定义了IoC容器的基本功能，包括获取bean、检查bean是否存在等。
3. **`org.springframework.beans.factory.config` 包：** 这个包包含了与 bean 定义和配置相关的核心功能。主要的接口是 `BeanDefinition`，它描述了 bean 的定义，包括类名、作用域、属性值等。其他与 bean 配置和定义相关的接口和类也位于这个包中，如 `ConfigurableBeanFactory`、`BeanPostProcessor` 等。
4. **`org.springframework.context` 包：** 这个包提供了应用程序级别的上下文支持，包括应用程序事件、国际化、资源加载等。`ApplicationContext` 接口是这个包的核心，它扩展了 `BeanFactory` 接口，并提供了更多的功能，如AOP集成、事件传播、国际化等。`WebApplicationContext` 是 `ApplicationContext` 的子接口，专门用于Web应用程序。

### BeanFactory 和 ApplicationContext

`BeanFactory` 和 `ApplicationContext` 都是在Spring框架中用于管理和组织Bean（对象）的容器。`BeanFactory` 是 Spring 框架的基础设施，面向 Spring 本身；`ApplicationContext` 面向使用Spring 框架的开发者，几乎所有的应用场合我们都直接使用 `ApplicationContext` 而非底层的 `BeanFactory`。

主要区别在于功能和特性：

1. **加载时机：**
   - `BeanFactory`是延迟加载的，只有在获取Bean时才会进行实例化。
   - `ApplicationContext`在容器启动时就进行了实例化和初始化，提前检测和解析Bean。

2. **性能：**
   - `BeanFactory`相对较轻量，适用于资源受限的环境。
   - `ApplicationContext`提供了更多的企业级功能，但相对而言更重量级。

3. **自动装配：**
   - `ApplicationContext`具有自动装配的特性，能够更方便地实现依赖注入。
   - `BeanFactory`需要手动配置依赖关系。

以下是几种较常见的 `ApplicationContext` 实现方式：

1. `ClassPathXmlApplicationContext` ：从 classpath 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中

   ```java
   ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”); 
   ```

2. `FileSystemXmlApplicationContext` ：由文件系统中的 XML 配置文件读取上下文。

   ```java
   ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
   ```

3. `XmlWebApplicationContext` ：由 Web 应用的 XML 文件读取上下文。

4. `AnnotationConfigApplicationContext` ：(基于 Java 配置启动容器)
