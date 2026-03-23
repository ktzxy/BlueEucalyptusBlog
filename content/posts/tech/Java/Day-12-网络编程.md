+++
date = '2024-03-21T16:53:43+08:00'
draft = false
title = 'Day 12 网络编程'
slug = "sdd7voc7m9"
description = "Day 12 网络编程"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 12 网络编程"
comments = true  
+++
﻿# Day-11-网络编程

### 引入

【1】网络编程:
把分布在不同地理区域的计算机与专门的外部设备用通信线路互连成一个规模大、功能强的网络系统，从而使众多的计算机可以方便地互相传递信息、共享硬件、软件、数据信息等资源。

设备之间在网络中进行数据的传输，发送/接收数据。

![image-20210223165944493](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637406_37f07e.webp)

【2】通信的两个重要要素：IP+PORT

![image-20210223170238557](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637141_e8d12e.webp)

【3】设备之间进行传输的时候，必须遵照一定的规则--》通信协议

![image-20210223170531905](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637879_8b8526.webp)



### lnetAddress 、lnetSocketAddress 

【1】 lnetAddress ---》封装了IP

```java
package com.zy.test1;

import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Test1 {
    public static void main(String[] args) throws UnknownHostException {
        //封装ip
        //InetAddress is = new InetAddress(); 不能直接创建对象，因为工netAddress()被default修饰了。
        InetAddress ia2 = InetAddress.getByName("localhost");
        System.out.println(ia2);
        InetAddress ia3 = InetAddress.getByName("www.baidu.com"); //封装域名
        System.out.println(ia3);
        System.out.println(ia3.getHostName()); //获取域名
        System.out.println(ia3.getHostAddress()); //获取ip地址
    }
}
```

【2】 lnetSocketAddress ---》封装了IP，端口号

```java
package com.zy.test1;

import java.net.InetAddress;
import java.net.InetSocketAddress;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Test2 {
    public static void main(String[] args) {
        InetSocketAddress isa = new InetSocketAddress("localhost", 8080);
        System.out.println(isa);
        System.out.println(isa.getHostName());
        System.out.println(isa.getPort());

        InetAddress ia = isa.getAddress();
        System.out.println(ia.getHostName());
        System.out.println(ia.getHostAddress());
    }
}
```

### 套接字

![image-20210223172415111](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637032_9d7ac8.webp)

### 基于TCP的网络编程

功能:模拟网站的登录，客户端录入账号密码，然后服务器端进行验证。

**单向通信：**

客户端：

```java
package com.zy.test2;

import java.io.DataOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestClient {
    public static void main(String[] args) throws IOException {
        //1。创建套接字:指定服务器的ip和端口号:
        Socket s = new Socket("localhost", 8585);
        //向外发送数据 利用传输流
        OutputStream os = s.getOutputStream();
        DataOutputStream dos = new DataOutputStream(os);
        //利用这个outputStream就可以向外发送数据了，但是没有直接发送string的方法
        //所以我们又在outputStream外面套了一个处理流: DataoutputStream
        dos.writeUTF("你好");

        //3.关闭流 + 关闭网络资源
        dos.close();
        os.close();
        s.close();
    }
}
```

服务端：

```java
package com.zy.test2;

import java.io.DataInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestServer {
    public static void main(String[] args) throws IOException {
        //1.创建套接字：指定服务器的端口号
        ServerSocket ss = new ServerSocket(8585);
        //2.等着客户端发来的消息
        Socket s = ss.accept();//阻塞方法:等待接收客户端的数据，什么时候接收到数据，什么时候程序继续向下执行。
        //accept()返回值为一个Socket，这个Socket其实就是客户端的Socket
        //接到这个Socket以后，客户端和服务器才真正产生了连接，才真正可以通信了
        //3.感受到的操作流：
        InputStream is = s.getInputStream();
        DataInputStream dis = new DataInputStream(is);
        
        //4.读取客户端发来的数据：
        String str = dis.readUTF();
        System.out.println("客户端发来的数据为"+str);

        //5.关闭流+关闭网络资源：
        dis.close();
        is.close();
        s.close();
        ss.close();

    }
}
```

测试: 先开服务器，再开启客户端

**双向通信：**

服务端：

```java
package com.zy.test2;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestServer {
    public static void main(String[] args) throws IOException {
        //1.创建套接字：指定服务器的端口号
        ServerSocket ss = new ServerSocket(8585);
        //2.等着客户端发来的消息
        Socket s = ss.accept();//阻塞方法:等待接收客户端的数据，什么时候接收到数据，什么时候程序继续向下执行。
        //accept()返回值为一个Socket，这个Socket其实就是客户端的Socket
        //接到这个Socket以后，客户端和服务器才真正产生了连接，才真正可以通信了
        //3.感受到的操作流：
        InputStream is = s.getInputStream();
        DataInputStream dis = new DataInputStream(is);
        
        //4.读取客户端发来的数据：
        String str = dis.readUTF();
        System.out.println("客户端发来的数据为"+str);

        //向客户端输出一句话 操作流---》输出流
        OutputStream os = s.getOutputStream();
        DataOutputStream dos = new DataOutputStream(os);
        dos.writeUTF("你好，我是服务器端，我就收到你的消息了");

        //5.关闭流+关闭网络资源：
        dos.close();
        os.close();
        dis.close();
        is.close();
        s.close();
        ss.close();

    }
}
```

客户端：

```java
package com.zy.test2;

import java.io.*;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestClient {
    public static void main(String[] args) throws IOException {
        //1。创建套接字:指定服务器的ip和端口号:
        Socket s = new Socket("localhost", 8585);
        //向外发送数据 利用传输流
        OutputStream os = s.getOutputStream();
        DataOutputStream dos = new DataOutputStream(os);
        //利用这个outputStream就可以向外发送数据了，但是没有直接发送string的方法
        //所以我们又在outputStream外面套了一个处理流: DataoutputStream
        dos.writeUTF("你好");

        //接收服务器端的回话--》利用输入流:
        InputStream is = s.getInputStream();
        DataInputStream dis = new DataInputStream(is);
        String str = dis.readUTF();
        System.out.println("服务器端对我说"+str);


        //3.关闭流 + 关闭网络资源
        dis.close();
        is.close();
        dos.close();
        os.close();
        s.close();
    }
}
```

综合：

封装的user类：

```java
package com.zy.test3;

import java.io.Serializable;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test3
 * @version: 1.0
 */
public class User implements Serializable {
    private static final long serialVersionUID = 949071293973714766L;
    private String name;
    private String pwd;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public User() {
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    public User(String name, String pwd) {
        this.name = name;
        this.pwd = pwd;
    }
}
```

服务端：

```java
package com.zy.test3;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestServer {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //1.创建套接字：指定服务器的端口号
        ServerSocket ss = new ServerSocket(8585);
        //2.等着客户端发来的消息
        Socket s = ss.accept();//阻塞方法:等待接收客户端的数据，什么时候接收到数据，什么时候程序继续向下执行。
        //accept()返回值为一个Socket，这个Socket其实就是客户端的Socket
        //接到这个Socket以后，客户端和服务器才真正产生了连接，才真正可以通信了
        //3.感受到的操作流：
        InputStream is = s.getInputStream();
        ObjectInputStream ois = new ObjectInputStream(is);

        //4.读取客户端发来的数据：
        User user = (User)(ois.readObject());

        //对对象进行验证：
        boolean flag = false;
        if (user.getName().equals("admin")&&user.getPwd().equals("123456")){
            flag = true;
        }


        //向客户端输出一句话 操作流---》输出流
        OutputStream os = s.getOutputStream();
        DataOutputStream dos = new DataOutputStream(os);
        dos.writeBoolean(flag);

        //5.关闭流+关闭网络资源：
        dos.close();
        os.close();
        ois.close();
        is.close();
        s.close();
        ss.close();

    }
}
```

客户端：

```java
package com.zy.test3;

import java.io.*;
import java.net.Socket;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestClient {
    public static void main(String[] args) throws IOException {
        //1。创建套接字:指定服务器的ip和端口号:
        Socket s = new Socket("localhost", 8585);

        //录入用户的账号和密码：
        Scanner sc = new Scanner(System.in);
        System.out.println("请录入你的账号：");
        String name = sc.next();
        System.out.println("请录入你的密码：");
        String pwd = sc.next();
        //将账号和密码封装为一个User的对象：
        User user = new User(name, pwd);



        //向外发送数据 利用传输流
        OutputStream os = s.getOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(os);
        oos.writeObject(user);

        //接收服务器端的回话--》利用输入流:
        InputStream is = s.getInputStream();
        DataInputStream dis = new DataInputStream(is);
        boolean b = dis.readBoolean();
        if (b){
            System.out.println("恭喜，登录成功");
        }else {
            System.out.println("对不起，登录失败");
        }


        //3.关闭流 + 关闭网络资源
        dis.close();
        is.close();
        oos.close();
        os.close();
        s.close();
    }
}
```

异常处理：

```java
package com.zy.test3;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestServer {
    public static void main(String[] args) {
        //1.创建套接字：指定服务器的端口号
        ServerSocket ss = null;
        DataOutputStream dos = null;
        OutputStream os = null;
        Socket s = null;
        InputStream is = null;
        ObjectInputStream ois = null;
        try {
            ss = new ServerSocket(8585);
            //2.等着客户端发来的消息
             s = ss.accept();//阻塞方法:等待接收客户端的数据，什么时候接收到数据，什么时候程序继续向下执行。
            //accept()返回值为一个Socket，这个Socket其实就是客户端的Socket
            //接到这个Socket以后，客户端和服务器才真正产生了连接，才真正可以通信了
            //3.感受到的操作流：
             is = s.getInputStream();
             ois = new ObjectInputStream(is);

            //4.读取客户端发来的数据：
            User user = null;
            try {
                user = (User)(ois.readObject());
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }

            //对对象进行验证：
            boolean flag = false;
            if (user.getName().equals("admin")&&user.getPwd().equals("123456")){
                flag = true;
            }


            //向客户端输出一句话 操作流---》输出流
             os = s.getOutputStream();
             dos= new DataOutputStream(os);
            dos.writeBoolean(flag);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //5.关闭流+关闭网络资源：
            try {
                if (dos!=null){
                    dos.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (os!=null){
                    os.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (ois!=null){
                    ois.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (is!=null){
                    is.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (s!=null){
                    s.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (ss!=null){
                    ss.close();
                }

            } catch (IOException e) {
                e.printStackTrace();
            }
        }




    }
}
```

```java
package com.zy.test3;

import java.io.*;
import java.net.Socket;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test2
 * @version: 1.0
 */
public class TestClient {
    public static void main(String[] args) {
        //1。创建套接字:指定服务器的ip和端口号:
        Socket s = null;
        OutputStream os =null;
        DataInputStream dis = null;
        InputStream is =null;
        ObjectOutputStream oos =null;
        try {
            s = new Socket("localhost", 8585);

            //录入用户的账号和密码：
            Scanner sc = new Scanner(System.in);
            System.out.println("请录入你的账号：");
            String name = sc.next();
            System.out.println("请录入你的密码：");
            String pwd = sc.next();
            //将账号和密码封装为一个User的对象：
            User user = new User(name, pwd);



            //向外发送数据 利用传输流
             os = s.getOutputStream();
             oos = new ObjectOutputStream(os);
            oos.writeObject(user);

            //接收服务器端的回话--》利用输入流:
             is = s.getInputStream();
             dis = new DataInputStream(is);
            boolean b = dis.readBoolean();
            if (b){
                System.out.println("恭喜，登录成功");
            }else {
                System.out.println("对不起，登录失败");
            }

        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //3.关闭流 + 关闭网络资源
            try {
                if (dis!=null){
                    dis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (is!=null){
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (oos!=null){
                    oos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (os!=null){
                    os.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (s!=null){
                    s.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }




    }
}
```

### 基于UDP的网络编程

TCP:
客户端: Socket           程序感受到的使用流∶输出流
服务器端:ServerSocket --->Socket程序感受到的使用流︰输入流

(客户端和服务器端地位不平等。)

UDP:
发送方:DatagramSocket 发送:数据包   DatagramPacket

接收方:DatagramSocket接收:数据包 DatagramPacket

(发送方和接收方的地址是平等的。)

**UDP案例：完成网站的咨询聊天**

**单向通信：**

发送方：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestSend {
    public static void main(String[] args) throws IOException {
        System.out.println("学生上线了");
        //1.准备套接字：指定发送方的端口号
        DatagramSocket ds = new DatagramSocket(8585);
        //2.准备数据包
        String str = "你好";
        byte[] bytes = str.getBytes();
        /*
        *需要四个参数:
        1.指的是传送数据转为字节数组
        2.字节数组的长度
        3.封装接收方的ip
        4.指定接收方的端口号
        * */
        DatagramPacket dp = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8888);
        //发送
        ds.send(dp);
        //关闭
        ds.close();
    }
}
```

接收方：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestReceive {
    public static void main(String[] args) throws IOException {
        System.out.println("老师上线了");
        //1.创建套接字：指定接收方的端口
        DatagramSocket ds = new DatagramSocket(8888);
        //2.有一个空的数据包，打算用来接受  对方传过来的数据包：
        byte[] b = new byte[1024];
        DatagramPacket dp = new DatagramPacket(b, b.length);
        //3.接受对方的数据包，然后放入我们的dp数据包中填充
        ds.receive(dp);
        //4.取出数据
        byte[] data = dp.getData();
        String s = new String(data,0,dp.getLength()); //数据包中的有效长度
        System.out.println("学生对我说："+s);
        //5.关闭
        ds.close();
    }
}
```

**双向通信：**

发送方：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestSend {
    public static void main(String[] args) throws IOException {
        System.out.println("学生上线了");
        //1.准备套接字：指定发送方的端口号
        DatagramSocket ds = new DatagramSocket(8585);
        //2.准备数据包
        Scanner sc = new Scanner(System.in);
        System.out.print("学生：");
        String str = sc.next();
        byte[] bytes = str.getBytes();
        /*
        *需要四个参数:
        1.指的是传送数据转为字节数组
        2.字节数组的长度
        3.封装接收方的ip
        4.指定接收方的端口号
        * */
        DatagramPacket dp = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8888);
        //发送
        ds.send(dp);

        //接受老师发送来的信息
        byte[] b = new byte[1024];
        DatagramPacket dp2 = new DatagramPacket(b, b.length);

        ds.receive(dp2);

        byte[] data = dp2.getData();
        String s = new String(data,0,dp2.getLength()); //数据包中的有效长度
        System.out.println("老师对我说："+s);




        //关闭
        ds.close();
    }
}
```

接收方：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestReceive {
    public static void main(String[] args) throws IOException {
        System.out.println("老师上线了");
        //1.创建套接字：指定接收方的端口
        DatagramSocket ds = new DatagramSocket(8888);
        //2.有一个空的数据包，打算用来接受  对方传过来的数据包：
        byte[] b = new byte[1024];
        DatagramPacket dp = new DatagramPacket(b, b.length);
        //3.接受对方的数据包，然后放入我们的dp数据包中填充
        ds.receive(dp);
        //4.取出数据
        byte[] data = dp.getData();
        String s = new String(data,0,dp.getLength()); //数据包中的有效长度
        System.out.println("学生对我说："+s);

        //老师进行回复
        Scanner sc = new Scanner(System.in);
        System.out.println("老师回复：");
        String str =sc.next();
        byte[] bytes = str.getBytes();
        DatagramPacket dp2 = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8585);
        ds.send(dp2);

        //5.关闭
        ds.close();
    }
}
```

异常处理：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestSend {
    public static void main(String[] args) {
        System.out.println("学生上线了");
        //1.准备套接字：指定发送方的端口号
        DatagramSocket ds = null;
        try {
            ds = new DatagramSocket(8585);
            //2.准备数据包
            Scanner sc = new Scanner(System.in);
            System.out.print("学生：");
            String str = sc.next();
            byte[] bytes = str.getBytes();
            DatagramPacket dp = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8888);
            //发送
            ds.send(dp);

            //接受老师发送来的信息
            byte[] b = new byte[1024];
            DatagramPacket dp2 = new DatagramPacket(b, b.length);

            ds.receive(dp2);

            byte[] data = dp2.getData();
            String s = new String(data,0,dp2.getLength()); //数据包中的有效长度
            System.out.println("老师对我说："+s);
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //关闭
            ds.close();
        }
    }
}
```

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestReceive {
    public static void main(String[] args) {
        System.out.println("老师上线了");
        //1.创建套接字：指定接收方的端口
        DatagramSocket ds = null;
        try {
            ds = new DatagramSocket(8888);
            //2.有一个空的数据包，打算用来接受  对方传过来的数据包：
            byte[] b = new byte[1024];
            DatagramPacket dp = new DatagramPacket(b, b.length);
            //3.接受对方的数据包，然后放入我们的dp数据包中填充
            ds.receive(dp);
            //4.取出数据
            byte[] data = dp.getData();
            String s = new String(data,0,dp.getLength()); //数据包中的有效长度
            System.out.println("学生对我说："+s);

            //老师进行回复
            Scanner sc = new Scanner(System.in);
            System.out.println("老师回复：");
            String str =sc.next();
            byte[] bytes = str.getBytes();
            DatagramPacket dp2 = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8585);
            ds.send(dp2);
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //5.关闭
            ds.close();
        }
    }
}
```

完整通信：

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestSend {
    public static void main(String[] args) {
        System.out.println("学生上线了");
        //1.准备套接字：指定发送方的端口号
        DatagramSocket ds = null;
        try {
            ds = new DatagramSocket(8585);
            while (true){
                //2.准备数据包
                Scanner sc = new Scanner(System.in);
                System.out.print("学生：");
                String str = sc.next();
                byte[] bytes = str.getBytes();

                DatagramPacket dp = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8888);
                //发送
                ds.send(dp);

                if (str.equals("byebye")){
                    System.out.println("学生下线了");
                    break;
                }

                //接受老师发送来的信息
                byte[] b = new byte[1024];
                DatagramPacket dp2 = new DatagramPacket(b, b.length);

                ds.receive(dp2);

                byte[] data = dp2.getData();
                String s = new String(data,0,dp2.getLength()); //数据包中的有效长度
                System.out.println("老师对我说："+s);
            }
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //关闭
            ds.close();
        }
    }
}
```

```java
package com.zy.test4;

import java.io.IOException;
import java.net.*;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test4
 * @version: 1.0
 */
public class TestReceive {
    public static void main(String[] args) {
        System.out.println("老师上线了");
        //1.创建套接字：指定接收方的端口
        DatagramSocket ds = null;
        try {
            ds = new DatagramSocket(8888);
            while (true){
                //2.有一个空的数据包，打算用来接受  对方传过来的数据包：
                byte[] b = new byte[1024];
                DatagramPacket dp = new DatagramPacket(b, b.length);
                //3.接受对方的数据包，然后放入我们的dp数据包中填充
                ds.receive(dp);
                //4.取出数据
                byte[] data = dp.getData();
                String s = new String(data,0,dp.getLength()); //数据包中的有效长度
                System.out.println("学生对我说："+s);
                if (s.equals("byebye")){
                    System.out.println("学生已经下线了，老师也下线了");
                    break;
                }

                //老师进行回复
                Scanner sc = new Scanner(System.in);
                System.out.println("老师回复：");
                String str =sc.next();
                byte[] bytes = str.getBytes();
                DatagramPacket dp2 = new DatagramPacket(bytes, bytes.length, InetAddress.getByName("localhost"), 8585);
                ds.send(dp2);
            }
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //5.关闭
            ds.close();
        }
    }
}
```

# 
