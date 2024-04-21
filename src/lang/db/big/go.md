

# GO  small talk

## 组合优于继承 ?

> 由于组合不像继承那样结构化，因此它具有更多种方法。然而，这可能是最简单的组合解决方案。不同的组合解决方案会更复杂一些，但会消除更多的代码重复。它将利用函数组合：

```go
//组合
interface Animal {
    doStuff()
}

class Dog implements Animal {
    doStuff() {
        genericDoStuff(function() {
            println("woof")
        })
    }
}

class Cat implements Animal {
    doStuff() {
        genericDoStuff(function() {
            println("meow")
        })
    }
}

class Human implements Animal {
    doStuff() {
        genericDoStuff(function() {
            println("hello")
        })
    }
}

function genericDoStuff(speak) {
    eat()
    poop()
    speak()
    sit()
    sleep()
}
```



```java
//继承
interface Animal {
    doStuff()
}

abstract class AbstractAnimal implements Animal {
    abstract speak()

    doStuff() {
        eat()
        poop()
        this.speak()
        sit()
        sleep()
    }
}

class Dog extends AbstractAnimal {
    speak() {
        println("woof")
    }
}

class Cat extends AbstractAnimal {
    speak() {
        println("meow")
    }
}

class Human extends AbstractAnimal {
    speak() {
        println("hello")
    }
}
```

## go  分析工具

pprof

`pprof` 是 Go 语言标准库中的性能分析工具，用于收集和分析程序的性能数据。它提供了多种分析报告，帮助开发人员识别和优化应用程序中的性能瓶颈。

`pprof` 的主要特点包括：

1. **低侵入性：** 使用 `pprof` 进行性能分析不需要修改源代码，只需在代码中导入 `net/http/pprof` 包，然后通过 HTTP 端点进行数据采集和查看。

2. **多种分析类型：** `pprof` 支持多种性能分析类型，包括 CPU 、内存、阻塞等。开发人员可以根据需求选择不同的分析类型来找出不同类型的性能问题。

3. **交互式命令行工具：** `pprof` 提供了一个交互式命令行工具，可以在终端中使用。通过 `go tool pprof` 命令，开发人员可以加载分析数据，查看报告，分析函数调用图等。

4. **图形界面支持：** 除了命令行工具，`pprtof` 还支持可视化的图形界面。开发人员可以使用 `go tool pprof -http` 命令启动一个本地 Web 服务器，通过浏览器查看分析报告。

5. **内置 HTTP 服务器：** `net/http/pprof` 包还提供了一个内置的 HTTP 服务器，可以方便地将性能分析数据暴露为 HTTP 端点。这使得远程性能监测和调试变得更加便捷。

6. **多种输出格式：** `pprof` 支持多种输出格式，包括文本、PDF、SVG、Graphviz 等，帮助开发人员以不同的方式查看分析报告。

`pprof` 可以帮助开发人员定位程序中的性能问题，比如 CPU 使用过高、内存泄漏、阻塞等。通过分析分析报告，开发人员可以找到性能瓶颈，并进行相应的优化。它是 Go 开发人员在性能调优方面的有力工具。
