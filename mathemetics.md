# 1 二进制
## 1.1 计算机为什么要使用二进制？

  - 二进制具有抗干扰能力强、可靠性高的优点

  - 二进制非常适合做逻辑运算
## 1.2 二进制的位操作

| 符号   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| <<     | 向左移位，向左移一位，就是将数据翻倍                         |
| \>\>\> | 算术右移，算术右移保持符号位不变，除符号位之外的右移一位并补符号位1，补的1仍然在符号位之后 |
| \>\>   | 逻辑右移，逻辑右移一位，左边补0即可                          |
| \|     | 参与或操作的位只要有一个是1，最终结果就是1                   |
| &      | 参与与操作的位必须全部是1，最终结果才会是1                   |
| ^      | 参与异或操作的位相同，结果为0，反之则为1                     |

## 1.3 原码、反码、补码
  - 正数的原码、反码及补码都一样
  - 负数的反码=原码除符号位外按位取反
  - 负数的补码=反码+1

# 2 余数

- 余数的特性
  - 整数是没有边界的，它可能是正无穷，也可能是负无穷。余数却总是在一个固定的范围内。余数可以用来计算星期，web编程中可以用在分页中。
- 同余定理
  - 两个整数a和b，如果它们除以正整数m得到的余数相等，我们就可以说，和b对于模m同余。同余定理其实就是用来分类的。
- 求余过程就是个哈希函数
  - 每个编程语言都有对应的哈希函数。哈希就是将任意长度的输入，通过哈希算法压缩为某一固定长度的输出。

# 3 迭代法

- 什么是迭代法
  - 就是不断地用旧的变量值，递推计算新的变量值。迭代法的思想很容易通过计算机语言中的循环语言来实现。我们可以通过循环语句，让计算机重复执行迭代中的递推步骤，推导出变量的最终值
- 迭代法的基本步骤
  - 确定用于迭代的变量
  - 建立迭代变量之间的递推关系
  - 控制迭代的过程
- 迭代法的具体应用
  - 求数值的精确或者近似解
  - 在一定范围内查找目标值
  - 机器学习算法中的迭代