## til
Today I Learned

## Categories ##
1. [RxSwift](#rxswift)

### RxSwift
1. Unsubscribe from an observable without a dispose bag in RxSwift
  DisposeBag is the most common way to manage subscriptions in RxSwift world. However, sometimes you don't have an existing dispose bag or what you need is only a temporary observable. There is a hidden gem - the rx extension on `AnyObject` called `deallocated`. It's an observable of type Void and will emit events when that object is deallocated. A normal use case will be applying `takeUntil(obj.rx.deallocated)` to the observable. 
  For example, I want to add a subview to my root view, I want to dismiss keyboard when users tap on that view. I would subscribe for gestures and takeUntil the view is removed from its superview and deallocated.
  ```
        _ = container.rx.tapGesture()
        .when(.recognized)
        .takeUntil(container.rx.deallocated)
        .subscribe(onNext: { [weak self] (_) in
            self?.view.endEditing(true)
        })
```
