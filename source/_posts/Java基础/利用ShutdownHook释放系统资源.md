---
title: Java知识点系列之--利用ShutdownHook释放系统资源
date: 2020-07-20 13:05:00
updated: 2020-07-20 13:05:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 如何利用ShutdownHook释放系统资源？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 参考书籍：《Java特种兵（上册）》 

当发生 `System.exit(int status)` 时，希望在系统退出前，执行一点任务来做一些资源方面的回收操作，**ShutdownHook** 可以达到这个目的，它利用 **hook** 的思路来实现，有些时候也把它叫作“钩子”。

假如在系统中通过 `Runtime.getRuntime().exec(String command)` 或 `new ProcessBuilder(List<String> command)` 启动了子进程`（Process）`，这个子进程一直在运行中，在当前进程退出时，子进程未必会退出，但此时业务上希望将它退出，就可以利用 `ShutdownHook` 。例如下面这个测试样例： 
```Java
	public class ShutdownHookTest {
	 
		public static void main(String[] args) {
			Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
				public void run() {
					System.out.println("执行ShutdownHook钩子操作，释放系统资源...");
				}
			}));
			Runtime.getRuntime().removeShutdownHook(new Thread());
			System.exit(1);
			System.out.println("主进程结束...");
		}
	}
```
执行结果：
```
主进程结束...
执行ShutdownHook钩子操作，释放系统资源...
```

> <font color=red>注意：</font>传入参数是通过 new Thread() 创建的线程对象，在Java进程调用 exit() 时，会调用该线程对象的 start()方法将其运行起来，所以不要手工先启动了。另外，这种回调线程就不要设定为死循环程序，否则就无法退出了。

# ShutdownHook执行原理

`java.lang`包下有一个`Shutdown` 类，提供了一个 `Runnable[] hooks` 数组，数组的长度为**10**，也就是最多定义**10**个**hook**（这与程序写入多少个线程回调没有关系），提供 `add()` 方法来注册新的 **hook** 对象。 
```Java
    // The system shutdown hooks are registered with a predefined slot.
    // The list of shutdown hooks is as follows:
    // (0) Console restore hook
    // (1) Application hooks
    // (2) DeleteOnExit hook
    private static final int MAX_SYSTEM_HOOKS = 10;
    private static final Runnable[] hooks = new Runnable[MAX_SYSTEM_HOOKS];
    /* Add a new shutdown hook.  Checks the shutdown state and the hook itself,
     * but does not do any security checks.
     */
    static void add(int slot, Runnable hook) {
        synchronized (lock) {
            if (state > RUNNING)
                throw new IllegalStateException("Shutdown in progress");
 
            if (hooks[slot] != null)
                throw new InternalError("Shutdown hook at slot " + slot + " already registered");
 
            hooks[slot] = hook;
        }
    }
```
在`java.lang.ApplicationShutdownHooks` 的 `static` 匿名块中，通过 `add()` 方法注册了一个**hook** ，在 `run()` 方法内部运行内部的静态方法 `runHooks()` ，它内部用一个 `IdentityHashMap` 来存放 **hook** 信息，可以通过 `add()` 方法来添加，而程序中定义的回调线程都放在了这里，它自身用 **hook** 的形式存在于 **hooks** 列表当中。 
```Java
	package java.lang;
	 
	import java.util.*;
	 
	/*
	 * Class to track and run user level shutdown hooks registered through
	 * <tt>{@link Runtime#addShutdownHook Runtime.addShutdownHook}</tt>.
	 *
	 * @see java.lang.Runtime#addShutdownHook
	 * @see java.lang.Runtime#removeShutdownHook
	 */
	 
	class ApplicationShutdownHooks {
	    static {
	        Shutdown.add(1 /* shutdown hook invocation order */,
	            new Runnable() {
	                public void run() {
	                    runHooks();
	                }
	            });
	    }
	 
	    /* The set of registered hooks */
	    private static IdentityHashMap<Thread, Thread> hooks = new IdentityHashMap<Thread, Thread>();
	 
	    private void ApplicationShutdownHooks() {}
	 
	    /* Add a new shutdown hook.  Checks the shutdown state and the hook itself,
	     * but does not do any security checks.
	     */
	    static synchronized void add(Thread hook) {
		if(hooks == null)
		    throw new IllegalStateException("Shutdown in progress");
	 
		if (hook.isAlive())
		    throw new IllegalArgumentException("Hook already running");
	 
		if (hooks.containsKey(hook))
		    throw new IllegalArgumentException("Hook previously registered");
	 
	        hooks.put(hook, hook);
	    }
	 
	    /* Remove a previously-registered hook.  Like the add method, this method
	     * does not do any security checks.
	     */
	    static synchronized boolean remove(Thread hook) {
		if(hooks == null)
		    throw new IllegalStateException("Shutdown in progress");
	 
		if (hook == null) 
		    throw new NullPointerException();
	 
		return hooks.remove(hook) != null;
	    }
	 
	    /* Iterates over all application hooks creating a new thread for each
	     * to run in. Hooks are run concurrently and this method waits for 
	     * them to finish.
	     */
	    static void runHooks() {
		Collection<Thread> threads;
		synchronized(ApplicationShutdownHooks.class) {
		    threads = hooks.keySet();
		    hooks = null;
		}
	 
		for (Thread hook : threads) {
		    hook.start();
		}
		for (Thread hook : threads) {
		    try {
			hook.join();
		    } catch (InterruptedException x) { }
		}
	    }
	}
```
当调用 `Runtime.getRuntime().addShutdownHook(Thread hook)`方法时，会间接调用 `ApplicationShutdownHooks.add(Thread hook)`将线程放到`IdentityHashMap` 中，`Runtime.getRuntime().removeShutdownHook(Thread)`用于删除一个钩子线程。 
```Java
	public void addShutdownHook(Thread hook) {
		SecurityManager sm = System.getSecurityManager();
		if (sm != null) {
		    sm.checkPermission(new RuntimePermission("shutdownHooks"));
		}
		ApplicationShutdownHooks.add(hook);
	}
	public boolean removeShutdownHook(Thread hook) {
		SecurityManager sm = System.getSecurityManager();
		if (sm != null) {
		    sm.checkPermission(new RuntimePermission("shutdownHooks"));
		}
		return ApplicationShutdownHooks.remove(hook);
	}
```
当调用 `System.exit(int status)`方法时，会间接调用 `Shutdown.exit(int status)`方法，再调用`sequence()-->runHooks()` 方法。 
```Java
	/* Invoked by Runtime.exit, which does all the security checks.
	 * Also invoked by handlers for system-provided termination events,
	 * which should pass a nonzero status code.
	 */
	static void exit(int status) {
		boolean runMoreFinalizers = false;
		synchronized (lock) {
		    if (status != 0) runFinalizersOnExit = false;
		    switch (state) {
		    case RUNNING:	/* Initiate shutdown */
			state = HOOKS;
			break;
		    case HOOKS:		/* Stall and halt */
			break;
		    case FINALIZERS:
			if (status != 0) {
			    /* Halt immediately on nonzero status */
			    halt(status);
			} else {
			    /* Compatibility with old behavior:
			     * Run more finalizers and then halt
			     */
			    runMoreFinalizers = runFinalizersOnExit;
			}
			break;
		    }
		}
		if (runMoreFinalizers) {
		    runAllFinalizers();
		    halt(status);
		}
		synchronized (Shutdown.class) {
		    /* Synchronize on the class object, causing any other thread
	             * that attempts to initiate shutdown to stall indefinitely
		     */
		    sequence();
		    halt(status);
		}
	}
	/* The actual shutdown sequence is defined here.
	 *
	 * If it weren't for runFinalizersOnExit, this would be simple -- we'd just
	 * run the hooks and then halt.  Instead we need to keep track of whether
	 * we're running hooks or finalizers.  In the latter case a finalizer could
	 * invoke exit(1) to cause immediate termination, while in the former case
	 * any further invocations of exit(n), for any n, simply stall.  Note that
	 * if on-exit finalizers are enabled they're run iff the shutdown is
	 * initiated by an exit(0); they're never run on exit(n) for n != 0 or in
	 * response to SIGINT, SIGTERM, etc.
	 */
	private static void sequence() {
		synchronized (lock) {
		    /* Guard against the possibility of a daemon thread invoking exit
		     * after DestroyJavaVM initiates the shutdown sequence
		     */
		    if (state != HOOKS) return;
		}
		runHooks();
		boolean rfoe;
		synchronized (lock) {
		    state = FINALIZERS;
		    rfoe = runFinalizersOnExit;
		}
		if (rfoe) runAllFinalizers();
	}
	/* Run all registered shutdown hooks
	 */
	private static void runHooks() {
		/* We needn't bother acquiring the lock just to read the hooks field,
		 * since the hooks can't be modified once shutdown is in progress
		 */
		for (Runnable hook : hooks) {
		    try {
				if (hook != null) hook.run();
		    } catch(Throwable t) { 
				if (t instanceof ThreadDeath) {
		   		    ThreadDeath td = (ThreadDeath)t;
				    throw td;
				} 
		    }
		}
	}
```
循环中的一个`hook`对象就是由 `ApplicationShutdownHooks` 的匿名块定义的，因此会调用 `ApplicationShutdownHooks` 类的 `run()` 方法，再调用它的`runHooks()`方法，这个 `runHooks()`方法内容如下。 
```Java
	/* Iterates over all application hooks creating a new thread for each
	 * to run in. Hooks are run concurrently and this method waits for 
	 * them to finish.
	 */
	static void runHooks() {
		Collection<Thread> threads;
		synchronized(ApplicationShutdownHooks.class) {
		    threads = hooks.keySet();
		    hooks = null;
		}
	 
		for (Thread hook : threads) {
		    hook.start();
		}
		for (Thread hook : threads) {
		    try {
			hook.join();
		    } catch (InterruptedException x) { }
		}
	}
```
从这个方法中取出所有的**Thread** ，然后将线程启动起来，最后通过 `join()` 方法等待各个线程结束的动作，换句话说，在进程关闭前，对多个回调任务的处理方式是每个任务单独有一个线程处理，而不是所有的任务在一个线程中串行处理。  

# ShutdownHook适用场景

- 程序正常退出
- 使用System.exit()
- 终端使用Ctrl+C触发的中断
- 系统关闭
- OutOfMemory宕机
- 使用Kill pid命令干掉进程（注：在使用kill -9 pid时，是不会被调用的）