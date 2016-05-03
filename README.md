- [Introduction](Introduction)
- [Subjects](Subjects)
- [Transforming Observables](Transforming_Observables)
- [Filtering Observables](Filtering_Observables)
- [Combining Observables](Combining_Observables)
- [Error Handling Operators](Error_Handling_Operators)
- [Observable Utility Operators](Observable_Utility_Operators)
- [Conditional and Boolean Operators](Conditional_and_Boolean_Operators)
- [Mathematical and Aggregate Operators](Mathematical_and_Aggregate_Operators)
- [Connectable Observable Operators](Connectable_Observable_Operators)

# Introduction

## 为什么使用 RxSwift?

我们写的很多代码实际上是为了解决和响应外部事件。当用户操作一个控件的时候，我们需要使用 @IBAction 来响应事件。我们需要观察通知来检测键盘改变位置。当 URL Sessions 带着响应的数据返回时，我们需要提供闭包来执行我们的操作。我们还需要使用 KVO 来检测变量的值改变。这些大量的编写机制使得我们的代码结构变的更加复杂。如果有一种统一的编写机制来完成所有的这些调用/响应代码是不是更棒呢？Rx 就是为解决这些问题而生的。

## Observable
理解 RxSwift 的关键是理解 Observable 的概念。要理解它的创建，操作以及为了对变化做出响应操作而进行的订阅（subscribe）。

## 创建和订阅 Observable
要理解本框架，第一步需要理解如何创建 Observable。有很多函数可以创建 Observable。

创建 Observable 之后，如果没有订阅者订阅该 observable，那么什么事情也不会发生，所以我们将同时解释创建和订阅。

### empty  

`empty` 创建一个空的序列。它仅发送 `.Completed` 消息。

```swift
example("empty") {
    let emptySequence = Observable<Int>.empty()

    let subscription = emptySequence
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- empty example ---
Completed
```

### never
`never` 创建一个序列，该序列永远不会发送消息，`.Completed` 消息也不会发送。

```swift
example("never") {
    let neverSequence = Observable<Int>.never()

    let subscription = neverSequence
        .subscribe { _ in
            print("This block is never called.")
        }
}
```

运行结果：
```
--- never example ---
```


### just
`just` 代表只包含一个元素的序列。它将向订阅者发送两个消息，第一个消息是其中元素的值，另一个是 `.Completed`。

```swift
example("just") {
    let singleElementSequence = Observable.just(32)

    let subscription = singleElementSequence
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- just example ---
Next(32)
Completed
```

### sequenceOf
`sequenceOf` 通过固定数目的元素创建一个序列

```swift
example("sequenceOf") {
    let sequenceOfElements/* : Observable<Int> */ = Observable.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

    let subscription = sequenceOfElements
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- sequenceOf example ---
Next(0)
Next(1)
Next(2)
Next(3)
Next(4)
Next(5)
Next(6)
Next(7)
Next(8)
Next(9)
Completed
```

### toObservable
`toObservable` 在一个数组的基础上创建一个序列

```swift
example("toObservable") {
    let sequenceFromArray = [1, 2, 3, 4, 5].toObservable()

    let subscription = sequenceFromArray
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- toObservable example ---
Next(1)
Next(2)
Next(3)
Next(4)
Next(5)
Completed
```


### create
`create` 使用 Swift 闭包来创建一个序列。该例子中，创建了 `just` 操作符的自定义版本。

```swift
example("create") {
    let myJust = { (singleElement: Int) -> Observable<Int> in
        return Observable.create { observer in
            observer.on(.Next(singleElement))
            observer.on(.Completed)

            return NopDisposable.instance
        }
    }

    let subscription = myJust(5)
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- create example ---
Next(5)
Completed
```

### generate
`generate` 创建的序列可以自己生成它的值，并且在之前值的基础上来判断什么时候结束。

```swift
example("generate") {
    let generated = Observable.generate(
        initialState: 0,
        condition: { $0 < 3 },
        iterate: { $0 + 1 }
    )

    let subscription = generated
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- generate example ---
Next(0)
Next(1)
Next(2)
Completed
```


### error
创建一个不发送任何 item 的 Observable，以 error 中指

```swift
example("error") {
    let error = NSError(domain: "Test", code: -1, userInfo: nil)

    let erroredSequence = Observable<Int>.error(error)

    let subscription = erroredSequence
        .subscribe { event in
            print(event)
        }
}
```

运行结果：

```
--- error example ---
Error(Error Domain=Test Code=-1 "(null)")
```


### deferred

直到 observer 订阅之后才创建 Observable，并且为每一个 observer 创建一个全新的 Observable
do not create the Observable until the observer subscribes, and create a fresh Observable for each observer

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/defer.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/defer.html )

```swift
example("deferred") {
    let deferredSequence: Observable<Int> = Observable.deferred {
        print("creating")
        return Observable.create { observer in
            print("emmiting")
            observer.on(.Next(0))
            observer.on(.Next(1))
            observer.on(.Next(2))

            return NopDisposable.instance
        }
    }

    _ = deferredSequence
        .subscribe { event in
            print(event)
         }

    _ = deferredSequence
        .subscribe { event in
            print(event)
         }
}
```

运行结果：

```
--- deferred example ---
creating
emmiting
Next(0)
Next(1)
Next(2)
creating
emmiting
Next(0)
Next(1)
Next(2)
```

在 RxCocoa 库中还有很多其他非常有用的方法，例如：

* `rx_observe` 存在于所有 NSObject 子类中，封装了 KVO
* `rx_tap` 存在于 button 中，封装了 @IBActions
* `rx_notification` 封装了 NotificationCenter
* ...
