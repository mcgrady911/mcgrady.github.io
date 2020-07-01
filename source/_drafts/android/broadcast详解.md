# Broadcast

## BroadcastReceiver
- `onReceive()` 方法中不能进行复杂工作否则会导致 ANR，`onReceive()` 方法一旦执行完，系统可能就认为这个广播接收器已经没用了，随时会杀掉包含这个广播接收器的进程，包括这个进程启动的线程。使用 `goAsync()` 方法可以在 `PendingResult#finish()` 方法执行前为广播接收器的存活争取更多的时间，但最好还是使用 `JobScheduler` 等方式进行长时间处理工作
- `sendBroadcast()`发送常规广播，广播接收器收到的顺序是不可控的。
- `sendOrderedBroadcast()`发送有序广播，根据广播接收器的优先级有序的传递广播，相同优先级的顺序不可控，广播接收器可以选择继续传递给下一个，也可以选择直接丢掉。
- 为了保证广播的`action`全局唯一性，最好声明应用的包名成静态字符串常量作为前缀。