# CompletableFuture

## 1. Future

**Future只能通过阻塞或者轮询的方式得到任务的结果**

方法                                  | 说明
------------------------------------ | -------------
boolean cancel(boolean interruptIf)  | 取消任务的执行
boolean isCancelled()                | 任务是否已取消，任务正常完成前将其取消，返回 true
boolean isDone()                     | 任务是否已完成，任务正常终止、异常或取消，返回true
V get()	                           | 等待任务结束，然后获取V类型的结果
V get(long timeout, TimeUnit unit)   | 获取结果，设置超时时间

```java
ExecutorService service = Executors.newCachedThreadPool();

Future<String> stringTask = service.submit(new Callable<String>() {
	@Override
	public String call() throws Exception {
		return "Hello World";
	}
});

while (true) {
	if (stringTask.isDone() && !stringTask.isCancelled()) {
		String result = stringTask.get();
		System.err.println("StringTask: " + result);
		break;
	}
}
```


