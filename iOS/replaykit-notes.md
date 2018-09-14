ReplayKit is a framework of Apple allows you to capture or record your screen at a very high frame rate. I have used ReplayKit to broadcast app screen frames (quite similar to livestreaming apps out there but this is screen livestreaming apps).

- From the app design step, you must be aware that ReplayKit captures the whole main window. It means every UI controls on the screen will be captured that you may not want to. A simplest case is that you should move your stop button and UI controls you dont want to be captured (maybe user private data) to another window.

- ReplayKit captures the main window. If you are exposed to Cocoa API for long enough, you must know what is key window, but what is main window. At the time I write this post, main window in iOS is the first window you create and bind into the application. If you want to display another window maybe an alert, a popup you use `window.makeKeyAndVisible()` then it will become key window. But the main window is still the first window you create.

- ReplayKit calls callbacks on different threads. It is not guaranteed that callbacks are called on the same thread. So it is your responsibility to understand what's going on with your frames. This is an example log from my app fyi.

```
<NSThread: 0x1c08692c0>{number = 32, name = (null)}
<NSThread: 0x1c6860640>{number = 28, name = (null)}
<NSThread: 0x1c2c64f40>{number = 38, name = (null)}
<NSThread: 0x1c6860640>{number = 28, name = (null)}
<NSThread: 0x1c2a79f00>{number = 35, name = (null)}
<NSThread: 0x1c08692c0>{number = 32, name = (null)}
<NSThread: 0x1c08692c0>{number = 32, name = (null)}
<NSThread: 0x1c08692c0>{number = 32, name = (null)}
<NSThread: 0x1c7073480>{number = 37, name = (null)}
<NSThread: 0x1c7073480>{number = 37, name = (null)}
<NSThread: 0x1c7073480>{number = 37, name = (null)}
<NSThread: 0x1c08692c0>{number = 32, name = (null)}
<NSThread: 0x1c2a79f00>{number = 35, name = (null)}
<NSThread: 0x1c7073480>{number = 37, name = (null)}
```

So if you want these frames are sent in correct order you need to verify yourself.