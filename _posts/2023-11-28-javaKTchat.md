---
layout: single
title: "[Java基于Java和Kotlin的多人聊天"
categories: [Java]
last_modified_at: 2023-11-28
excerpt: "学习和理解Java和Kotlin在网络编程中的应用，特别是Socket编程，以及如何在局域网下实现多人通讯"
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---

> 一个简单的聊天室，包括一个服务器端和一个客户端。服务器端使用Kotlin编写，客户端使用Java编写。

## 相关知识点：  

Socket编程：Java和Kotlin都提供了Socket API，用于实现网络通信。服务器端使用ServerSocket监听新的连接，客户端使用Socket与服务器建立连接。  
多线程：服务器端为每个客户端创建一个新的线程来处理客户端的消息。客户端也在一个新的线程中监听服务器发送的消息。  
IO流：Java和Kotlin都提供了IO流API，用于读写数据。我们使用BufferedReader和PrintWriter来读写Socket的输入输出流。  

## 在局域网下实现多人通讯： 

在局域网下实现多人通讯，我们需要将服务器端的IP地址改为局域网的IP地址。在Windows系统下，我们可以使用ipconfig命令来获取局域网的IPV4地址。然后，将客户端的代码中，与服务器建立连接的IP地址改为服务器端的局域网IP地址。

```java
// 在Client.java中，将localhost改为服务器端的局域网IP地址
Socket socket = new Socket("服务器端的局域网IP地址", 8080);
```

## 服务端

服务器端（Server.kt）：服务器端使用ServerSocket监听8080端口，当有新的客户端连接时，创建一个新的ClientHandler来处理客户端的消息。ClientHandler会读取客户端发送的消息，并将消息广播到所有已连接的客户端。

```kotlin
import java.io.BufferedReader
import java.io.IOException
import java.io.InputStreamReader
import java.io.PrintWriter
import java.net.ServerSocket
import java.net.Socket
import java.text.SimpleDateFormat
import java.util.*
import java.util.concurrent.CopyOnWriteArrayList

object Server {
    private val clients = CopyOnWriteArrayList<ClientHandler>()
    @JvmStatic
    fun main(args: Array<String>) {
        try {
            ServerSocket(8080).use { server ->
                while (true) {
                    val socket = server.accept()
                    val clientHandler = ClientHandler(socket)
                    clients.add(clientHandler)
                    Thread(clientHandler).start()
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    internal class ClientHandler(private val socket: Socket) : Runnable {
        private var name: String? = null
        private val reader: BufferedReader = BufferedReader(InputStreamReader(socket.getInputStream()))
        private val writer: PrintWriter = PrintWriter(socket.getOutputStream(), true)

        override fun run() {
            try {
                name = reader.readLine()
                broadcastMessage("$name 加入了聊天室")
                var message: String
                while (reader.readLine().also { message = it } != null) {
                    broadcastMessage(SimpleDateFormat("HH:mm:ss").format(Date()) + " " + name + ": " + message + " ✔✔")
                }
            } catch (e: Exception) {
                // 忽略客户端断开连接引起的异常
            } finally {
                clients.remove(this)
                broadcastMessage("$name 离开了聊天室")
                try {
                    socket.close()
                } catch (e: IOException) {
                    e.printStackTrace()
                }
            }
        }

        private fun broadcastMessage(message: String?) {
            for (client in clients) {
                client.writer.println(message)
            }
        }
    }
}
```

---

## 客户端

客户端（Client.java）：客户端首先与服务器建立连接，然后在一个新的线程中监听服务器发送的消息。同时，客户端会读取用户的输入，并将输入的消息发送到服务器。 

```java
import java.io.*;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Scanner;

public class Client {
    public static void main(String[] args) {
        try (Socket socket = new Socket("localhost", 8080);
             Scanner scanner = new Scanner(System.in)){
            OutputStream stream = socket.getOutputStream();
            OutputStreamWriter writer = new OutputStreamWriter(stream);
            BufferedReader reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));

            System.out.print("请输入您的姓名: ");
            String name = scanner.nextLine();
            writer.write(name + '\n');
            writer.flush();

            new Thread(() -> {
                while (true) {
                    try {
                        String response = reader.readLine();
                        if (response != null) {
                            System.out.println(new SimpleDateFormat("HH:mm:ss").format(new Date()) + " Server: " + response + " ✔✔");
                        } else {
                            break;
                        }
                    } catch (IOException e) {
                        break;
                    }
                }
            }).start();

            while (true) {
                System.out.print(name+"-Client: ");
                String text = scanner.nextLine();
                writer.write(text + '\n');
                writer.flush();
                System.out.println(new SimpleDateFormat("HH:mm:ss").format(new Date()) + " ✔✔");
            }
        } catch (IOException e){
            System.out.println("与服务器的连接已断开");
        }
    }
}
```

在Java和Kotlin中，我们可以通过创建Thread对象并重写其run方法来创建一个新的线程。在run方法中，我们可以定义线程的行为。然后，我们可以通过调用Thread对象的start方法来启动线程。

服务器端为每个连接的客户端创建了一个新的线程。这个线程负责读取客户端的消息，并将消息广播到所有已连接的客户端。这样，服务器就可以同时处理多个客户端的消息，而不需要等待一个客户端的消息处理完毕后再处理下一个客户端的消息。  
客户端也创建了一个新的线程来监听服务器发送的消息。这样，客户端就可以同时发送消息和接收消息，而不需要等待发送完一条消息后再接收消息。

## TODO

- 🎉首先，让我们看看Client.java和Client2.java中的代码重复部分。我们可以创建一个ClientBase类，将共享代码放在那里，然后让Client和Client2类继承它。可以避免重复代码，使代码更加整洁和易于维护。  
- 🚀在Server.kt中，我们看到ClientHandler类是内部类。这可能会导致一些问题，比如测试的困难。让我们将ClientHandler类移出来，使其成为一个顶级类。这样，就可以更容易地对其进行单元测试。  
- 🌟在Client.java和Client2.java中，我们看到了一些硬编码的值，比如服务器的IP地址和端口号。我们可以将这些值提取出来，作为常量或者配置文件中的值，这样在需要改变这些值时，就不需要在代码中到处寻找它们了。  
- 🎈在Server.kt中，我们看到了一些异常被打印出来，但没有被处理。柯创建一个异常处理策略，比如记录日志或者向用户显示错误信息。这样，就可以更好地了解系统中发生了什么，以及如何解决问题。  
- 🎯最后，看看是否可以添加一些新的功能。比如，可以添加一个用户列表，显示当前在线的用户。或者，可以添加一个私聊功能，让用户可以直接向其他用户发送消息。 

