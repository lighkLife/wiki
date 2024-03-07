# String 实现

## 存储结构
- Java 9 以前，String 使用 char[] 来存储字符串，字符串的长度就是 char[] 的大小；
- Java 9 及以后，String 使用 byte[] 来存储字符串，并且新增`COMPACT_STRINGS`
代表当前是否采用压缩存储，`coder`代表当前存储方式（`LATIN1` 或 `UTF16`），具体一个字符占用多少个字节还是取决于字符编码和字符本身的码值。

```java
    static final byte LATIN1 = 0;
    static final byte UTF16  = 1;

    static final boolean COMPACT_STRINGS;
    private final byte coder;

    public String(int[] codePoints, int offset, int count) {
         // ...
        if (COMPACT_STRINGS) {
            this.coder = LATIN1;
            return;
        }
        this.coder = UTF16;
        // ...
    }

    public int length() {
        return value.length >> coder();
    }
```

## 为什么改为 char[]

采用更节省空间的字符串内部表示形式。参考 [JEP254](https://openjdk.org/jeps/254) 。

String 类的当前实现将字符存储在 char 数组中，每个字符使用两个字节（十六位）。从许多不同应用程序收集的数据表明字符串是堆使用的主要组成部分，而且大多数 String 对象仅包含 Latin-1 字符。此类字符仅需要一个字节的存储空间，因此此 String 对象的内部 char 数组中的一半空间未使用,存在浪费。


参考：
1. [Java 9为什么要将String的底层实现由char数组改成了byte数组?](https://javabetter.cn/basic-extra-meal/jdk9-char-byte-string.html)
2. [Java一个汉字占几个字节（详解与原理）](https://www.cnblogs.com/lslk89/p/6898526.html)

