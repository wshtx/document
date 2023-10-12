KMP算法

https://segmentfault.com/a/1190000040980202 

##### next数组的两种初始化方式

- next[0] = -1 next[1] = 0
- next[0] = 0 next[1] = 1

 ![preview](https://segmentfault.com/img/bVcV6RH/view) 



##### 求next数组的过程

求next数组的过程可以看成字符串匹配的过程，即**以模式字符串为主字符串，以模式字符串的前缀为目标字符串**，一旦字符串匹配成功，那么当前的next值就是匹配成功的字符串的长度。 

 ![preview](https://pic1.zhimg.com/v2-645f3ec49836d3c680869403e74f7934_r.jpg?source=1940ef5c) 

 ![preview](https://pic2.zhimg.com/v2-06477b79eadce2d7d22b4410b0d49aba_r.jpg?source=1940ef5c) 

 ![preview](https://pic1.zhimg.com/v2-8a1a205df5cad7ab2f07498484a54a89_r.jpg?source=1940ef5c) 

![preview](https://pic1.zhimg.com/v2-f2b50c15e7744a7b358154610204cc62_r.jpg?source=1940ef5c) 

 ![preview](https://pic3.zhimg.com/v2-bd42e34a9266717b63706087a81092ac_r.jpg?source=1940ef5c) 