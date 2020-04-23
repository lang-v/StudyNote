***

*2020.03.30*

 	1. 控制小数显示位数

```java

DecimalFormat decimalFormat = new DecimalFormat("0.0000000");
//7位小数
System.out.println(decimalFormat.format(r*r*PI));
```

2. 内置快排
```java

import static java.util.Arrays.sort;
sort(int[]);//升序
```

3. 进制转换

| 10进制转化其他进制 | 对应的方法,参数:n(原10进制数据),r(进制) | 返回值 |
| ------------------ | --------------------------------------- | :----- |
| 10进制转2进制 | Integer.toBinaryString(n) | 一个二进制字符串 |
| 10进制转8进制 | Integer.toOctalString(n); | 一个八进制字符串 |
| 10进制转16进制 | Integer.toHexString(n); | 一个16进制字符串 |
| 10进制转 r 进制 | Integer.toString(100, 16); | 一个r进制字符串 |
| r进制转10进制 | Integer.parseInt((String) s,(int) radix);（Long.parseLong） | 整型 |

4. 任意大 的数据转换进制

   ```java
   import java.math.BigInteger;
   BigInteger big = new BigInteger("16548ABC",16).toString(8);
   //16进制转8进制，返回字符串
   ```

5. 

