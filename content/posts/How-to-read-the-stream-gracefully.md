---
title: 如何优雅的读取流
date: 2016-06-22 23:50:00
tag: java
---


Java中关于InputStream读取很多人立马就想到下面的写法:

```bash
        InputStream inputStream = null;
        InputStreamReader reader = null;
        BufferedReader bufferedReader = null;
        try {
            inputStream = new FileInputStream("/home/0x1024/1024.txt");
            reader = new InputStreamReader(inputStream);
            bufferedReader = new BufferedReader(reader);
            String temp;
            StringBuilder sb = new StringBuilder();
            while (null != (temp = bufferedReader.readLine())) {
                sb.append(temp);
            }
            System.out.print(sb.toString());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (null != inputStream){
                inputStream.close();
            }
            if (null != reader){
                reader.close();
            }
            if (null != bufferedReader){
                bufferedReader.close();
            }
        }
```

下面这几个类都实现了Closeable,所以读取完毕后都需要关闭。

```bash
        InputStream inputStream
        InputStreamReader reader;
        BufferedReader bufferedReader;
```

最终读取成功失败与否，还需要将这个流关闭掉,然后又是一陀臃肿的代码

```bash
            if (null != inputStream){
                inputStream.close();
            }
            if (null != reader){
                reader.close();
            }
            if (null != bufferedReader){
                bufferedReader.close();
            }
```

很多朋友估计每天都在承受这样的痛苦，不过自从JDK7发布以来，Closeable接口扩展了一个叫AutoCloseable的接口.

```bash
public interface Closeable extends AutoCloseable {    
        void close() throws IOException;
}
```
那么上面的这个文件读取可以很C#地写出:

```bash
     try (InputStream inputStream = new FileInputStream("/home/0x1024/1024.txt");
             InputStreamReader reader = new InputStreamReader(inputStream);
             BufferedReader bufferedReader = new BufferedReader(reader)) {
            String temp;
            StringBuilder sb = new StringBuilder();
            while (null != (temp = bufferedReader.readLine())) {
                sb.append(temp);
            }
            System.out.print(sb.toString());
        }catch (IOException e){
            e.printStackTrace();
        }
```

这样就完了吗???现在都已经JDK8了, 好的常用类都扩展了Steam,比如BufferedReader,有一个public Stream<String> lines()方法,那么上面的的代码就可以更优雅了.

```bash
    try (InputStream inputStream = new FileInputStream("/home/0x1024/1024.txt");
             InputStreamReader reader = new InputStreamReader(inputStream);
             BufferedReader bufferedReader = new BufferedReader(reader)) {
            System.out.println(bufferedReader.lines().collect(Collectors.joining()));
        }catch (IOException e){
            e.printStackTrace();
        }
```
是的，你没看错。。。就那么点代码了！！！

只要你的jdk是7以上，只要实现了Closeable就可以用 try(...){...} 的方式撸代码了.


### 后话

更偷懒的朋友可能会这样写:

```bash
        try (BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new FileInputStream("/home/0x1024/1024.txt")))) {
            System.out.println(bufferedReader.lines().collect(Collectors.joining()));
        }catch (IOException e){
            e.printStackTrace();
        }
```
那么我告诉你，这样写是不正确的，详情请阅读:《JAVA程序员修炼之道》
