---
title: "Socket Programming trong Java: Lập Trình Mạng Cơ Bản"
date: 2024-12-27T15:45:00+07:00
description: "Học cách xây dựng ứng dụng client-server với Socket trong Java - Nền tảng cho lập trình mạng"
tags: ["java", "socket", "network", "client-server", "tcp"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **Socket Programming trong Java**! Đây là kiến thức nền tảng cho lập trình mạng và phát triển ứng dụng client-server.

## Socket Là Gì?

### 🔌 Khái Niệm Socket
Socket là điểm cuối (endpoint) của kết nối mạng hai chiều:
- Cho phép hai máy tính giao tiếp với nhau qua mạng
- Hoạt động theo mô hình Client-Server
- Sử dụng giao thức TCP/IP hoặc UDP
- Cơ sở cho hầu hết các ứng dụng mạng hiện đại

### 🎯 Tại Sao Học Socket?
Hiểu Socket giúp bạn:
- Xây dựng ứng dụng chat, game online
- Phát triển API và web services
- Hiểu cách Internet hoạt động
- Nền tảng cho học Network Programming nâng cao

### 💡 TCP vs UDP
**TCP (Transmission Control Protocol)**
- Đảm bảo dữ liệu đến đúng thứ tự
- Có cơ chế kiểm tra lỗi
- Phù hợp: Chat, file transfer, web browsing

**UDP (User Datagram Protocol)**
- Nhanh hơn nhưng không đảm bảo
- Không kiểm tra lỗi
- Phù hợp: Video streaming, gaming, VoIP

## Cài Đặt và Chuẩn Bị

Java có sẵn package `java.net` cho Socket Programming:
```java
import java.net.Socket;
import java.net.ServerSocket;
import java.io.*;
```

Không cần cài thêm thư viện ngoài!

## Socket Server - Máy Chủ

### Server Cơ Bản
```java
import java.io.*;
import java.net.*;

public class SimpleServer {
    public static void main(String[] args) {
        int port = 8888;
        
        try {
            // Tạo ServerSocket lắng nghe ở cổng 8888
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("Server đang chạy trên port " + port);
            System.out.println("Đang chờ client kết nối...");
            
            // Chấp nhận kết nối từ client (blocking call)
            Socket clientSocket = serverSocket.accept();
            System.out.println("Client đã kết nối: " + clientSocket.getInetAddress());
            
            // Tạo luồng input để nhận dữ liệu
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );
            
            // Tạo luồng output để gửi dữ liệu
            PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true
            );
            
            // Nhận message từ client
            String clientMessage = in.readLine();
            System.out.println("Client gửi: " + clientMessage);
            
            // Gửi response về client
            out.println("Server đã nhận: " + clientMessage);
            
            // Đóng kết nối
            clientSocket.close();
            serverSocket.close();
            
        } catch (IOException e) {
            System.out.println("Lỗi: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### Giải Thích Code Server

**ServerSocket serverSocket = new ServerSocket(port)**
- Tạo server socket lắng nghe trên port chỉ định
- Port phải từ 1024-65535 (0-1023 là system ports)

**Socket clientSocket = serverSocket.accept()**
- Chờ client kết nối (blocking method)
- Trả về Socket object khi có client connect

**BufferedReader và PrintWriter**
- `BufferedReader`: Đọc dữ liệu text từ client
- `PrintWriter`: Gửi dữ liệu text cho client
- `InputStreamReader`: Chuyển byte stream thành character stream

## Socket Client - Máy Khách

### Client Cơ Bản
```java
import java.io.*;
import java.net.*;

public class SimpleClient {
    public static void main(String[] args) {
        String serverAddress = "localhost"; // hoặc "127.0.0.1"
        int port = 8888;
        
        try {
            // Kết nối đến server
            Socket socket = new Socket(serverAddress, port);
            System.out.println("Đã kết nối đến server!");
            
            // Tạo luồng output để gửi dữ liệu
            PrintWriter out = new PrintWriter(
                socket.getOutputStream(), true
            );
            
            // Tạo luồng input để nhận dữ liệu
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            
            // Gửi message đến server
            String message = "Xin chào Server!";
            out.println(message);
            System.out.println("Đã gửi: " + message);
            
            // Nhận response từ server
            String serverResponse = in.readLine();
            System.out.println("Server phản hồi: " + serverResponse);
            
            // Đóng kết nối
            socket.close();
            
        } catch (UnknownHostException e) {
            System.out.println("Không tìm thấy server: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("Lỗi I/O: " + e.getMessage());
        }
    }
}
```

## Multi-Client Server

Server thực tế cần xử lý nhiều client cùng lúc:
```java
import java.io.*;
import java.net.*;

public class MultiClientServer {
    public static void main(String[] args) {
        int port = 8888;
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Multi-Client Server đang chạy trên port " + port);
            
            // Vòng lặp vô tận để chấp nhận nhiều client
            while (true) {
                // Chờ client kết nối
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client mới kết nối: " + 
                    clientSocket.getInetAddress());
                
                // Tạo thread mới cho mỗi client
                ClientHandler handler = new ClientHandler(clientSocket);
                Thread thread = new Thread(handler);
                thread.start();
            }
            
        } catch (IOException e) {
            System.out.println("Lỗi server: " + e.getMessage());
        }
    }
}

// Class xử lý mỗi client
class ClientHandler implements Runnable {
    private Socket clientSocket;
    
    public ClientHandler(Socket socket) {
        this.clientSocket = socket;
    }
    
    @Override
    public void run() {
        try {
            // Tạo input/output streams
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true
            );
            
            // Gửi welcome message
            out.println("Chào mừng đến với server!");
            
            // Xử lý messages từ client
            String message;
            while ((message = in.readLine()) != null) {
                System.out.println("Client gửi: " + message);
                
                // Echo message về client
                out.println("Echo: " + message);
                
                // Thoát nếu client gửi "bye"
                if (message.equalsIgnoreCase("bye")) {
                    System.out.println("Client ngắt kết nối");
                    break;
                }
            }
            
            // Đóng kết nối
            clientSocket.close();
            
        } catch (IOException e) {
            System.out.println("Lỗi xử lý client: " + e.getMessage());
        }
    }
}
```

## Ứng Dụng Chat Đơn Giản

### Chat Client với Input từ User
```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class ChatClient {
    public static void main(String[] args) {
        String serverAddress = "localhost";
        int port = 8888;
        
        try {
            Socket socket = new Socket(serverAddress, port);
            System.out.println("=== CHAT CLIENT ===");
            System.out.println("Đã kết nối đến server!");
            
            // Input/Output streams
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                socket.getOutputStream(), true
            );
            
            // Thread để nhận message từ server
            Thread receiver = new Thread(() -> {
                try {
                    String serverMessage;
                    while ((serverMessage = in.readLine()) != null) {
                        System.out.println("\n[Server]: " + serverMessage);
                        System.out.print("Bạn: ");
                    }
                } catch (IOException e) {
                    System.out.println("Mất kết nối với server");
                }
            });
            receiver.start();
            
            // Gửi message đến server
            Scanner scanner = new Scanner(System.in);
            System.out.println("Nhập 'quit' để thoát\n");
            
            while (true) {
                System.out.print("Bạn: ");
                String message = scanner.nextLine();
                
                if (message.equalsIgnoreCase("quit")) {
                    break;
                }
                
                out.println(message);
            }
            
            socket.close();
            scanner.close();
            
        } catch (IOException e) {
            System.out.println("Lỗi: " + e.getMessage());
        }
    }
}
```

## File Transfer với Socket

### Server Nhận File
```java
import java.io.*;
import java.net.*;

public class FileServer {
    public static void main(String[] args) {
        int port = 8888;
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("File Server đang chạy...");
            
            Socket clientSocket = serverSocket.accept();
            System.out.println("Client kết nối!");
            
            // Nhận tên file
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );
            String fileName = in.readLine();
            System.out.println("Nhận file: " + fileName);
            
            // Nhận nội dung file
            InputStream inputStream = clientSocket.getInputStream();
            FileOutputStream fileOutput = new FileOutputStream("received_" + fileName);
            
            byte[] buffer = new byte[4096];
            int bytesRead;
            
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                fileOutput.write(buffer, 0, bytesRead);
            }
            
            fileOutput.close();
            clientSocket.close();
            
            System.out.println("File đã được lưu!");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Client Gửi File
```java
import java.io.*;
import java.net.*;

public class FileClient {
    public static void main(String[] args) {
        String serverAddress = "localhost";
        int port = 8888;
        String filePath = "test.txt"; // File cần gửi
        
        try {
            Socket socket = new Socket(serverAddress, port);
            
            File file = new File(filePath);
            
            // Gửi tên file
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            out.println(file.getName());
            
            // Gửi nội dung file
            FileInputStream fileInput = new FileInputStream(file);
            OutputStream outputStream = socket.getOutputStream();
            
            byte[] buffer = new byte[4096];
            int bytesRead;
            
            System.out.println("Đang gửi file...");
            while ((bytesRead = fileInput.read(buffer)) != -1) {
                outputStream.write(buffer, 0, bytesRead);
            }
            
            fileInput.close();
            socket.close();
            
            System.out.println("Gửi file thành công!");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Xử Lý Lỗi và Best Practices

### Try-with-Resources
```java
// Tự động đóng resources
try (ServerSocket serverSocket = new ServerSocket(port);
     Socket socket = serverSocket.accept();
     BufferedReader in = new BufferedReader(
         new InputStreamReader(socket.getInputStream()));
     PrintWriter out = new PrintWriter(
         socket.getOutputStream(), true)) {
    
    // Code xử lý
    
} catch (IOException e) {
    e.printStackTrace();
}
```

### Timeout cho Socket
```java
Socket socket = new Socket();
socket.connect(new InetSocketAddress(host, port), 5000); // 5s timeout
socket.setSoTimeout(10000); // 10s timeout cho read operations
```

### Kiểm Tra Kết Nối
```java
if (socket.isConnected() && !socket.isClosed()) {
    // Socket còn hoạt động
}
```



## Troubleshooting

**Port already in use**
```java
// Thêm SO_REUSEADDR
ServerSocket serverSocket = new ServerSocket();
serverSocket.setReuseAddress(true);
serverSocket.bind(new InetSocketAddress(port));
```

**Connection refused**
- Kiểm tra server đã chạy chưa
- Kiểm tra port number
- Kiểm tra firewall

**SocketTimeoutException**
- Tăng timeout value
- Kiểm tra network connection

## Tips Học Socket Programming

1. **Hiểu TCP/IP** - Đọc về OSI model và TCP/IP stack
2. **Test Local First** - Dùng localhost trước khi test qua mạng
3. **Use Wireshark** - Quan sát packets để debug
4. **Handle Exceptions** - Socket operations dễ bị lỗi
5. **Thread Safety** - Cẩn thận khi dùng multi-threading

## Kết Luận

Trong bài viết này, chúng ta đã học:
- Khái niệm Socket và Client-Server model
- Cách tạo Socket Server và Client cơ bản
- Multi-client server với threading
- Ứng dụng chat và file transfer
- Best practices và error handling


## Tài Liệu Tham Khảo

- Oracle Java Socket Documentation
- TCP/IP Illustrated (Stevens)
- Java Network Programming (O'Reilly)
- RFC 793 (TCP Specification)

---