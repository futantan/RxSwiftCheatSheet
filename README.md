> 本文档内容来自于 [RxSwift](https://github.com/ReactiveX/RxSwift) 的 Playground。记录大多数 ReactiveX 的概念和操作符。

(部分翻译和注解来自 [ReactiveX文档中文翻译](https://mcxiaoke.gitbooks.io/rxdocs/content/Subject.html))

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


# Subjects
Subject 可以看成是一个桥梁或者代理，在某些ReactiveX实现中，它同时充当了 Observer 和 Observable 的角色。因为它是一个Observer，它可以订阅一个或多个 Observable；又因为它是一个 Observable，它可以转发它收到(Observe)的数据，也可以发射新的数据。

辅助函数：

```swift
func writeSequenceToConsole<O: ObservableType>(name: String, sequence: O) -> Disposable {
    return sequence
        .subscribe { e in
            print("Subscription: \(name), event: \(e)")
        }
}
```

## PublishSubject

`PublishSubject` 只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/publishsubject.png)

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/publishsubject_error.png)

```swift
example("PublishSubject") {
    let disposeBag = DisposeBag()

    let subject = PublishSubject<String>()
    writeSequenceToConsole("1", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("a"))
    subject.on(.Next("b"))
    writeSequenceToConsole("2", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("c"))
    subject.on(.Next("d"))
}
```

运行结果：

```
--- PublishSubject example ---
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Next(d)
Subscription: 2, event: Next(d)
```

## ReplaySubject

`ReplaySubject` 会发射所有来自原始Observable的数据给观察者，无论它们是何时订阅的。当一个新的 observer 订阅了一个 `ReplaySubject` 之后，他将会收到当前缓存在 buffer 中的数据和这之后产生的新数据。在下面的例子中，缓存大小为 `1` 所以 observer 将最多能够收到订阅时间点之前的一个数据。例如，`Subscription: 2` 能够收到消息 `"b"`，而这个消息是在他订阅之前发送的，但是没有办法收到消息 `"a"` 因为缓存的容量小于 `2`。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/replaysubject.png)

```swift
example("ReplaySubject") {
    let disposeBag = DisposeBag()
    let subject = ReplaySubject<String>.create(bufferSize: 1)

    writeSequenceToConsole("1", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("a"))
    subject.on(.Next("b"))
    writeSequenceToConsole("2", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("c"))
    subject.on(.Next("d"))
}
```

运行结果：

```
--- ReplaySubject example ---  
Subscription: 1, event: Next(a)  
Subscription: 1, event: Next(b)  
Subscription: 2, event: Next(b)  
Subscription: 1, event: Next(c)  
Subscription: 2, event: Next(c)  
Subscription: 1, event: Next(d)  
Subscription: 2, event: Next(d)  
```

## BehaviorSubject

当观察者订阅 `BehaviorSubject` 时，它开始发射原始 Observable 最近发射的数据（如果此时还没有收到任何数据，它会发射一个默认值），然后继续发射其它任何来自原始Observable的数据。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/behaviorsubject.png)

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/behaviorsubject_error.png)

```swift
example("BehaviorSubject") {
    let disposeBag = DisposeBag()

    let subject = BehaviorSubject(value: "z")
    writeSequenceToConsole("1", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("a"))
    subject.on(.Next("b"))
    writeSequenceToConsole("2", sequence: subject).addDisposableTo(disposeBag)
    subject.on(.Next("c"))
    subject.on(.Next("d"))
    subject.on(.Completed)
}
```

运行结果：

```
--- BehaviorSubject example ---
Subscription: 1, event: Next(z)
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 2, event: Next(b)
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Next(d)
Subscription: 2, event: Next(d)
Subscription: 1, event: Completed
Subscription: 2, event: Completed
```

## Variable

`Variable` 封装了 `BehaviorSubject`。使用 variable 的好处是 variable 将不会显式的发送 `Error` 或者 `Completed`。在 deallocated 的时候，`Variable` 会自动的发送 complete 事件。

```swift
example("Variable") {
    let disposeBag = DisposeBag()
    let variable = Variable("z")
    writeSequenceToConsole("1", sequence: variable.asObservable()).addDisposableTo(disposeBag)
    variable.value = "a"
    variable.value = "b"
    writeSequenceToConsole("2", sequence: variable.asObservable()).addDisposableTo(disposeBag)
    variable.value = "c"
    variable.value = "d"
}
```

运行结果：

```
--- Variable example ---
Subscription: 1, event: Next(z)
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 2, event: Next(b)
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Next(d)
Subscription: 2, event: Next(d)
Subscription: 1, event: Completed
Subscription: 2, event: Completed
```


## 变换操作

下面列出了可用于对 Observable 发射的数据执行变换操作的各种操作符。

### `map` / `select`

对序列的每一项都应用一个函数来变换 Observable 发射的数据序列

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/map.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/map.html )

```swift
example("map") {
    let originalSequence = Observable.of(1, 2, 3)

    _ = originalSequence
        .map { number in
            number * 2
        }
        .subscribe { print($0) }
}
```

运行结果：

```
--- map example ---
Next(2)
Next(4)
Next(6)
Completed
```



### `flatMap`

将每个 Obserable 发射的数据变换为 Observable 的集合，然后将其 "拍扁"（降维 flatten）成一个 Observable。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/flatmap.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/flatmap.html )

```swift
example("flatMap") {
    let sequenceInt = Observable.of(1, 2, 3)

    let sequenceString = Observable.of("A", "B", "C", "D", "E", "F", "--")

    _ = sequenceInt
        .flatMap { (x:Int) -> Observable<String> in
            print("from sequenceInt \(x)")
            return sequenceString
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- flatMap example ---
from sequenceInt 1
Next(A)
Next(B)
Next(C)
Next(D)
Next(E)
Next(F)
Next(--)
from sequenceInt 2
Next(A)
Next(B)
Next(C)
Next(D)
Next(E)
Next(F)
Next(--)
from sequenceInt 3
Next(A)
Next(B)
Next(C)
Next(D)
Next(E)
Next(F)
Next(--)
Completed
```

### `scan`

对 Observable 发射的每一项数据应用一个函数，然后按顺序依次发射每一个值

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/scan.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/scan.html )

```swift
example("scan") {
    let sequenceToSum = Observable.of(0, 1, 2, 3, 4, 5)

    _ = sequenceToSum
        .scan(0) { acum, elem in
            acum + elem
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- scan example ---
Next(0)
Next(1)
Next(3)
Next(6)
Next(10)
Next(15)
Completed
```

## 过滤操作

从源 Observable 中选择特定的数据发送

### `filter`

只发送 Observable 中通过特定测试的数据

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/filter.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/filter.html )

```swift
example("filter") {
    let subscription = Observable.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter {
            $0 % 2 == 0
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- filter example ---
Next(0)
Next(2)
Next(4)
Next(6)
Next(8)
Completed
```


### `distinctUntilChanged`

过滤掉连续重复的数据

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/distinct.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/distinct.html )

```swift
example("distinctUntilChanged") {
    let subscription = Observable.of(1, 2, 3, 1, 1, 4)
        .distinctUntilChanged()
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- distinctUntilChanged example ---
Next(1)
Next(2)
Next(3)
Next(1)
Next(4)
Completed
```


### `take`

仅发送 Observable 的前 n 个数据项

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/take.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/take.html )

```swfit
example("take") {
    let subscription = Observable.of(1, 2, 3, 4, 5, 6)
        .take(3)
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- take example ---
Next(1)
Next(2)
Next(3)
Completed
```

## 结合操作(Combination operators)

将多个 Observable 结合成一个 Observable


### `startWith`

在数据序列的开头增加一些数据

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/startwith.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/startwith.html )

```swift
example("startWith") {

    let subscription = Observable.of(4, 5, 6, 7, 8, 9)
        .startWith(3)
        .startWith(2)
        .startWith(1)
        .startWith(0)
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- startWith example ---
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


### `combineLatest`

当两个 Observables 中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/combinelatest.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/combinelatest.html )

```swift
example("combineLatest 1") {
    let intOb1 = PublishSubject<String>()
    let intOb2 = PublishSubject<Int>()

    _ = Observable.combineLatest(intOb1, intOb2) {
        "\($0) \($1)"
        }
        .subscribe {
            print($0)
        }

    intOb1.on(.Next("A"))

    intOb2.on(.Next(1))

    intOb1.on(.Next("B"))

    intOb2.on(.Next(2))
}
```

运行结果：

```
--- combineLatest 1 example ---
Next(A 1)
Next(B 1)
Next(B 2)
```

为了能够产生结果，两个序列中都必须保证至少有一个元素

```swift
example("combineLatest 2") {
    let intOb1 = Observable.just(2)
    let intOb2 = Observable.of(0, 1, 2, 3, 4)

    _ = Observable.combineLatest(intOb1, intOb2) {
            $0 * $1
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- combineLatest 2 example ---
Next(0)
Next(2)
Next(4)
Next(6)
Next(8)
Completed
```

Combine latest 有超过 2 个参数的版本

```swift
example("combineLatest 3") {
    let intOb1 = Observable.just(2)
    let intOb2 = Observable.of(0, 1, 2, 3)
    let intOb3 = Observable.of(0, 1, 2, 3, 4)

    _ = Observable.combineLatest(intOb1, intOb2, intOb3) {
            ($0 + $1) * $2
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- combineLatest 3 example ---
Next(0)
Next(5)
Next(10)
Next(15)
Next(20)
Completed
```

Combinelatest 可以作用于不同数据类型的序列

```swift
example("combineLatest 4") {
    let intOb = Observable.just(2)
    let stringOb = Observable.just("a")

    _ = Observable.combineLatest(intOb, stringOb) {
            "\($0) " + $1
        }
        .subscribe {
            print($0)
    }
}
```

运行结果：

```
--- combineLatest 4 example ---
Next(2 a)
Completed
```

`combineLatest` 方法可以在 Array 上使用，数组元素类型必须遵循 `ObservableType` 协议  
数组中的元素类型必须为 `Observables`

```swift
example("combineLatest 5") {
    let intOb1 = Observable.just(2)
    let intOb2 = Observable.of(0, 1, 2, 3)
    let intOb3 = Observable.of(0, 1, 2, 3, 4)

    _ = [intOb1, intOb2, intOb3].combineLatest { intArray -> Int in
            Int((intArray[0] + intArray[1]) * intArray[2])
        }
        .subscribe { (event: Event<Int>) -> Void in
            print(event)
        }
}
```

### `withLatestFrom`

Merges two observable sequences into one observable sequence by using latest element from the second sequence every time when self emitts an element.

将两个 Observable 序列合并为一个。每当 self 队列发射一个元素时，从第二个序列中取出最新的一个值。

```swift
example("withLatestFrom") {
    let subjectA = PublishSubject<String>()
    let subjectB = PublishSubject<String>()

    subjectA
        .withLatestFrom(subjectB)
        .subscribeNext({ (data) in
            print(data)
        })

    subjectA.onNext("a1")
    subjectB.onNext("b1")
    subjectA.onNext("a2")
    subjectA.onNext("a3")
    subjectB.onNext("b2")
    subjectA.onNext("a4")
}
```

运行结果：

```
--- withLatestFrom example ---
b1
b1
b2
```

### `zip`

使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果(从序列中依次取数据)

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/zip.png)

[更多相关内容请查看 reactive.io](http://reactivex.io/documentation/operators/zip.html)

```swift
example("zip 1") {
    let intOb1 = PublishSubject<String>()
    let intOb2 = PublishSubject<Int>()

    _ = Observable.zip(intOb1, intOb2) {
        "\($0) \($1)"
        }
        .subscribe {
            print($0)
        }

    intOb1.on(.Next("A"))

    intOb2.on(.Next(1))

    intOb1.on(.Next("B"))

    intOb1.on(.Next("C"))

    intOb2.on(.Next(2))
}
```

运行结果：

```
--- zip 1 example ---
Next(A 1)
Next(B 2)
```

```swift
example("zip 2") {
    let intOb1 = Observable.just(2)

    let intOb2 = Observable.of(0, 1, 2, 3, 4)

    _ = Observable.zip(intOb1, intOb2) {
            $0 * $1
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- zip 2 example ---
Next(0)
Completed
```


```swift
example("zip 3") {
    let intOb1 = Observable.of(0, 1)
    let intOb2 = Observable.of(0, 1, 2, 3)
    let intOb3 = Observable.of(0, 1, 2, 3, 4)

    _ = Observable.zip(intOb1, intOb2, intOb3) {
            ($0 + $1) * $2
        }
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- zip 3 example ---
Next(0)
Next(2)
Completed
```


### `merge`

合并多个 Observables 的组合成一个

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/merge.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/merge.html )

```swift
example("merge 1") {
    let subject1 = PublishSubject<Int>()
    let subject2 = PublishSubject<Int>()

    _ = Observable.of(subject1, subject2)
        .merge()
        .subscribeNext { int in
            print(int)
        }

    subject1.on(.Next(20))
    subject1.on(.Next(40))
    subject1.on(.Next(60))
    subject2.on(.Next(1))
    subject1.on(.Next(80))
    subject1.on(.Next(100))
    subject2.on(.Next(1))
}
```

运行结果：

```
--- merge 1 example ---
20
40
60
1
80
100
1
```

```swift
example("merge 2") {
    let subject1 = PublishSubject<Int>()
    let subject2 = PublishSubject<Int>()

    _ = Observable.of(subject1, subject2)
        .merge(maxConcurrent: 2)
        .subscribe {
            print($0)
        }

    subject1.on(.Next(20))
    subject1.on(.Next(40))
    subject1.on(.Next(60))
    subject2.on(.Next(1))
    subject1.on(.Next(80))
    subject1.on(.Next(100))
    subject2.on(.Next(1))
}
```

运行结果：

```
--- merge 2 example ---
Next(20)
Next(40)
Next(60)
Next(1)
Next(80)
Next(100)
Next(1)
```


### `switchLatest`

将一个发射多个 Observables 的 Observable 转换成另一个单独的 Observable，后者发射那些 Observables 最近发射的数据项

Switch 订阅一个发射多个 Observables 的 Observable。它每次观察那些 Observables 中的一个，Switch 返回的这个Observable取消订阅前一个发射数据的 Observable，开始发射最近的Observable 发射的数据。注意：当原始 Observable 发射了一个新的 Observable 时（不是这个新的 Observable 发射了一条数据时），它将取消订阅之前的那个 Observable。这意味着，在后来那个 Observable 产生之后到它开始发射数据之前的这段时间里，前一个 Observable 发射的数据将被丢弃

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/switch.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/switch.html )


```swift
example("switchLatest") {
    let var1 = Variable(0)

    let var2 = Variable(200)

    // var3 is like an Observable<Observable<Int>>
    let var3 = Variable(var1.asObservable())

    let d = var3
        .asObservable()
        .switchLatest()
        .subscribe {
            print($0)
        }

    var1.value = 1
    var1.value = 2
    var1.value = 3
    var1.value = 4

    var3.value = var2.asObservable()

    var2.value = 201

    var1.value = 5
    var1.value = 6
    var1.value = 7
}
```

运行结果：

```
--- switchLatest example ---
Next(0)
Next(1)
Next(2)
Next(3)
Next(4)
Next(200)
Next(201)
Completed
```


## Error Handling Operators

下面的操作符帮助我们从 Observable 发射的 error 通知做出响应或者从错误中恢复。

### `catchError`

收到 `Error` 通知之后，转而发送一个没有错误的序列。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/catch.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/catch.html )


```swift
example("catchError 1") {
    let sequenceThatFails = PublishSubject<Int>()
    let recoverySequence = Observable.of(100, 200, 300, 400)

    _ = sequenceThatFails
        .catchError { error in
            return recoverySequence
        }
        .subscribe {
            print($0)
        }

    sequenceThatFails.on(.Next(1))
    sequenceThatFails.on(.Next(2))
    sequenceThatFails.on(.Next(3))
    sequenceThatFails.on(.Next(4))
    sequenceThatFails.on(.Error(NSError(domain: "Test", code: 0, userInfo: nil)))
}
```

运行结果：

```
--- catchError 1 example ---
Next(1)
Next(2)
Next(3)
Next(4)
Next(100)
Next(200)
Next(300)
Next(400)
Completed
```

```swift
example("catchError 2") {
    let sequenceThatFails = PublishSubject<Int>()

    _ = sequenceThatFails
        .catchErrorJustReturn(100)
        .subscribe {
            print($0)
        }

    sequenceThatFails.on(.Next(1))
    sequenceThatFails.on(.Next(2))
    sequenceThatFails.on(.Next(3))
    sequenceThatFails.on(.Next(4))
    sequenceThatFails.on(.Error(NSError(domain: "Test", code: 0, userInfo: nil)))
}
```

运行结果：

```
--- catchError 2 example ---
Next(1)
Next(2)
Next(3)
Next(4)
Next(100)
Completed
```

### `retry`

如果原始 Observable 遇到错误，重新订阅，心里默念，不会出错不会出错...

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/retry.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/retry.html )


```swift
example("retry") {
    var count = 1 // bad practice, only for example purposes
    let funnyLookingSequence = Observable<Int>.create { observer in
        let error = NSError(domain: "Test", code: 0, userInfo: nil)
        observer.on(.Next(0))
        observer.on(.Next(1))
        observer.on(.Next(2))
        if count < 2 {
            observer.on(.Error(error))
            count += 1
        }
        observer.on(.Next(3))
        observer.on(.Next(4))
        observer.on(.Next(5))
        observer.on(.Completed)

        return NopDisposable.instance
    }

    _ = funnyLookingSequence
        .retry()
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- retry example ---
Next(0)
Next(1)
Next(2)
Next(0)
Next(1)
Next(2)
Next(3)
Next(4)
Next(5)
Completed
```

## Observable Utility Operators

下面的操作符可以当做一个工具集，方便操作 Observable

### `subscribe`

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/subscribe.html )

```swift
example("subscribe") {
    let sequenceOfInts = PublishSubject<Int>()

    _ = sequenceOfInts
        .subscribe {
            print($0)
        }

    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}
```

运行结果：

```
--- subscribe example ---
Next(1)
Completed
```

下面是几个 `subscribe` 操作符的变体


### `subscribeNext`

```swift
example("subscribeNext") {
    let sequenceOfInts = PublishSubject<Int>()

    _ = sequenceOfInts
        .subscribeNext {
            print($0)
        }

    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}
```

运行结果：

```
--- subscribeNext example ---
1
```


### `subscribeCompleted`

```swift
example("subscribeCompleted") {
    let sequenceOfInts = PublishSubject<Int>()

    _ = sequenceOfInts
        .subscribeCompleted {
            print("It's completed")
        }

    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}
```

运行结果：

```
--- subscribeCompleted example ---
It's completed
```


### `subscribeError`

```swift
example("subscribeError") {
    let sequenceOfInts = PublishSubject<Int>()

    _ = sequenceOfInts
        .subscribeError { error in
            print(error)
        }

    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Error(NSError(domain: "Examples", code: -1, userInfo: nil)))
}
```

运行结果：

```
--- subscribeError example ---
Error Domain=Examples Code=-1 "(null)"
```

### `doOn`

注册一个操作来监听事件的生命周期
（register an action to take upon a variety of Observable lifecycle events）

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/do.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/do.html )


```swift
example("doOn") {
    let sequenceOfInts = PublishSubject<Int>()

    _ = sequenceOfInts
        .doOn {
            print("Intercepted event \($0)")
        }
        .subscribe {
            print($0)
        }

    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}
```

运行结果：

```
--- doOn example ---
Intercepted event Next(1)
Next(1)
Intercepted event Completed
Completed
```


## 条件和布尔操作（Conditional and Boolean Operators）

下面的操作符可用于根据条件发射或变换 Observables，或者对它们做布尔运算：


### `takeUntil`

当第二个 Observable 发送数据之后，丢弃第一个 Observable 在这之后的所有消息。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/takeuntil.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/takeuntil.html )

```swift
example("takeUntil") {
    let originalSequence = PublishSubject<Int>()
    let whenThisSendsNextWorldStops = PublishSubject<Int>()

    _ = originalSequence
        .takeUntil(whenThisSendsNextWorldStops)
        .subscribe {
            print($0)
        }

    originalSequence.on(.Next(1))
    originalSequence.on(.Next(2))
    originalSequence.on(.Next(3))
    originalSequence.on(.Next(4))

    whenThisSendsNextWorldStops.on(.Next(1))

    originalSequence.on(.Next(5))
}
```

运行结果：

```
--- takeUntil example ---
Next(1)
Next(2)
Next(3)
Next(4)
Completed
```


### `takeWhile`

发送原始 Observable 的数据，直到一个特定的条件为 false

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/takewhile.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/takewhile.html )

```swift
example("takeWhile") {

    let sequence = PublishSubject<Int>()

    _ = sequence
        .takeWhile { int in
            int < 4
        }
        .subscribe {
            print($0)
        }

    sequence.on(.Next(1))
    sequence.on(.Next(2))
    sequence.on(.Next(3))
    sequence.on(.Next(4))
    sequence.on(.Next(5))
}
```

运行结果：

```
--- takeWhile example ---
Next(1)
Next(2)
Next(3)
Completed
```


## 算数和聚合(Mathematical and Aggregate Operators)

### `concat`

合并两个或者以上的 Observable 的消息，并且这些消息的发送时间不会交叉。（队列先后顺序不会交叉）

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/concat.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/concat.html )

```swift
example("concat") {
    let var1 = BehaviorSubject(value: 0)
    let var2 = BehaviorSubject(value: 200)

    // var3 is like an Observable<Observable<Int>>
    let var3 = BehaviorSubject(value: var1)

    let d = var3
        .concat()
        .subscribe {
            print($0)
        }

    var1.on(.Next(1))
    var1.on(.Next(2))
    var1.on(.Next(3))
    var1.on(.Next(4))

    var3.on(.Next(var2))

    var2.on(.Next(201))

    var1.on(.Next(5))
    var1.on(.Next(6))
    var1.on(.Next(7))
    var1.on(.Completed)

    var2.on(.Next(202))
    var2.on(.Next(203))
    var2.on(.Next(204))
}
```

运行结果：

```
--- concat example ---
Next(0)
Next(1)
Next(2)
Next(3)
Next(4)
Next(5)
Next(6)
Next(7)
Next(201)
Next(202)
Next(203)
Next(204)
```

### `reduce`

按顺序对Observable发射的每项数据应用一个函数并发射最终的值。  
`Reduce` 操作符对原始 Observable 发射数据的第一项应用一个函数，然后再将这个函数的返回值与第二项数据一起传递给函数，以此类推，持续这个过程知道原始Observable发射它的最后一项数据并终止，此时 Reduce 返回的 Observable 发射这个函数返回的最终值。与数组序列的 `reduce` 操作类似。

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/reduce.png)

[更多相关内容请查看 reactive.io]( http://reactivex.io/documentation/operators/reduce.html )

```swift
example("reduce") {
    _ = Observable.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
        .reduce(0, accumulator: +)
        .subscribe {
            print($0)
        }
}
```

运行结果：

```
--- reduce example ---
Next(45)
Completed
```
