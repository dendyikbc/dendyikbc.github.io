---
layout: post
title: '口口相传的 大端序 与 小端序'
subtitle: '关于字节顺序的解释与测试'
date: 2020-05-02
author: Dave
cover: ''
tags: Java
---



### 大端/小端模式
- [字节顺序](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)

>以 **0x12345678** 存放到内存地址 **0x400-0x403** 为例
低字节：0x78
高位内存：0x403


|        | 说明           | 0x400  | 0x401  |0x402|0x403(**高位内存**)|
| ------------- |:-------------:| -----:|-----:|-----:|-----:|
| 大端模式    | 低字节在高位内存地址 | 0x12(**高字节**) |0x34|0x56|0x78|
| 小端模式    | 低字节在低位内存地址 |   0x78 |0x56|0x34|0x12|


```java
//对于字节数组b[3]为高位内存
//这里是判断java的大小端，而非CPU
import java.io.ByteArrayOutputStream;
import java.io.DataOutputStream;

public class Main {
 
    /**
     * @param args
     */
    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        int a = 0x12345678;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream dos = new DataOutputStream(baos);
        dos.writeInt(a);
        byte[] b = baos.toByteArray();
        for(int i = 0;i<4;i++){
            System.out.println("b["+i+"]:"+Integer.toHexString(b[i]));
        }    
    }
}
/*输出结果
b[0]:12
b[1]:34
b[2]:56
b[3]:78
*/
```




但其实，JAVA中可以偷懒的测试
- **java.nio.ByteOrder**

```
*ByteOrder定义了写入buffer时字节的顺序
---java默认是 BIG_ENDIAN
---但是cpu不一定，Intel的x86默认是LITTLE_ENDIAN

*API
---2个内置的ByteOrder
ByteOrder.BIG_ENDIAN 和 ByteOrder.LITTLE_ENDIAN

---ByteOrder.nativeOrder()
返回本地jvm运行的硬件的字节顺序.使用和硬件一致的字节顺序可能使buffer更加有效.

注意：直接用这里是硬件的字节顺序，举例：你的x86计算机的字节顺序

---ByteOrder.toString()

```

```java
//java下的大小端测试
import java.nio.ByteOrder;
import java.nio.ByteBuffer;
public class Main {
    public static void main(String[] args){
        
    System.out.println("本地jvm运行的硬件的存储字节顺序:"+ByteOrder.nativeOrder().toString());
    System.out.println("===================================");

    ByteBuffer buf =ByteBuffer.allocate(4);//创建缓冲区，执行后，再获取该对象的字节序 才是jvm的字节序
    System.out.println("JVM Default Endian: "+buf.order().toString());

    buf.putShort((short) 1);//0x0 0x1 先按照大端存储进缓冲

    buf.order(ByteOrder.LITTLE_ENDIAN);
    System.out.println("Now: "+buf.order().toString());

    buf.putShort((short) 2);//0x2 0x0 再按照小端存储进缓冲

    buf.flip();//把limit设为当前position，把position设为0，从Buffer读出数据前调用。

    for(int i=0;i<buf.limit();i++)
        System.out.println(buf.get()&0xFF);
		
    }

}

/*
本地jvm运行的硬件的存储字节顺序:LITTLE_ENDIAN
===================================
JVM Default Endian: BIG_ENDIAN
Now: LITTLE_ENDIAN
0
1
2
0
*/
```

部分源码
```java
//nativeOrder
 public static ByteOrder nativeOrder() {
        return Bits.byteOrder();
    }

```

```java
//byteOrder
static ByteOrder byteOrder() {
        if (byteOrder == null)
            throw new Error("Unknown byte order");
        return byteOrder;
    }

    static {
        long a = unsafe.allocateMemory(8);
        try {
            unsafe.putLong(a, 0x0102030405060708L);
            byte b = unsafe.getByte(a);
            switch (b) {
            case 0x01: byteOrder = ByteOrder.BIG_ENDIAN;     break;
            case 0x08: byteOrder = ByteOrder.LITTLE_ENDIAN;  break;
            default:
                assert false;
                byteOrder = null;
            }
        } finally {
            unsafe.freeMemory(a);
        }
    }

```


java中大小端转换

```java
//数组中b[3]  为高位内存地址

//将整数按照小端存放，低字节出访低位
public static byte[] toLH(int n) {
    byte[] b = new byte[4];
    b[0] = (byte) (n & 0xff);
    b[1] = (byte) (n >> 8 & 0xff);
    b[2] = (byte) (n >> 16 & 0xff);
    b[3] = (byte) (n >> 24 & 0xff);
    return b;
}

//将int转为大端，低字节存储高位
public static byte[] toHH(int n) {
    byte[] b = new byte[4];
    b[3] = (byte) (n & 0xff);
    b[2] = (byte) (n >> 8 & 0xff);
    b[1] = (byte) (n >> 16 & 0xff);
    b[0] = (byte) (n >> 24 & 0xff);
    return b;
}

```
附一个C++测试
```c++
//判断当前机器的大小端存储
#include <iostream>
using namespace std;

int main(){
    int i=0x12345678;
    char *c = (char*)&i;
    if(*c==0x78){//低位地址*c存放低位字节0x78
        cout<<"little_end"<<endl;
    }else{
        cout<<"big_end"<<endl;
    }
    return 0;
}

/*
little_end
*/
```

说了很多，什么时候使用大小端呢

**当你的数据离开主机，经过网络传输，别人需要接收解析时需要**



| 主机字节顺序（HBO，Host Byte Order） |
| ------------- |
| 不同的机器HBO不相同，与CPU设计有关，数据的顺序是由cpu决定的,与操作系统无关。 |

**Intelx86结构为小端，其余架构CPU基本为大端。**

| 网络字节顺序NBO（Network Byte Order）  |
| ------------- |
| 按从高到低的顺序存储，在网络上使用统一的网络字节顺序，可以避免兼容性问题。 |



Socket中的字节序转换函数

| Direction   | Socket FUNC        | Description          | 
| ------------- | ------------- |:-------------:| 
| NBO ⇀ HBO     | ntohl()     | "Network to Host Long" | 
|               | ntohs()      | "Network to Host Short"|  
| HBO ⇀ NBO     | htonl() | "Host to Network Long"      |   
|               | htons() | "Host to Network Short"      |   



