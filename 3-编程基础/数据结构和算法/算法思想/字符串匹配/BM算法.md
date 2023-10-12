https://blog.csdn.net/qq_42642142/article/details/115287325

##### 坏字符规则

主串中，与模式串不匹配的字符叫做坏字符。在模式串中查找坏字符，

- 坏字符不存在时，将模式串移动至坏字符之后的位置，继续查找；

   ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzRlLzY0LzRlMzZjNGQ0OGQxYjZjM2I0OTlmYjAyMWYwM2M3ZjY0LmpwZw?x-oss-process=image/format,png)

- 坏字符存在时，将模式串中的字符与坏字符对齐，继续查找。

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlL2E4L2NhL2E4ZDIyOWFhMjE3YTY3MDUxZmJiMzFiOGFlYjJlZGNhLmpwZw?x-oss-process=image/format,png)

- 坏字符在模式串中存在多个时，选择最靠后的；

 设坏字符对应的模式串中的字符下标记作 si。如果坏字符在模式串中存在，我们把这个坏字符在模式串中的下标记作 xi。如果不存在，我们把 xi 记作 -1。那模式串往后移动的位数就等于 **si-xi**。 

 单纯使用坏字符规则还是不够的。因为根据 si-xi 计算出来的移动位数，有可能是负数，比如主串是 cccccccccccc，模式串是 accc。不但不会向后滑动模式串，还有可能倒退，因为根据以上计算方式，si = 0 ,xi = 1, si - xi = -1。 

##### 好后缀规则

规则类似坏字符， 把已经匹配的 bc 叫作好后缀，记作{u}。在模式串中查找：

- 如果找到了另一个跟{u}相匹配的子串{u * }，那我们就将模式串滑动到子串{u * }与主串中{u}对齐的位置。 

 ![img](https://img-blog.csdnimg.cn/20210328205415715.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjQyMTQy,size_16,color_FFFFFF,t_70#pic_center) 

- 找不到时，判断好后缀的后缀子串与模式串的前缀是否存在匹配的情况；若存在，则将最长的模式串的前缀和好后缀的后缀子串对齐；

   ![img](https://img-blog.csdnimg.cn/20210328205441251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjQyMTQy,size_16,color_FFFFFF,t_70#pic_center) 

##### 如何选择规则？

 分别计算好后缀和坏字符往后滑动的位数，然后取两个数中最大的，作为模式串往后滑动的位数。 



