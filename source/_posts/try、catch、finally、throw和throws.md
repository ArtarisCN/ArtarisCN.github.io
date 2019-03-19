---
title: try、catch、finally、throw和throws
date: 2017-07-10 09:48:02
tags: [Java]
categories:
- Java
---
try、catch、finally 是 Java 中处理异常的一套机制。

<!-- more -->

完整的代码流程如下：

```
    try {
        //尝试执行的代码
    } catch (Exception e){
        //捕获的异常
    } finally {
        //最后总是会执行的代码
    }
```

在 try 代码块中执行尝试的代码，在 catch 中捕获可能在 try 代码块会出现的异常并做相应的处理，最后在 finally 代码块执行总会执行的代码。这是简单的行文，当在这些代码块中出现了 return 时会按照什么顺序执行呢？

1. 在不抛出异常的情况下，程序执行完 try 里面的代码块之后，该方法并不会立即结束，而是继续试图去寻找该方法有没有 finally 的代码块；
2. 如果没有 finally 代码块，整个方法在执行完 try 代码块后返回相应的值来结束整个方法；
3. 如果有 finally 代码块，此时程序执行到 try 代码块里的 return 语句之时并不会立即执行 return，而是先去执行 finally 代码块里的代码；
4. 若 finally 代码块里没有 return 或没有能够终止程序的代码，程序将在执行完 finally 代码块代码之后再返回 try 代码块执行 return 语句来结束整个方法；
5. 若 finally 代码块里有 return 或含有能够终止程序的代码，方法将在执行完 finally 之后被结束，不再跳回 try 代码块执行 return；

有异常时将上述 try 代码块替换成 catch 即可。

当程序遇见 `System.exit(0);` 时不管执行到什么地方都会立即执行推出当前进程。
