---
layout: post
title: 转摘-程序员应该知道的耗时
date: 2014-08-20 00:44
author: charles
comments: true
tags:
 - tech
 - repaste
 
---

- 1纳秒(ns) = 1000皮秒(ps)
- 1纳秒(ns) = 0.001微秒(μs)
- 1纳秒(ns) = 0.000001毫秒(ms) 　
- 1纳秒(ns) = 0.000000001秒(s)

###文字版###

```sh
L1 cache reference ......................... 0.5 ns
Branch mispredict ............................ 5 ns
L2 cache reference ........................... 7 ns
Mutex lock/unlock ........................... 25 ns
Main memory reference ...................... 100 ns             
Compress 1K bytes with Zippy ............. 3,000 ns  =   3 µs
Send 2K bytes over 1 Gbps network ....... 20,000 ns  =  20 µs
SSD random read ........................ 150,000 ns  = 150 µs
Read 1 MB sequentially from memory ..... 250,000 ns  = 250 µs
Round trip within same datacenter ...... 500,000 ns  = 0.5 ms
Read 1 MB sequentially from SSD* ..... 1,000,000 ns  =   1 ms
Disk seek ........................... 10,000,000 ns  =  10 ms
Read 1 MB sequentially from disk .... 20,000,000 ns  =  20 ms
Send packet CA->Netherlands->CA .... 150,000,000 ns  = 150 ms

Notes
-----
1 ns = 10-9 seconds  
1 ms = 10-3 seconds  
* Assuming ~1GB/sec SSD  
 
Credit
------
By Jeff Dean:               http://research.google.com/people/jeff/
Originally by Peter Norvig: http://norvig.com/21-days.html#answers
```

