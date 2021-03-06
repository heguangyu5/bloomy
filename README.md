Description
-----------

This extension implements a Bloom filter, which is a space-efficient
probabilistic data structure that is used to test whether an element is a
member of a set.

# 大致原理

1. 分配一块内存当作bit array.
2. 对输入内容做k次hash(当然是k个hash函数,不然每次hash结果都一样了),将每次hash的结果对应到bit array的一个位置上,将此处设为1.
3. 查找时一样对要查找的内容做k次hash,然后检查对应bit array的位置上是否为1,如果是,那**要查找的内容可能存在,否则一定不存在**.(这个容易理解,hash function存在冲突,hashtable通过链表解决冲突,bloom filter没有链表,所以只能说可能存在).

# 需要确定下来的值

1. 分配多大内存当作bit array?
2. 对输入内容做k次hash,k等于几?
3. 由于hash存在冲突,只能判断可能存在,允许多大误差?

@see https://en.wikipedia.org/wiki/Bloom_filter

假设我们要分配m个bits来保存n个元素,hash func个数 k = m/n * ln2

假设我们有n个元素要保存,可以接受的误差是p,那么应该分配的bits数 m = - n * lnp / (ln2)^2

# bloomy的实现

    <?php
        $o = new BloomFilter(capacity = n, error_rate = p = 默认为1%);
            // 调用 bloom_calc_optimal() 计算出需要分配的内存大小和hash func个数
            // 不明白bloomy的计算方法,它根据n,p,k计算m,从k=0开始计算,最终取m的最小值.
            // @see https://en.wikipedia.org/wiki/IEEE_floating_point
            // Division by zero (an operation on finite operands gives an exact infinite result, e.g., 1/0 or log(0)) (returns ±infinity by default).
            // 定下来m和k之后,就可以分配内存块了
        $o->add(item);
            // 调用 bloom_hash() 计算 item 的hash值, 这里计算出两个hash值
            // 然后循环k遍,设bit array的 (hash1 + k * hash2) % m 位置处的bit为1
        $o->has(item);
            // 再次计算item的hash值,同样计算出两个hash
            // 循环k遍过程中,每次都检查对应的bit位置上是否为1,如果不是,立马确定 NOTFOUND
            // 如果全为1,那就找到了
        serialize/unserialize
            // 其实就是保存/恢复bits array了
    ?>

# 关于hash function

@see http://www.burtleburtle.net/bob/hash/

这个人写了两版的hash function, 一个是bloomy用的lookup3.c, 它可以生成32位(uint32_t)的hash值,如果生成两个hash的话,那就是64位了.

另一个是SpookyV2.cpp, 它可以生成64位(uint64_t)的hash值,如果生成两个hash的话,那就是128位了.

# 应用例子

cdn做缓存,用bloom filter来记录一个网址是否之前被访问过,如果是,才缓存,否则,不缓存.

也就是说,当一个网址在第二次被访问时才缓存它.

这个方法解决了 "one-hit-wonders" 问题.
