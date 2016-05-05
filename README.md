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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/map.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/flatmap.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/scan.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/filter.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/distinct.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/take.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/startwith.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/combinelatest.html )

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

### `zip`

使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果(从序列中依次取数据)

![](https://raw.githubusercontent.com/kzaher/rxswiftcontent/master/MarbleDiagrams/png/zip.png)

[More info in reactive.io website](http://reactivex.io/documentation/operators/zip.html)

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/merge.html )

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

[More info in reactive.io website]( http://reactivex.io/documentation/operators/switch.html )


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



