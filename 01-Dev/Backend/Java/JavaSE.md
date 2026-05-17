# 一.字符串
## 1.String，StringBuilder，StringBuffer有什么区别？
+ String是不可变字符序列，每次修改都会new一个对象，线程安全
+ StringBuilder是可变的，线程不安全，但效率高
+ StringBuffer是可变的，线程安全，使用synchronized修饰，但效率低

# 二.方法
## 1.为什么重写equals()要重写hashcode()？
为了确保对象在哈希集合中正常工作，因为需要hashcode来分配存储桶的位置。这些集合在寻找对象位置时，会先通过hashcode来找到位置，在通过equals来精确比较。如果不重写，可能两个逻辑相同的对象有不同的哈希值，导致哈希集合存储重复元素，无法正常查找对象。

