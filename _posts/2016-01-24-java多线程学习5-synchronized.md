---
layout:     post
title:      "Java多线程学习系列"
subtitle:   " \"5-synchronized\""
date:       2016-01-24 12:00:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - java多线程
---
# synchronized
## 先来看看jsr133中lock的语义

<p>There are multiple mechanisms for communicating between threads. The most basic of these methods is synchronization, which is implemented using monitors. Each object is associated with a monitor, which a thread can lock or unlock. Only one thread at a time may hold a lock on a monitor. Any other threads attempting to lock that monitor are blocked until they can obtain a lock on that monitor.</p>
<p>A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation.</p>
<p>The synchronized statement computes a reference to an object; it then attempts to perform a lock action on that object’s monitor and does not proceed further until the lock action has successfully completed. After the lock action has been performed, the body of the synchronized statement is executed. If execution of the body is ever completed, either normally or abruptly, an unlock action is automatically performed on that same monitor.
A synchronized method automatically performs a lock action when it is invoked; its body is not executed until the lock action has successfully completed. If the method is an instance method, it locks the monitor associated with the instance for which it was invoked (that is, the object that will be known as this during execution of the body of the method). If the method is static, it locks the monitor associated with the Class object that represents the class in which the method is defined. If execution of the method’s body is ever completed, either normally or abruptly, an unlock action is automatically performed on that same monitor.</p>

<p>The semantics neither prevents nor requires detection of deadlock conditions. Programs where threads hold (directly or indirectly) locks on multiple objects should use conventional techniques for deadlock avoidance, creating higher-level locking primitives that don’t deadlock, if necessary.</p>

<p>Other mechanisms, such as reads and writes of volatile variables and classes provided in the java.util.concurrent package, provide alternative mechanisms for synchronization.</p>

## demo
<pre>
	<code>
package com.github.dunno.concurrent.sync;

import org.junit.Test;

import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Created by liang.he on 16/1/16.
 */
public class TestSync {
	//类锁：在代码中的方法上加了static和synchronized的锁，或者synchronized(xxx.class）的代码段
	//对象锁：在代码中的方法上加了synchronized的锁，或者synchronized(this）的代码段
	//私有锁：在类内部声明一个私有属性如private Object lock，在需要加锁的代码段synchronized(lock）

	private Integer lock = new Integer(1);

	private Integer lock2 = new Integer(2);

	private final static Integer staticLock = new Integer(3);

	//无锁
	public void noLockMethod() {
		long start = System.currentTimeMillis();
		System.out.println("noLockMethod start! start:" + start);
		int sleep = 2;
		while (sleep > 0) {
			try {
				Thread.sleep(1000);
				System.out.println("noLockMethod-lock:" + lock2);
				System.out.println("noLockMethod-lock:" + lock);
				System.out.println("noLockMethod-lock:" + staticLock);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			sleep --;
		}
		long end = System.currentTimeMillis();
		System.out.println("noLockMethod end! end:" + end + " cost:" + (end - start));
	}

	//对象锁1
	public synchronized void objectLockMethod() {
		long start = System.currentTimeMillis();
		System.out.println("objectLockMethod start! start:" + start);
		int sleep = 2;
		while (sleep > 0) {
			try {
				Thread.sleep(1000);
				System.out.println("objectLockMethod-lock:" + lock2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			sleep --;
		}
		long end = System.currentTimeMillis();
		System.out.println("objectLockMethod end! end:" + end + " cost:" + (end - start));
	}

	//对象锁2
	public void objectLockMethod2() {
		synchronized (this) {
			long start = System.currentTimeMillis();
			System.out.println("objectLockMehtod2 start! start:" + start);
			int sleep = 2;
			while (sleep > 0) {
				try {
					Thread.sleep(1000);
					System.out.println("objectLockMehtod2-lock:" + lock2);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				sleep --;
			}
			long end = System.currentTimeMillis();
			System.out.println("objectLockMehtod2 end! end:" + end + " cost:" + (end - start));
		}
	}

	//私有锁 锁变量1
	public void privateLockMethod() {
		synchronized (lock) {
			long start = System.currentTimeMillis();
			System.out.println("privateLockMethod start! start:" + start);
			int sleep = 2;
			while (sleep > 0) {
				try {
					Thread.sleep(1000);
					System.out.println("privateLockMethod1-lock:" + lock2);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				sleep --;
			}
			long end = System.currentTimeMillis();
			System.out.println("privateLockMethod end! end:" + end + " cost:" + (end - start));
		}
	}

	//私有锁 锁变量2
	public void privateLockMethod2() {
		synchronized (lock2) {
			long start = System.currentTimeMillis();
			System.out.println("privateLockMethod2 start! mills:" + start);
			int sleep = 2;
			while (sleep > 0) {
				try {
					Thread.sleep(1000);
					System.out.println("privateLockMethod2-lock:" + lock);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				sleep --;
			}
			long end = System.currentTimeMillis();
			System.out.println("privateLockMethod end! end:" + end + " cost:" + (end - start));
		}
	}

	//类锁
	public synchronized static void classLockMethod() {
		long start = System.currentTimeMillis();
		System.out.println("classLockMethod start! mills:" + start);
		int sleep = 2;
		while (sleep > 0) {
			try {
				Thread.sleep(1000);
				System.out.println("classLockMethod-lock:" + staticLock);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			sleep --;
		}
		long end = System.currentTimeMillis();
		System.out.println("classLockMethod end! end:" + end + " cost:" + (end - start));
	}

	//类锁
	public static void classLockMethod2() {
		synchronized (TestSync.class) {
			long start = System.currentTimeMillis();
			System.out.println("classLockMethod2 start! mills:" + start);
			int sleep = 2;
			while (sleep > 0) {
				try {
					Thread.sleep(1000);
					System.out.println("classLockMethod2-lock:" + staticLock);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				sleep --;
			}
			long end = System.currentTimeMillis();
			System.out.println("classLockMethod2 end! end:" + end + " cost:" + (end - start));
		}
	}

	private ReentrantLock reentrantLock = new ReentrantLock();

	//独占锁
	public void reentrantLock() {
		try {
			reentrantLock.lock();
			long start = System.currentTimeMillis();
			System.out.println("reentrantLock start! start:" + start);
			int sleep = 2;
			while (sleep > 0) {
				try {
					Thread.sleep(1000);
					System.out.println("reentrantLock-lock:" + lock2);
					System.out.println("reentrantLock-lock:" + lock);
					System.out.println("reentrantLock-lock:" + staticLock);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				sleep --;
			}
			long end = System.currentTimeMillis();
			System.out.println("reentrantLock end! end:" + end + " cost:" + (end - start));
		} finally {
			reentrantLock.unlock();
		}
	}

	private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();


	public void readWriteLock() {

	}

	//测试私有锁锁不同变量是否会有竞争
	//结论:
	//只打开注释1,注释2,可以说明“当私有锁监视同一资源会竞争,等待其中一个资源释放,当监视的资源不是同一个不会产生竞争”
	//只打开注释3,注释4,可以说明“对象锁会互相竞争，等待另一个方法结束”
	//只打开注释5,注释6,可以说明"类锁会互相竞争，等待另一个释放资源"
	//只打开注释1,注释6,打开注释1,注释5,可以说明"不同的锁不会竞争"
	//只打开注释1,注释7,3-7或者5-7,可以说明"无锁变量不会和锁变量竞争"
	//不同类的锁不会竞争
	@Test
	public void test1() {
		Thread thread = new Thread(new Runnable() {
			@Override
			public void run() {
				Thread thread1 = new Thread(new Runnable() {
					@Override
					public void run() {

						//privateLockMethod();//注释1
						//objectLockMethod();//注释3
						//classLockMethod();//注释5
						reentrantLock();//注释8
					}
				});
				Thread thread2 = new Thread(new Runnable() {
					@Override
					public void run() {

						//privateLockMethod2();//注释2
						//objectLockMethod2();//注释4
						//classLockMethod2();//注释6
						//noLockMethod();//注释7
						reentrantLock();//注释9
					}
				});
				thread1.start();
				thread2.start();

				try {
					thread1.join();
					thread2.join();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

			}
		});
		thread.start();
		try {
			thread.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
</code>
</pre>