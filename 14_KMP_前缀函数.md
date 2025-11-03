# 什么是前缀函数？

对于长度为n的字符串`s[0...n-1]`,我们构造一个**同样长度**的数组`prefixSuffixLen[0...n-1]`:

- `prefixSuffixLen[i]`表示子串`s[0...i]`中，最长的**真前缀**（proper prefix， 不包括整个子串本身）与**真后缀**（proper suffix, 不包括整个子串本身）** 相同的长度**；

例如 `s = ababa`

`i=4`时，子串`ababa`

前缀和后缀相同的部分是`aba`，长度为3

所以： `prefixSuffixLen[4] = 3`

