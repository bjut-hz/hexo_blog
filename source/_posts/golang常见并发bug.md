---
title: golang常见并发bug
date: 2019-03-16 21:47:43
categories:
- golang
tags:
- 并发bug
---



## golang并发编程

并发编程有两种方案：共享内存及消息传递。常见的编程语言(c/c++、java等)只提供了共享内存的方式进行变并发编程，并提供了锁、信号量等的基础操作来避免race condition带来的bug。

golang语言不仅支持了共享内存的方式，还支持消息传递的方案，并且提倡使用消息传递来共享数据。golang在语言级别内置了goroutine(一种轻量级的用户级线程)，提倡使用channel以及goroutine来进行并发编程。

因此，使用golang进行并发编程时，有两类并发bug，一种就是基于共享内存模型，这类bug(例如死锁)与用c++、java进行并发编程解决方案一样。另一种就是基于消息传递模型引发的并发bug，需要特别注意。

<!-- more -->

## golang常见并发bug

下面列举常见的并发bug，并分析原因，给出解决方案。谨记这些常见的错误，在以后的使用golang并发编程实战中防止再犯类似错误。

- 阻塞bug

  - channel

  ```go
  1 func finishReq(timeout time.Duration) r ob {
  2     - ch := make(chan ob)
  3     + ch := make(chan ob, 1)
  4     go func() {
  5         result := fn()
  6         ch <- result // block
  7     } ()
  8     select {
  9         case result = <- ch:
  10            return result
  11        case <- time.After(timeout):
  12            return nil
  13    }
  14 }
  ```

  **错误原因：** `finishReq`函数使用匿名函数创建了一个子goroutine来处理请求，并把结果通过ch返回给父goroutine。但是如果在子goroutine中处理请求的时候超时了， 就会导致父goroutine超时返回。那么这个时候，没有其他的goroutine从ch中读取数据了，此时创建的子goroutine会一直阻塞，导致资源泄露。

  **解决方案： ** 修改的方法只需要把ch改成缓冲的channel即可

  - WaitGroup

    ```go
    1 var group sync.WaitGroup
    2 group.Add(len(pm.plugins))
    3 for _, p := range pm.plugins {
    4 	go func(p *plugin) {
    5 		defer group.Done()
    6 	}
    7 	- group.Wait()
    8 }
    9 + group.Wait()
    ```

    **错误原因：** 在使用WaitGroup的时候，需要保证Add(N)的参数N与Done调用的次数相同。在本例中，由于在循环内部调用了Wait函数，导致主goroutine会阻塞，因此不会再创建新的子goroutine。主goroutine会一直阻塞在第7行。

    **解决方案：** 把Wait调用放在循环之外。

  - context 

    ```go
    1 - hctx, hcancel := context.WithCancel(ctx)
    2 + var hctx context.Context
    3 + var hcancel context.CancelFunc
    4 	if timeout > 0 {
    5 		hctx, hcancel = context.WithTimeout(ctx, timeout)
    6 + } else {
    7 + 	hctx, hcancel = context.WithCancel(ctx)
    8 	}
    ```

    **错误原因：** 在第一行代码中在创建了hctx及hcancel的同时，也创建了一个新的goroutine，可以通过hcancel来向该goroutine发送消息，进行并发控制。但是在后续的代码中hcancel被赋予了新的值，那么就没有其他的方式来关闭新创建的goroutine。

    **解决方案：** 只进行变量的定义，然后根据不同的情形进行赋值

  - wrong usage of channel with lock

    ```go
    1 func goroutine1() {
    2 	m.Lock()
    3 - ch <- request //blocks
    4 + select {
    5 + 	case ch <- request
    6 + 	default:
    7 + }
    8 	m.Unlock()
    9 }
    
    1 func goroutine2() {
    2 	for {
    3 		m.Lock() //blocks
    4 		m.Unlock()
    5 		request <- ch
    6 	}
    7 }
    ```

    **错误原因：** 在goroutine1中，获取锁之后，对ch的写操作导致该goroutine会一直阻塞。然而在goroutine中，Lock操作会导致该goroutine一直阻塞。

    **解决方案：** 在goroutine1中使用select操作，并添加default分支，避免写ch阻塞。



- 非阻塞bug

  - data race caused by anonymous function

    ```go
    1 	for i := 17; i <= 21; i++ { // write
    2 - 	go func() { /* Create a new goroutine */
    3 + 	go func(i int) {
    4 			apiVersion := fmt.Sprintf("v1.%d", i) // read
    5 			...
    6 - 	}()
    7 + 	}(i)
    8 	}
    ```

    **错误原因：** 在golang中，闭包捕获外部变量时，持有的是引用。因此appVersion的值不确定，可能去那是21。

    **解决方案：** 使用函数参数传递变量。函数参数的传递是按值传递的，并且会复制一份。

  - WaitGroup

    ```go
    1 	func (p *peer) send() {
    2 		p.mu.Lock()
    3 		defer p.mu.Unlock()
    4 		switch p.status {
    5 			case idle:
    6 + 			p.wg.Add(1) // has to be invoked before Wait
    7 				go func() {
    8 - 				p.wg.Add(1)
    9 					...
    10 					p.wg.Done()
    11 				}()
    12 			case stopped:
    13 		}
    14 	}
    
    1 	func (p * peer) stop() {
    2 		p.mu.Lock()
    3 		p.status = stopped
    4 		p.mu.Unlock()
    5 		p.wg.Wait()
    6 	}
    ```

    **错误原因：** 在使用WaitGroup时，Add函数必须在Wait之前被调用。在上述代码中，状态是idle时，可能在解锁之后，Add函数还未执行，因此并不能保证该规则。

    **解决方案：** 把Add函数添加在受锁保护访问临界资源的代码块内。

  - 重复关闭channel

    ```go
    1 - select {
    2 - 	case <- c.closed:
    3 - 	default:
    4 + 		Once.Do(func() {
    5 				close(c.closed)
    6 + 		})
    7 - }
    ```

    **错误原因：** 当多个goroutine执行这一块代码时，会导致c.closed管道被多次关闭，引发bug。

    **解决方案：** 使用Once

  - select and channel

    ```go
    1 	ticker := time.NewTicker()
    2 	for {
    3 + 	select {
    4 + 		case <- stopCh:
    5 + 			return
    6 + 		default:
    7 + 	}
    8 		f()
    9 		select {
    10 			case <- stopCh:
    11 				return
    12 			case <- ticker:
    13 		}
    14 	}
    ```

    **错误原因：** select语句中如果有多个分支满足条件，那么会随机选择一个分支进行执行。在上述代码中，如果f()需要执行很长时间，那么会存在stopCh与ticker同时满足条件的情况，如果选择了ticker分支继续执行，那么就会导致f()函数会被多执行一次。

    **解决方案：** 在f()开始之前多次判断stopCh

  - Timer

    ```go
    1 - timer := time.NewTimer(0)
    2 + var timeout <- chan time.Time
    3 	if dur > 0 {
    4 - 	timer = time.NewTimer(dur)
    5 + 	timeout = time.NewTimer(dur).C
    6 	}
    7 	select {
    8 - 	case <- timer.C:
    9 + 	case <- timeout:
    10 		case <-ctx.Done():
    11 			return nil
    12 	}
    ```

    **出错原因：** 第一行创建timer后，go运行时就开始通过internal goroutine计时。如果dur不大于0时，那么timer.C会立即满足条件，导致该函数提早结束，ctx.Done没有机会执行。

    **解决方案：** 使用Time结构，不立即进行计时操作。只有在dur大于0时，才开始计时操作



## reference

[1] Tengfei Tu, Xiaoyu Liu, Linhai Song, and Yiying Zhang. 2019. Understanding Real-World Concurrency Bugs in Go . In Proceedings of 2019 Architectural Support for Programming Languages and Operating Systems (ASPLOS’19). ACM, New York, NY, USA, 14 pages. https://doi.org/http://dx.doi.org/10.1145/3297858.3304069