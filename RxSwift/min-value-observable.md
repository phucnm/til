# Custom `min` operator for observables based on `reduce` operator in RxSwift

RxSwift is a framework that utilizes FRP to make code styles elegant and side-effect free. There was a case I had to emit the minimum value of an observable: I had to ping several servers and then conclude which server has the lowest ping to work with. I decided to write different versions of custom operators `min` and `accumulativeMin` based on `reduce` (and `scan`).

> `min` must be only available on an observable contains comparable elements.
```
extension ObservableType where E: Comparable {
    func min() -> Observable<Self.E> {
        return reduce(Optional<E>.none, accumulator: { (min, e) -> Optional<E> in
            if let m = min, m < e {
                return m
            }
            return e
        }).unwrap()
    }   
}
```
`unwrap` is a cool wrapper to ignore `nil` values. [Refer here](https://github.com/RxSwiftCommunity/RxSwiftExt/blob/master/Source/RxSwift/unwrap.swift)

Let's try out what we've done.
```
let publisher = PublishSubject<Int>()
publisher.asObserver().map(Double.init).min().debug("min").subscribe()
publisher.onNext(9)
publisher.onNext(8)
publisher.onNext(7)
publisher.onNext(6)
publisher.onNext(5)
publisher.onCompleted()
```

Results:
```
min -> subscribed
min -> Event next(5.0)
min -> Event completed
min -> isDisposed
```

This operator will only emit the minimum value when the observable is completed. Now I would need my `accumulativeMin` to track the minimum value whenever an element is emitted. I may reuse the accumulator in `min` as well. The only different here is I used `scan` instead of `reduce` to get accumulative events.
```
extension ObservableType where E: Comparable {
    private var accumulator: ((Optional<E>, E) -> Optional<E>) {
        return { min, e -> Optional<E> in
            if let m = min, m < e  {
                return m
            }
            return e
        }
    }

    func accumulativeMin() -> Observable<Self.E>  {
        return scan(Optional<E>.none, accumulator: { self.accumulator($0, $1) }).unwrap()
    }
}
```
Let's test this with above publisher.
Results:
```
accumulativeMin -> subscribed
accumulativeMin -> Event next(9.0)
accumulativeMin -> Event next(8.0)
accumulativeMin -> Event next(7.0)
accumulativeMin -> Event next(6.0)
accumulativeMin -> Event next(5.0)
accumulativeMin -> Event completed
accumulativeMin -> isDisposed
```

Seems to be so good so far. But how about custom classes/structs? Easily, I just need to make my models conform to Comparable protocol.

```
struct MyStruct: Comparable {
    var a: Int
    public static func <(lhs: MyStruct, rhs: MyStruct) -> Bool {
        return lhs.a < rhs.a
    }
}
```
Now I'm able to use my operators on Observable<MyStruct> instances. Then if I don't have rights to extend that model, or I want to use different comparators depended on situations, I will write another custom operator to receive a custom comparator instead of default `<`.
```
extension ObservableType {
    func min(by areIncreasingOrder: @escaping (Self.E, Self.E) -> Bool) -> Observable<Self.E> {
        return reduce(Optional<E>.none, accumulator: { (min, e) -> Optional<E> in
            if let m = min, areIncreasingOrder(m, e) {
                return m
            }
            return e
        }).unwrap()
    }
}
```
You can easily apply the same way I made `accumulativeMin` to write an `accumulativeMin` which accepts a custom comparator.

You may ask: "When using `accumulativeMin`, I don't want minimum values to be emitted if it wasn't changed." Then you could apply `distinctUntilChanged` to distinguish them.

By applying `reduce` or `scan`, you may create many more similar convenient operators such as `average`, `sum`, `count`, `max`,...