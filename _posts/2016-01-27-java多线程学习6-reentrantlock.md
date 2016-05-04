---
layout:     post
title:      "Java多线程学习系列"
subtitle:   " \"6-ReentrantLock(可重入锁)\""
date:       2016-01-27 12:00:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - java多线程
---
# ReentrantLock(可重入锁)
# 介绍
重入锁
	
 * 一个ReentrantLock的拥有者是最近一次使用该锁lock了却还没有unlock的线程。
 * 如果一个线程成功的lock了，方法会成功返回，同时该线程会获得锁。如果该线程已经拥有该锁，再调用lock方法，lock方法会立马返回。
 * 
 
重要的方法

public void lock()

* 获得锁
	* 如果该锁没有被其他的线程持有，则该方法会立马返回，调用该方法的线程持有该锁，并且lock hold count(锁持有数)设置成1。
	* 如果该线程已经拥有该锁，则该方法立马返回，且lock hold count（锁持有数）增加1。
	* 如果该锁已经被其他线程持有，那么当前线程将会block直到获得该锁。获得的那个时候lock hold count（锁持有数）会被设置成1。（可以说明该锁是“独占的”，不同线程只能有一个线程占有）

public void lockInterruptibly() throws InterruptedException

* 获得锁除非当前线程被打断
	* 如果该锁没有被其他的线程持有，则该方法会立马返回，调用该方法的线程持有该锁，并且lock hold count(锁持有数)设置成1。 
	* 如果该线程已经拥有该锁，则该方法立马返回，且lock hold count（锁持有数）增加1。
	* 如果这个锁已经被其他线程持有，那么当前线程将不会被线程调度，同时当前线程会进入休眠状态直到以下两种情况之一发生：
		* 当前线程获得锁
		* 当前线程被Interrupt	
		
		
	
	
		
