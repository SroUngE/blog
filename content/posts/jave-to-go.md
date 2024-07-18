---
title: "Jave to Go"
date: 2024-06-11T22:54:40+08:00
draft: true
---

### 函数（Function）和方法（Method）

在Java里，函数（Function）和方法（Method）并未过多区分，Java文件要求至少要有一个类，并且方法/函数 必须定义在类里。

在Go里，函数是过程式的，可以直接定义在go文件中；方法是面向对象式的（将调用一个方法称为“向一个对象发送消息”），叫做方法的接收器（receiver）。在Go语言中，我们并不会像其它语言那样用this或者self作为接收器；我们可以任意的选择接收器的名字。

由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。

```go
package geometry

import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

这种 `p.Distance` 的表达式叫做选择器，因为他会选择合适的对应p这个对象的Distance方法来执行。选择器也会被用来选择一个struct类型的字段，比如 `p.X`。

```go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
```

Path是一个命名的slice类型，而不是Point那样的struct类型，然而我们依然可以为它定义方法。在能够给任意类型定义方法这一点上，Go和很多其它的面向对象的语言不太一样。

```go
// A Path is a journey connecting the points with straight lines.
type Path []Point
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
            sum += path[i-1].Distance(path[i])
        }
    }
    return sum
}
```

在Go语言中，指定接收器出了用 `receiver.Method()` 的方式，还可以用 `Method(..., reveiver)`，即接收器作为函数的参数传入即可：

```go
p := Point{1, 2}
q := Point{4, 6}

distanceFromP := p.Distance        // method value
fmt.Println(distanceFromP(q))      // "5"

distance := Point.Distance   // method expression
fmt.Println(distance(p, q))  // "5"
```

https://gopl-zh.github.io/ch6/ch6-01.html

