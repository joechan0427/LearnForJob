# 奇怪的排序算法
[排序](https://zhuanlan.zhihu.com/p/53464092)

# rand5 实现 rand7
当实现比 randn 更小的数时, 可以直接丢弃大于 x 的值
```java
Rand5 rand5 = new Rand5();
public int rand3(Rand5 rand5) {
    int ret = Math.MAX_VALUE;
    while (ret > 3) {
        ret = rand5.nextInt();
    }
    return ret;
}
```

因此, 基于这样的思想, 我们可以构造一个大于目标数字的区间, 计算公式
```math
n * (randN - 1) + randN
```
如 rand5, rand5-1 = 0, 1, 2, 3, 4, 再乘 5, 则得到 0, 5, 10, 15, 20 的概率相等, 再加上 rand5, 则每个的概率为 1/5 * 1/5

```java
Rand5 rand5 = new Rand5();
public int rand7(Rand5 rand5) {
    int ret = Math.MAX_VALUE;
    // 能减少运行时间
    while (ret > 21) {
        ret = rand5.nextInt();
    }
    return ret % 7 + 1;
}
```