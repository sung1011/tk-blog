---
layout: post
title: "Php-operation-mode"
tags:
 - php
description: php运行模式
---

### code
```
exec $源代码(人认识); // 执行源代码

exec $字节码(解释器认识); // 执行字节码

exec $机器码(硬件认识); // 执行机器码
```

### php code
```
exec *.php; // 执行php源代码

if(!exist($opcache)) { // 是否存在opcache字节码

    init $opcode; // zend编译生成opcode字节码

    $opcache = $opcode; // 生成opcache字节码缓存

} 

exec opcache; // php解释器执行opcache字节码缓存

exec 机器码; // 执行机器码
```
### php jit 
```

if(!exist(机器码)) { // 是否存在机器码

    init 机器码; // jit编译生成机器码

}

exec 机器码; // 执行机器码
```