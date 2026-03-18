---
title: "TCP Programming trong Java: Xây Dựng Kết Nối Tin Cậy"
date: 2024-12-27T17:15:00+07:00
description: "Tìm hiểu chuyên sâu TCP Socket Programming trong Java: kết nối tin cậy, xử lý stream và xây dựng ứng dụng client-server"
tags: ["java", "tcp", "socket", "network", "client-server"]
featured_image: ""
---

Chào mừng bạn đến với bài viết chuyên sâu về **lập trình TCP trong Java**. Sau khi đã tìm hiểu Socket cơ bản và Multicast, bài viết này sẽ đi sâu vào TCP – giao thức nền tảng của Internet.

## TCP Là Gì?

### 🔄 Giao thức điều khiển truyền tải (Transmission Control Protocol)

TCP là giao thức truyền tải hướng kết nối:
- **Hướng kết nối**: Phải thiết lập kết nối trước khi truyền dữ liệu  
- **Độ tin cậy cao**: Đảm bảo dữ liệu đến đúng và đủ  
- **Đúng thứ tự**: Dữ liệu được nhận theo đúng thứ tự gửi  
- **Kiểm soát lỗi**: Phát hiện và sửa lỗi tự động  
- **Kiểm soát luồng**: Điều chỉnh tốc độ truyền phù hợp  

### 🎯 TCP vs UDP

| Đặc điểm | TCP | UDP |
|--------|-----|-----|
| Kết nối | Có (bắt tay 3 bước) | Không |
| Độ tin cậy | Cao | Thấp |
| Thứ tự gói tin | Có | Không |
| Tốc độ | Chậm hơn | Nhanh hơn |
| Overhead | Cao | Thấp |
| Ứng dụng | HTTP, FTP, Email | Streaming, Game, DNS |


### 💡 Khi Nào Dùng TCP?

Sử dụng TCP khi:
- Cần đảm bảo dữ liệu đến đầy đủ
- Thứ tự dữ liệu quan trọng
- Có thể chấp nhận độ trễ cao hơn
- Ví dụ: Web browsing, file transfer, database connections

## Cơ chế bắt tay ba bước của TCP (Three-Way Handshake)

### Quá Trình Thiết Lập Kết Nối
```
Client                          Server
  |                               |
  |--- SYN (seq=x) -------------->|  1. Client gửi SYN
  |                               |
  |<-- SYN-ACK (seq=y, ack=x+1)---|  2. Server phản hồi SYN-ACK
  |                               |
  |--- ACK (seq=x+1, ack=y+1) --->|  3. Client xác nhận ACK
  |                               |
  |      Kết nối được thiết lập   |
```

### Minh Họa Trong Code
```java
import java.io.*;
import java.net.*;

public class TCPHandshakeDemo {
    public static void main(String[] args) {
        // SERVER SIDE
        new Thread(() -> {
            try {
                ServerSocket serverSocket = new ServerSocket(8888);
                System.out.println("[Server] Đang lắng nghe port 8888...");
                System.out.println("[Server] Step 1: Chờ SYN từ client");
                
                Socket clientSocket = serverSocket.accept();
                System.out.println("[Server] Step 2: Nhận SYN, gửi SYN-ACK");
                System.out.println("[Server] Step 3: Nhận ACK");
                System.out.println("[Server] ✓ Kết nối đã thiết lập!");
                
                clientSocket.close();
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
        
        // CLIENT SIDE
        try {
            Thread.sleep(1000); // Đợi server khởi động
            
            System.out.println("\n[Client] Bắt đầu kết nối...");
            System.out.println("[Client] Step 1: Gửi SYN");
            
            Socket socket = new Socket("localhost", 8888);
            
            System.out.println("[Client] Step 2: Nhận SYN-ACK, gửi ACK");
            System.out.println("[Client] ✓ Kết nối đã thiết lập!");
            
            socket.close();
            
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## TCP Socket Programming Nâng Cao

### Echo Server với Error Handling
```java
import java.io.*;
import java.net.*;

public class RobustTCPServer {
    private static final int PORT = 8888;
    private static final int BACKLOG = 50; // Số kết nối chờ tối đa
    
    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        
        try {
            // Tạo server socket với backlog
            serverSocket = new ServerSocket(PORT, BACKLOG);
            
            // Cấu hình socket options
            serverSocket.setReuseAddress(true);
            
            System.out.println("=== TCP Echo Server ===");
            System.out.println("Listening on port: " + PORT);
            System.out.println("Backlog: " + BACKLOG);
            System.out.println("Waiting for connections...\n");
            
            while (true) {
                try {
                    // Accept client connection
                    Socket clientSocket = serverSocket.accept();
                    
                    // Log connection info
                    InetAddress clientAddress = clientSocket.getInetAddress();
                    int clientPort = clientSocket.getPort();
                    System.out.println("New connection from: " + 
                        clientAddress.getHostAddress() + ":" + clientPort);
                    
                    // Handle client in new thread
                    new Thread(new ClientHandler(clientSocket)).start();
                    
                } catch (IOException e) {
                    System.err.println("Error accepting connection: " + 
                        e.getMessage());
                }
            }
            
        } catch (IOException e) {
            System.err.println("Server error: " + e.getMessage());
            e.printStackTrace();
            
        } finally {
            if (serverSocket != null && !serverSocket.isClosed()) {
                try {
                    serverSocket.close();
                    System.out.println("Server closed");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class ClientHandler implements Runnable {
    private Socket socket;
    
    public ClientHandler(Socket socket) {
        this.socket = socket;
    }
    
    @Override
    public void run() {
        BufferedReader in = null;
        PrintWriter out = null;
        
        try {
            // Set socket timeout
            socket.setSoTimeout(30000); // 30 seconds
            
            // Setup streams
            in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            out = new PrintWriter(
                socket.getOutputStream(), true
            );
            
            // Send welcome message
            out.println("Welcome to Echo Server!");
            out.println("Type 'quit' to disconnect");
            
            // Echo loop
            String line;
            while ((line = in.readLine()) != null) {
                System.out.println("[" + socket.getInetAddress() + 
                    "] Received: " + line);
                
                if (line.equalsIgnoreCase("quit")) {
                    out.println("Goodbye!");
                    break;
                }
                
                // Echo back
                out.println("Echo: " + line);
            }
            
        } catch (SocketTimeoutException e) {
            System.out.println("Client timeout: " + socket.getInetAddress());
            
        } catch (IOException e) {
            System.err.println("Client handler error: " + e.getMessage());
            
        } finally {
            // Cleanup
            try {
                if (in != null) in.close();
                if (out != null) out.close();
                if (socket != null && !socket.isClosed()) {
                    socket.close();
                }
                System.out.println("Connection closed: " + 
                    socket.getInetAddress());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### TCP Client với Reconnection
```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class RobustTCPClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 8888;
    private static final int MAX_RETRIES = 3;
    private static final int RETRY_DELAY = 5000; // 5 seconds
    
    public static void main(String[] args) {
        Socket socket = null;
        int retryCount = 0;
        
        while (retryCount < MAX_RETRIES) {
            try {
                System.out.println("Connecting to server...");
                
                // Connect with timeout
                socket = new Socket();
                socket.connect(
                    new InetSocketAddress(SERVER_HOST, SERVER_PORT), 
                    5000
                );
                
                System.out.println("Connected to: " + SERVER_HOST + 
                    ":" + SERVER_PORT);
                
                // Configure socket
                socket.setKeepAlive(true);
                socket.setTcpNoDelay(true);
                
                // Start communication
                communicate(socket);
                
                break; // Exit if successful
                
            } catch (SocketTimeoutException e) {
                System.err.println("Connection timeout");
                retryCount++;
                
            } catch (ConnectException e) {
                System.err.println("Connection refused: " + e.getMessage());
                retryCount++;
                
            } catch (IOException e) {
                System.err.println("Connection error: " + e.getMessage());
                retryCount++;
            }
            
            // Retry logic
            if (retryCount < MAX_RETRIES) {
                System.out.println("Retrying in " + (RETRY_DELAY/1000) + 
                    " seconds... (" + retryCount + "/" + MAX_RETRIES + ")");
                try {
                    Thread.sleep(RETRY_DELAY);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        
        if (retryCount == MAX_RETRIES) {
            System.err.println("Failed to connect after " + 
                MAX_RETRIES + " attempts");
        }
    }
    
    private static void communicate(Socket socket) {
        try (
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                socket.getOutputStream(), true
            );
            Scanner scanner = new Scanner(System.in)
        ) {
            // Start receiver thread
            Thread receiver = new Thread(() -> {
                try {
                    String response;
                    while ((response = in.readLine()) != null) {
                        System.out.println("Server: " + response);
                    }
                } catch (IOException e) {
                    System.err.println("Connection lost");
                }
            });
            receiver.setDaemon(true);
            receiver.start();
            
            // Send messages
            System.out.println("\nEnter messages (type 'quit' to exit):");
            while (true) {
                System.out.print("> ");
                String message = scanner.nextLine();
                
                if (message.isEmpty()) continue;
                
                out.println(message);
                
                if (message.equalsIgnoreCase("quit")) {
                    break;
                }
            }
            
        } catch (IOException e) {
            System.err.println("Communication error: " + e.getMessage());
            
        } finally {
            try {
                if (socket != null && !socket.isClosed()) {
                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Socket Options và Configuration

### Các Tùy Chọn Socket Quan Trọng
```java
import java.io.IOException;
import java.net.*;

public class SocketOptionsDemo {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8888);
            
            // 1. TCP_NODELAY - Tắt Nagle's algorithm
            // Gửi dữ liệu ngay lập tức, không buffer
            socket.setTcpNoDelay(true);
            System.out.println("TCP_NODELAY: " + socket.getTcpNoDelay());
            
            // 2. SO_KEEPALIVE - Gửi keepalive packets
            // Phát hiện kết nối chết
            socket.setKeepAlive(true);
            System.out.println("SO_KEEPALIVE: " + socket.getKeepAlive());
            
            // 3. SO_TIMEOUT - Read timeout
            // Thời gian chờ tối đa khi đọc dữ liệu
            socket.setSoTimeout(10000); // 10 seconds
            System.out.println("SO_TIMEOUT: " + socket.getSoTimeout() + "ms");
            
            // 4. SO_LINGER - Behavior khi đóng socket
            // true, 0 = đóng ngay, bỏ qua dữ liệu chưa gửi
            // true, n = đợi tối đa n giây
            socket.setSoLinger(true, 5);
            System.out.println("SO_LINGER: " + socket.getSoLinger());
            
            // 5. SO_RCVBUF - Receive buffer size
            socket.setReceiveBufferSize(65536); // 64KB
            System.out.println("Receive Buffer: " + 
                socket.getReceiveBufferSize() + " bytes");
            
            // 6. SO_SNDBUF - Send buffer size
            socket.setSendBufferSize(65536); // 64KB
            System.out.println("Send Buffer: " + 
                socket.getSendBufferSize() + " bytes");
            
            // 7. SO_REUSEADDR - Cho phép bind port đang TIME_WAIT
            ServerSocket serverSocket = new ServerSocket();
            serverSocket.setReuseAddress(true);
            serverSocket.bind(new InetSocketAddress(9999));
            System.out.println("SO_REUSEADDR: " + 
                serverSocket.getReuseAddress());
            
            // 8. Connection timeout khi connect
            Socket timeoutSocket = new Socket();
            timeoutSocket.connect(
                new InetSocketAddress("example.com", 80), 
                5000 // 5 seconds timeout
            );
            
            socket.close();
            serverSocket.close();
            timeoutSocket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Giải Thích Socket Options

**TCP_NODELAY – Thuật toán Nagle**
```java
// Tắt = Gửi ngay (low latency)
socket.setTcpNoDelay(true); // Cho gaming, real-time apps

// Bật = Buffer nhỏ trước khi gửi (throughput)
socket.setTcpNoDelay(false); // Cho bulk data transfer
```

**SO_KEEPALIVE – Giữ kết nối**
```java
// Phát hiện connection chết
socket.setKeepAlive(true);
// OS sẽ gửi keepalive probes định kỳ
// Nếu không nhận response -> connection dead
```

**SO_LINGER – Hành vi khi đóng Socket**
```java
// Đóng ngay, drop unsent data
socket.setSoLinger(true, 0);

// Đợi tối đa 5s để gửi hết data
socket.setSoLinger(true, 5);

// Mặc định: đóng ngay nhưng OS vẫn cố gửi data ở background
socket.setSoLinger(false, 0);
```

## Truyền dữ liệu nhị phân qua TCP

### Gửi và nhận dữ liệu nhị phân
```java
import java.io.*;
import java.net.*;

public class BinaryDataServer {
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(8888)) {
            System.out.println("Binary Data Server running...");
            
            Socket socket = serverSocket.accept();
            System.out.println("Client connected");
            
            // Use DataInputStream/DataOutputStream for primitives
            DataInputStream dis = new DataInputStream(
                socket.getInputStream()
            );
            DataOutputStream dos = new DataOutputStream(
                socket.getOutputStream()
            );
            
            // Nhận các kiểu dữ liệu primitive
            int number = dis.readInt();
            long timestamp = dis.readLong();
            double value = dis.readDouble();
            boolean flag = dis.readBoolean();
            String text = dis.readUTF();
            
            System.out.println("Received:");
            System.out.println("  Int: " + number);
            System.out.println("  Long: " + timestamp);
            System.out.println("  Double: " + value);
            System.out.println("  Boolean: " + flag);
            System.out.println("  String: " + text);
            
            // Gửi response
            dos.writeUTF("Data received successfully!");
            dos.flush();
            
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Client gửi dữ liệu nhị phân
```java
import java.io.*;
import java.net.*;

public class BinaryDataClient {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8888);
            
            DataOutputStream dos = new DataOutputStream(
                socket.getOutputStream()
            );
            DataInputStream dis = new DataInputStream(
                socket.getInputStream()
            );
            
            // Gửi các kiểu dữ liệu primitive
            dos.writeInt(12345);
            dos.writeLong(System.currentTimeMillis());
            dos.writeDouble(3.14159);
            dos.writeBoolean(true);
            dos.writeUTF("Hello Binary World!");
            dos.flush();
            
            System.out.println("Data sent!");
            
            // Nhận response
            String response = dis.readUTF();
            System.out.println("Server: " + response);
            
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Tuần tự hóa đối tượng (Object Serialization) qua TCP

### Gửi Objects qua Network
```java
import java.io.*;
import java.net.*;

// Serializable object
class Student implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int id;
    private double gpa;
    
    public Student(String name, int id, double gpa) {
        this.name = name;
        this.id = id;
        this.gpa = gpa;
    }
    
    @Override
    public String toString() {
        return String.format("Student{name='%s', id=%d, gpa=%.2f}", 
            name, id, gpa);
    }
}

public class ObjectServer {
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(8888)) {
            System.out.println("Object Server running...");
            
            Socket socket = serverSocket.accept();
            System.out.println("Client connected");
            
            // Nhận object
            ObjectInputStream ois = new ObjectInputStream(
                socket.getInputStream()
            );
            
            Student student = (Student) ois.readObject();
            System.out.println("Received: " + student);
            
            // Gửi response object
            ObjectOutputStream oos = new ObjectOutputStream(
                socket.getOutputStream()
            );
            
            Student responseStudent = new Student("Server", 999, 4.0);
            oos.writeObject(responseStudent);
            oos.flush();
            
            socket.close();
            
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### Object Client
```java
import java.io.*;
import java.net.*;

public class ObjectClient {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8888);
            
            // Gửi object
            ObjectOutputStream oos = new ObjectOutputStream(
                socket.getOutputStream()
            );
            
            Student student = new Student("John Doe", 12345, 3.75);
            oos.writeObject(student);
            oos.flush();
            
            System.out.println("Sent: " + student);
            
            // Nhận response object
            ObjectInputStream ois = new ObjectInputStream(
                socket.getInputStream()
            );
            
            Student response = (Student) ois.readObject();
            System.out.println("Received: " + response);
            
            socket.close();
            
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

## Thiết kế giao thức ứng dụng tùy chỉnh

### Giao thức đơn giản: thông điệp có độ dài xác định
```java
import java.io.*;
import java.net.*;
import java.nio.charset.StandardCharsets;

public class ProtocolServer {
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(8888)) {
            System.out.println("Protocol Server running...");
            
            while (true) {
                Socket socket = serverSocket.accept();
                new Thread(() -> handleClient(socket)).start();
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void handleClient(Socket socket) {
        try {
            DataInputStream in = new DataInputStream(
                socket.getInputStream()
            );
            DataOutputStream out = new DataOutputStream(
                socket.getOutputStream()
            );
            
            while (true) {
                // Đọc message theo protocol: [length][data]
                int length = in.readInt();
                
                if (length <= 0 || length > 65536) {
                    System.err.println("Invalid message length: " + length);
                    break;
                }
                
                byte[] buffer = new byte[length];
                in.readFully(buffer);
                
                String message = new String(buffer, StandardCharsets.UTF_8);
                System.out.println("Received (" + length + " bytes): " + message);
                
                // Echo back với protocol
                String response = "Echo: " + message;
                byte[] responseBytes = response.getBytes(StandardCharsets.UTF_8);
                
                out.writeInt(responseBytes.length);
                out.write(responseBytes);
                out.flush();
            }
            
        } catch (EOFException e) {
            System.out.println("Client disconnected");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Client sử dụng giao thức tùy chỉnh
```java
import java.io.*;
import java.net.*;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

public class ProtocolClient {
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("localhost", 8888);
            
            DataOutputStream out = new DataOutputStream(
                socket.getOutputStream()
            );
            DataInputStream in = new DataInputStream(
                socket.getInputStream()
            );
            
            Scanner scanner = new Scanner(System.in);
            
            while (true) {
                System.out.print("Enter message: ");
                String message = scanner.nextLine();
                
                if (message.equalsIgnoreCase("quit")) {
                    break;
                }
                
                // Gửi theo protocol: [length][data]
                byte[] messageBytes = message.getBytes(StandardCharsets.UTF_8);
                out.writeInt(messageBytes.length);
                out.write(messageBytes);
                out.flush();
                
                // Nhận response
                int responseLength = in.readInt();
                byte[] responseBuffer = new byte[responseLength];
                in.readFully(responseBuffer);
                
                String response = new String(
                    responseBuffer, StandardCharsets.UTF_8
                );
                System.out.println("Server: " + response);
            }
            
            scanner.close();
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Xây dựng giao thức dạng HTTP đơn giản

### Ví dụ HTTP Server cơ bản
```java
import java.io.*;
import java.net.*;
import java.text.SimpleDateFormat;
import java.util.*;

public class SimpleHTTPServer {
    private static final int PORT = 8080;
    
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("HTTP Server running on port " + PORT);
            System.out.println("Open http://localhost:" + PORT + " in browser\n");
            
            while (true) {
                Socket socket = serverSocket.accept();
                new Thread(() -> handleRequest(socket)).start();
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void handleRequest(Socket socket) {
        try {
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter out = new PrintWriter(
                socket.getOutputStream()
            );
            
            // Parse HTTP request
            String requestLine = in.readLine();
            if (requestLine == null) return;
            
            System.out.println("Request: " + requestLine);
            
            // Read headers
            Map<String, String> headers = new HashMap<>();
            String line;
            while ((line = in.readLine()) != null && !line.isEmpty()) {
                String[] parts = line.split(": ", 2);
                if (parts.length == 2) {
                    headers.put(parts[0], parts[1]);
                }
            }
            
            // Parse request line
            String[] requestParts = requestLine.split(" ");
            String method = requestParts[0];
            String path = requestParts[1];
            
            // Generate response
            String htmlContent = generateHTML(method, path, headers);
            
            // Send HTTP response
            out.println("HTTP/1.1 200 OK");
            out.println("Content-Type: text/html; charset=UTF-8");
            out.println("Content-Length: " + htmlContent.length());
            out.println("Connection: close");
            out.println("Server: SimpleJavaHTTP/1.0");
            out.println("Date: " + getHTTPDate());
            out.println(); // Empty line between headers and body
            out.println(htmlContent);
            out.flush();
            
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static String generateHTML(String method, String path, 
                                      Map<String, String> headers) {
        StringBuilder html = new StringBuilder();
        html.append("<!DOCTYPE html>\n");
        html.append("<html>\n");
        html.append("<head>\n");
        html.append("  <title>Simple HTTP Server</title>\n");
        html.append("  <style>\n");
        html.append("    body { font-family: Arial; margin: 40px; }\n");
        html.append("    h1 { color: #0ea5e9; }\n");
        html.append("    .info { background: #f1f5f9; padding: 20px; ");
        html.append("border-radius: 8px; }\n");
        html.append("  </style>\n");
        html.append("</head>\n");
        html.append("<body>\n");
        html.append("  <h1>🚀 Simple HTTP Server in Java</h1>\n");
        html.append("  <div class='info'>\n");
        html.append("    <h2>Request Information</h2>\n");
        html.append("    <p><strong>Method:</strong> " + method + "</p>\n");
        html.append("    <p><strong>Path:</strong> " + path + "</p>\n");
        html.append("    <p><strong>Time:</strong> " + new Date() + "</p>\n");
        html.append("    <h3>Headers:</h3>\n");
        html.append("    <ul>\n");
        
        for (Map.Entry<String, String> entry : headers.entrySet()) {
            html.append("      <li><strong>").append(entry.getKey())
                .append(":</strong> ").append(entry.getValue())
                .append("</li>\n");
        }
        
        html.append("    </ul>\n");
        html.append("  </div>\n");
        html.append("</body>\n");
        html.append("</html>");
        
        return html.toString();
    }
    
    private static String getHTTPDate() {
        SimpleDateFormat dateFormat = new SimpleDateFormat(
            "EEE, dd MMM yyyy HH:mm:ss z", Locale.US
        );
        dateFormat.setTimeZone(TimeZone.getTimeZone("GMT"));
        return dateFormat.format(new Date());
    }
}
```

## Quản lý Pool kết nối TCP

### Ví dụ Connection Pool đơn giản
```java
import java.io.IOException;
import java.net.Socket;
import java.util.concurrent.*;

public class ConnectionPool {
    private final String host;
    private final int port;
    private final int maxConnections;
    private final BlockingQueue<Socket> pool;
    
    public ConnectionPool(String host, int port, int maxConnections) {
        this.host = host;
        this.port = port;
        this.maxConnections = maxConnections;
        this.pool = new LinkedBlockingQueue<>(maxConnections);
        
        // Initialize pool
        for (int i = 0; i < maxConnections; i++) {
            try {
                Socket socket = new Socket(host, port);
                socket.setKeepAlive(true);
                pool.offer(socket);
                System.out.println("Connection " + (i+1) + " created");
            } catch (IOException e) {
                System.err.println("Failed to create connection: " + 
                    e.getMessage());
            }
        }
    }
    
    public Socket getConnection() throws InterruptedException {
        return pool.take();
    }
    
    public void releaseConnection(Socket socket) {
        if (socket != null && !socket.isClosed()) {
            pool.offer(socket);
        }
    }
    
    public void shutdown() {
        for (Socket socket : pool) {
            try {
                if (socket != null && !socket.isClosed()) {
                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        pool.clear();
        System.out.println("Connection pool shutdown");
        }

public int getAvailableConnections() {
    return pool.size();
}

// Demo usage
public static void main(String[] args) {
    ConnectionPool pool = new ConnectionPool("localhost", 8888, 5);
    
    // Simulate multiple clients
    ExecutorService executor = Executors.newFixedThreadPool(10);
    
    for (int i = 0; i < 10; i++) {
        final int clientId = i;
        executor.submit(() -> {
            try {
                Socket socket = pool.getConnection();
                System.out.println("Client " + clientId + 
                    " got connection. Available: " + 
                    pool.getAvailableConnections());
                
                // Simulate work
                Thread.sleep(2000);
                
                pool.releaseConnection(socket);
                System.out.println("Client " + clientId + 
                    " released connection. Available: " + 
                    pool.getAvailableConnections());
                
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
    
    executor.shutdown();
    try {
        executor.awaitTermination(30, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    pool.shutdown();
}
}
## Giám sát hiệu năng TCP

### Theo dõi kết nối và lưu lượng TCP
```java
import java.io.IOException;
import java.net.*;
import java.util.concurrent.atomic.*;

public class TCPMonitor {
    private AtomicLong totalConnections = new AtomicLong(0);
    private AtomicLong activeConnections = new AtomicLong(0);
    private AtomicLong bytesReceived = new AtomicLong(0);
    private AtomicLong bytesSent = new AtomicLong(0);
    private long startTime;
    
    public TCPMonitor() {
        this.startTime = System.currentTimeMillis();
        startMonitorThread();
    }
    
    public void connectionAccepted() {
        totalConnections.incrementAndGet();
        activeConnections.incrementAndGet();
    }
    
    public void connectionClosed() {
        activeConnections.decrementAndGet();
    }
    
    public void bytesReceived(long bytes) {
        bytesReceived.addAndGet(bytes);
    }
    
    public void bytesSent(long bytes) {
        bytesSent.addAndGet(bytes);
    }
    
    private void startMonitorThread() {
        Thread monitor = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(5000); // Report every 5 seconds
                    printStats();
                } catch (InterruptedException e) {
                    break;
                }
            }
        });
        monitor.setDaemon(true);
        monitor.start();
    }
    
    private void printStats() {
        long uptime = (System.currentTimeMillis() - startTime) / 1000;
        
        System.out.println("\n=== TCP Server Statistics ===");
        System.out.println("Uptime: " + uptime + "s");
        System.out.println("Total Connections: " + totalConnections.get());
        System.out.println("Active Connections: " + activeConnections.get());
        System.out.println("Bytes Received: " + 
            formatBytes(bytesReceived.get()));
        System.out.println("Bytes Sent: " + 
            formatBytes(bytesSent.get()));
        System.out.println("Avg Connection Rate: " + 
            String.format("%.2f", totalConnections.get() / (double)uptime) + 
            " conn/s");
        System.out.println("============================\n");
    }
    
    private String formatBytes(long bytes) {
        if (bytes < 1024) return bytes + " B";
        if (bytes < 1024 * 1024) return String.format("%.2f KB", bytes / 1024.0);
        return String.format("%.2f MB", bytes / (1024.0 * 1024.0));
    }
    
    // Usage example
    public static void main(String[] args) {
        TCPMonitor monitor = new TCPMonitor();
        
        try (ServerSocket serverSocket = new ServerSocket(8888)) {
            System.out.println("Monitored server running on port 8888");
            
            while (true) {
                Socket socket = serverSocket.accept();
                monitor.connectionAccepted();
                
                new Thread(() -> {
                    try {
                        byte[] buffer = new byte[1024];
                        int bytesRead;
                        
                        while ((bytesRead = socket.getInputStream().read(buffer)) != -1) {
                            monitor.bytesReceived(bytesRead);
                            socket.getOutputStream().write(buffer, 0, bytesRead);
                            monitor.bytesSent(bytesRead);
                        }
                        
                    } catch (IOException e) {
                        // Connection closed
                    } finally {
                        monitor.connectionClosed();
                        try {
                            socket.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Các nguyên tắc thực hành tốt (Best Practices)

### 1. Luôn đóng tài nguyên
```java
// ❌ BAD
Socket socket = new Socket("localhost", 8888);
// ... use socket ...
socket.close(); // Có thể không được gọi nếu exception

// ✅ GOOD
try (Socket socket = new Socket("localhost", 8888)) {
    // ... use socket ...
} // Tự động close

// ✅ GOOD (manual)
Socket socket = null;
try {
    socket = new Socket("localhost", 8888);
    // ... use socket ...
} finally {
    if (socket != null) {
        try {
            socket.close();
        } catch (IOException e) {
            // Log error
        }
    }
}
```

### 2. Thiết lập timeout hợp lý
```java
// Timeout cho connect
Socket socket = new Socket();
socket.connect(new InetSocketAddress(host, port), 5000);

// Timeout cho read operations
socket.setSoTimeout(10000);
```

### 3. Xử lý đọc dữ liệu không đầy đủ
```java
// ❌ BAD - read() có thể không đọc hết buffer
int bytesRead = inputStream.read(buffer);

// ✅ GOOD - Đọc đến khi đầy buffer
public static void readFully(InputStream in, byte[] buffer) 
    throws IOException {
    int offset = 0;
    int remaining = buffer.length;
    
    while (remaining > 0) {
        int bytesRead = in.read(buffer, offset, remaining);
        if (bytesRead == -1) {
            throw new EOFException();
        }
        offset += bytesRead;
        remaining -= bytesRead;
    }
}
```

### 4. Sử dụng Buffered Streams
```java
// ✅ GOOD - Tăng performance
BufferedReader reader = new BufferedReader(
    new InputStreamReader(socket.getInputStream())
);

BufferedOutputStream out = new BufferedOutputStream(
    socket.getOutputStream()
);
```

## Xử lý các lỗi thường gặp

### Lỗi: Address already in use
```java
// Solution: Use SO_REUSEADDR
ServerSocket serverSocket = new ServerSocket();
serverSocket.setReuseAddress(true);
serverSocket.bind(new InetSocketAddress(port));
```

### Lỗi: Broken pipe / Connection reset
```java
// Solution: Implement heartbeat/keepalive
socket.setKeepAlive(true);

// Or manual ping-pong
void sendHeartbeat(Socket socket) {
    try {
        socket.getOutputStream().write(0); // Ping
        socket.getOutputStream().flush();
    } catch (IOException e) {
        // Connection dead
    }
}
```

### Lỗi: Kết nối nửa đóng (Half-closed)
```java
// Solution: Proper shutdown sequence
socket.shutdownOutput(); // No more writes
// Read remaining data
socket.shutdownInput();  // No more reads
socket.close();
```

## Kết Luận

Trong bài viết này, chúng ta đã học:
- TCP three-way handshake và cách hoạt động
- Socket options và configuration chi tiết
- Binary data và object serialization
- Custom protocol design
- HTTP server implementation
- Connection pooling
- Performance monitoring
- Best practices và troubleshooting

## Tài Liệu Tham Khảo

- RFC 793 - Transmission Control Protocol
- TCP/IP Illustrated Vol. 1 (Stevens)
- Java Network Programming (O'Reilly)
- High Performance Browser Networking

---

*Happy TCP Coding! 🚀 Hãy thử implement các ví dụ và chia sẻ kết quả nhé!*