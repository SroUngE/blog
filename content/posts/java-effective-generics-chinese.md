---
title: "[翻译]Java高效泛型——Effective Generics"
description: "翻译《Java Generics and Collections》这本书的第8章节Effective Generics，其中介绍了一些关于如何在代码实践中有效使用泛型的建议。"
date: 2023-11-26
tags: ["Java"]
featuredImage: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
draft: false
---

<!--more-->


# CHAPTER 8 Effective Generics

本章节包含如果在实践编程中有效使用泛型的建议。我们会讨论到受检的集合，安全问题，特殊类和字节码兼容等方面。本章的标题源自对 Joshua Bloch’s book, Effective Java(Addison-Wesley)的致敬。

注：  
checked collection: 这里的意思就是声明了明确泛型的集合，因为泛型会收到编译器检查，所以叫受检查的集合，例如 List<String>。
specialized class：

## 8.1 谨慎调用遗留代码
我们都知道，泛型会在编译时被检查而不是运行时。通常，这就是我们想要的效果，因为在编译时去检查能更早地报告错误，并且不会导致运行时的开销。然而，有时候这并不适用，比如当我们无法保证运行时的检查是完全充分的（当我们要传入一个参数化类型的实例到另一个遗留的客户端或者是到一个我们并不信任的客户端），或者当我们需要获取在运行时的类型信息时（比如我们想要使用可具体化类型的数组）。通常 checked collection 可以解决问题，而当它无法解决上述这些问题的时候，我们可以创建 specialized class。这一小节我们会讨论 checked collections，关于安全性问题会在下一小节讨论，specialized class 会在这之后被讨论。

来看这个旧库，addItems 方法用于给输入的 List 添加某些元素，getItems 用于得到一个新的包含某些元素的 List。

```java
class LegacyLibrary {
    public static void addItems(List list) {
        list.add(new Integer(1)); list.add("two");
    }
    public static List getItems() {
        List list = new ArrayList();
        list.add(new Integer(3));
        list.add("four");
        return list;
    }
}
```

现在假设有一个客户端使用这个旧库，它被（错误地）告知 List 里的所有元素都是整数：

```java
class NaiveClient {
    public static void processItems() {
        List<Integer> list = new ArrayList<Integer>();
        Legacy Library.addItems(list);
        List<Integer> list2 = LegacyLibrary.getItems(); // unchecked
        // sometime later ...
        int s = 0;
        for (int i : list) s += i; // class cast exception
        for (int i : list2) s += i; // class cast exception
    }
}
```

把 List<Integer> 传到 addItems 方法的时候没有任何警告，因为 List<Integer> 被认为是 List 的子类型。把 getItems 返回的 List 转为 List<Integer> 时，它会提示一个 unchecked warning。在运行时，当尝试从 list 里面拿数据的时候，就会引发一个类型转换异常，因为由擦除隐式插入的转为 Integer 的类型转换会失败。（这种类型转换的失败并不与 cast-iron guarantee 相违背，因为 cast-iron guarantee 在遗留代码或者 unchecked warnings 面前不成立）。因为出现异常的这行代码离添加 string 到 list 的代码很远，所以这个 bug 很难被精确定位。

如果旧库通过应用最小更改或存根技术生成，只要正确分配了泛型类型，就不会出现这些问题。

一个不那么天真的客户端可能会设计代码来尽早地捕获错误，并且也能容易 debug。

```java
class WaryClient {
    public static void processItems() {
        List<Integer> list = new ArrayList<Integer>();
        List<Integer> view = Collections.checkedList(list, Integer.class);
        LegacyLibrary.addItems(view); // class cast exception
        List<Integer> list2 = LegacyLibrary.getItems(); // unchecked
        for (int i : list2) {} // class cast exception
        // sometime later ...
        int s = 0;
        for (int i : list) s += i;
        for (int i : list2) s += i;
    }
}
```

Collections类里的checkedList方法接受一个list和一个class返回一个checked view。每当尝试往这个checked view添加元素的时候，在把元素加到list之前会通过反射来检查这个元素是否属于指定类型。使用 checked view 的话，每当尝试往list里添加string的时候，会导致类型转换异常发生在 addItems 方法内部。但是对于 getItems 方法，它创建了自己的list，因为客户端无法以同样的方式使用包装器。然而，在这时候可以对返回的list添加一个空的循环，以保证在靠近违规方法调用的地方捕获异常。

只有当list的元素是可具体化的类型时，Checked lists才能提供有用的保证。如果你想要应用这些技术当即使这个list不是一个可具体化类型时，你可能需要考虑应用8.3节的专门化技术。

## 8.2 使用 Checked Collections 来强制保证安全性
很重要的一点是要意识到由泛型提供的担保只适用于没有 unchecked warnings的时候。这意味着泛型对于别人写的代码是无法保证安全性的，因为你没办法知道当编译后这个代码是否会引起 unchecked warnings。

假设我们有一个定义了订单的 Order类，还有一个定义了有效订单的 AuthenticatedOrder子类：
```java
class Order { ... }
class AuthenticatedOrder extends Order { ... }
```
接口定义了订单的供应商和处理器。这里供应商被要求只能提供验证过的订单，而处理器则能处理所有类型的订单：
```java
interface OrderSupplier {
    public void addOrders(List<AuthenticatedOrder> orders);
}
interface OrderProcessor {
    public void processOrders(List<? extends Order> orders);
}
```
从所涉及到的类型来看，你可能会认为以下代理商保证了只有验证过的订单才能从供应商传到处理器中：
```java
class NaiveBroker {
    public void connect(OrderSupplier supplier, OrderProcessor processor) {
        List<AuthenticatedOrder> orders = new ArrayList<AuthenticatedOrder>();
        supplier.addOrders(orders);
        processor.processOrders(orders);
    }
}
```
但是事实上，一个狡猾的供应商可能提供未经验证的订单：
```java
class DeviousSupplier implements OrderSupplier {
        public void addOrders(List<AuthenticatedOrder> orders) {
        List raw = orders;
        Order order = new Order(); // not authenticated
        raw.add(order); // unchecked call
    }
}
```
编译 DeviousSupplier 将会发出一个 unchecked warning，然而 NaiveBroker 无从得知这一点。
使用不当也会导致和迂回一样多的问题。任何在编译时发出 unchecked warnings 的代码都可能导致类似的问题，可能仅仅是因为写代码的人犯了错误。像上节提到的一样，尤其是遗留代码可能引起这样的问题。
正确的解决方法是让经纪人将 checked list 传递给 OrderSupplier。
```java
class WaryBroker {
    public void connect(OrderSupplier supplier, OrderProcessor processor) {
        List<AuthenticatedOrder> orders = new ArrayList<AuthenticatedOrder>();
        supplier.addOrders(
            Collections.checkedList(orders, AuthenticatedOrder.class)
        );
        processor.processOrders(orders);
    }
}
```
现在，如果 OrderSupplier 试图添加任何不是验证的订单到列表中，就会引起一个类型转换异常。Checked collections 并不是保证安全性的唯一技术。if OrderSupplier 返回一个 list 而不是接收一个 list，那么经纪人就可以，在传递订单之前，用上节提到的空循环技术来保证这个列表只包括验证的订单。我们还可以使用专门化，如下一节所述，来创建一个只能包含已授权订单的特殊类型的列表。

## 8.3 专门处理可具体化的类型
参数化类型不是可具体化的，而有些操作，比如实例的类型检测和转换以及数组创建只适用于具体化类型。在这种情况下，一种解决方法是创建一个参数类型的专用版本。专业化版本可以由委派(也就是包装类)或继承(也就是子类化)创建，我们接下来会轮流讨论。

我们首先把 List 接口专门化为所需类型：
```java
interface ListString extends List<String> {}

class ListStrings {
    public static ListString wrap(final List<String> list) {
        class Random extends AbstractList<String> implements ListString, RandomAccess {
            public int size() { return list.size(); }
            public String get(int i) { return list.get(i); }
            public String set(int i, String s) { return list.set(i,s); }
            public String remove(int i) { return list.remove(i); }
            public void add(int i, String s) { list.add(i,s); }
        }
        class Sequential extends AbstractSequentialList<String> implements ListString {
            public int size() { return list.size(); }
            public ListIterator<String> listIterator(int index) {
                final ListIterator<String> it = list.listIterator(index);
                return new ListIterator<String>() {
                    public void add(String s) { it.add(s); }
                    public boolean hasNext() { return it.hasNext(); }
                    public boolean hasPrevious() { return it.hasPrevious(); }
                    public String next() { return it.next(); }
                    public int nextIndex() { return it.nextIndex(); }
                    public String previous() { return it.previous(); }
                    public int previousIndex() { return it.previousIndex(); }
                    public void remove() { it.remove(); }
                    public void set(String s) { it.set(s); }
                };
            }
        }
    return list instanceof RandomAccess ? new Random() : new Sequential();
    }  
}
class ArrayListString extends ArrayList<String> implements ListString {
    public ArrayListString() { super(); }
    public ArrayListString(Collection<? extends String> c) { super(c); }
    public ArrayListString(int capacity) { super(capacity); }
}
```
这里声明了 ListString (可具体化的类型) 为 List<String>(不可具体化的类型) 的一个子类型。因此，第一个类型的每个值都属于第二个类型，但这不是可逆的。ListString 接口没有声明任何新的方法。它只是将现有方法专门化为类型参数 String。
为了基于委托实现专门化，我们定义了一个静态方法 wrap， 它接受一个List<String>的入参，返回 ListString 的类型。Java 库将作用于接口 Collection 的方法 放在一个名为 Collections 的类中，同样地，我们将 wrap 放在 一个名为 ListStrings 的类中。
这里有一个关于它如何使用的例子:
```java
List<? extends List<?>> lists = Arrays.asList(
    ListStrings.wrap(Arrays.asList("one","two")),
    Arrays.asList(3,4),
    Arrays.asList("five","six"),
    ListStrings.wrap(Arrays.asList("seven","eight"))
);
ListString[] array = new ListString[2];
int i = 0;
for (List<?> list : lists)
    if (list instanceof ListString)
        array[i++] = (ListString)list;
assert Arrays.toString(array).equals("[[one, two], [seven, eight]]");
```
它创建一个包含列表的lists，然后扫描找出实现了`ListString`的列表，并把他们放到一个数组里。数组创建，实例测试和类型转换现在都没问题，因为他们操作的是可具体化类型`ListString`，而不是不可具体化类型`List<String>`。请注意，未被包装过的`List<String>`不会被认为是`ListString`的一个实例。这也是为什么lists里的第三个list没有被复制到数组中。

`ListStrings`类的实现简单明了，尽管需要小心一些以保持良好的性能。Java集合框架指定，当一个列表支持快速随机访问时，它应该实现标记接口`RandomAccess`，以使得泛型算法在应用于随机或者顺序访问列表时表现良好。它也提供两个抽象类，`AbstractList`和
`AbstractSequentialList`，适用于定义随机或者顺序的访问列表。比如，`ArrayList`实现了` RandomAccess`，也继承了`AbstractList`，而`LinkedList`继承了`AbstractSequentialList`。`AbstractList`类根据提供随机访问的五种抽象方法来定义`List`接口的方法(size, get, set, add, remove)，这些方法必须在子类中定义。同样的，`AbstractSequentialList`类根据提供顺序访问的两种抽象方法来定义(size, listIterator)`List`接口的所有方法，并且必须在子类中定义。

`wrap`方法检查给定的列表是否实现了`RandomAccess`接口。如果实现了，它返回一个继承了`AbstractList`类且实现了`RandomAccess`接口的`Random`类的实例，否则它返回一个继承了`AbstractSequentialList`类的`Sequential`类的实例。`Random`类实现了必须由`AbstractList`的一个子类提供的五个方法。类似地，`Sequential`类实现了必须由`AbstractSequentialList`的一个子类提供的两个方法，其中第二个方法返回一个类，它实现了`ListIterator`接口的9个方法。如下所述，通过委托来实现列表迭代器，而不是简单地返回原始的列表迭代器，这样可以改进包装器的安全属性。所有这些方法都可以通过委托直接实现。

`wrap`方法返回一个底层列表的视图，如果尝试插入不是String类型的元素，将引起类型转换异常。这些检查类似于`checkedList`包装类提供的检查。然而，对于wrap方法，编译器会插入的相关转换(通过委托实现`listIterator`接口的这九个方法的一个原因是为了确保插入这些转换)，而对于`checked list`，这些转换是通过反射执行的。泛型通常会使得这些检查多余，但是当存在遗留代码或者unchecked warnings时，或者当处理如8.2章节提到的安全问题时，他们会很有用。



## 8.4 保持二进制兼容

正如我们所强调的，为了简化升级，泛型是通过擦除来实现的。当要升级遗留代码为泛型时，我们想要保证新的泛型代码能够和任何现有代码一起工作，包括那些我们没有源代码的类文件。在这种情况下，我们才说遗留代码和泛型代码是二进制兼容的( *binary compatible*)。

当泛型代码被擦除之后的代码签名和遗留代码的签名一样，并且两个版本的代码都被编译成相同的字节码的时候，才能保证二进制兼容性。通常，这是泛型的自然结果，但在这一节，让我们看看一些不常见的可能导致问题的情况。这一节的某些例子取自Mark  Reinhold 写在Sun内部的笔记。

**调整擦除** 一个不常见的情况与Collections类中的`max`方法的泛型有关。我们在小节3.2和小节3.6讨论过它，但它值得快速一看。

这里是遗留代码的方法签名：

```java
// legacy version
public static Object max(Collection coll)
```

这里是自然的泛型签名，使用通配符来获得最大的灵活性：

```java
// generic version -- breaks binary compatibility
public static <T extends Comparable<? super T>> 
T max(Collection<? extends T> coll)
```

但这个方法擦除之后是错的——它的返回类型是`Comparable`而不是`Object`。俄日了得到正确的签名，我们需要用多个界限来调整类型参数的界限。这是改正后的版本：

```java
// generic version -- maintains binary compatibility
public static <T extends Object & Comparable<? super T>>
 T max(Collection<? extends T> coll)
```

当有多个边界的时候，类型擦除会取最左边界。所以现在 `T` 擦除之后的类型是 `Object`，给出了我们需要的结果类型。

泛化过程中出现了一些问题也可能因为原始的遗留代码包含的类型比它可能有的要小。比如，遗留代码版的 `max` 可能返回类型`Comparable`，这比`Object`更具体，这样的话，就不需要使用多个边界来调整类型了。

**桥接** 另一个很重要的不常见的情况跟桥接有关。`Comparable`又一次提供了很好的例子。