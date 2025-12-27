---
title: "UDP Programming trong Java: Truyền Dữ Liệu Nhanh và Hiệu Quả"
date: 2024-12-27T18:00:00+07:00
description: "Tìm hiểu UDP Socket Programming - Giao thức truyền dữ liệu không kết nối, phù hợp cho real-time applications, gaming và streaming"
tags: ["java", "udp", "datagram", "network", "real-time"]
featured_image: ""
---

Chào mừng bạn đến với bài viết về **UDP Programming trong Java**! Sau khi đã nắm vững TCP, giờ chúng ta sẽ tìm hiểu UDP - giao thức nhanh và hiệu quả cho các ứng dụng real-time.

## UDP Là Gì?

### 📦 User Datagram Protocol

UDP là giao thức truyền tải không hướng kết nối:
- **Connectionless**: Không cần thiết lập kết nối trước
- **Unreliable**: Không đảm bảo dữ liệu đến đích
- **Unordered**: Packets có thể đến không đúng thứ tự
- **Fast**: Overhead thấp, độ trễ thấp
- **Lightweight**: Không có flow control, congestion control

### 🎯 TCP vs UDP - So Sánh Chi Tiết

| Đặc điểm | TCP | UDP |
|----------|-----|-----|
| **Kết nối** | Connection-oriented | Connectionless |
| **Độ tin cậy** | Đảm bảo 100% | Best effort (không đảm bảo) |
| **Thứ tự** | Packets đến đúng thứ tự | Có thể bị đảo lộn |
| **Tốc độ** | Chậm hơn | Nhanh hơn |
| **Overhead** | 20 bytes header + retransmission | 8 bytes header |
| **Error checking** | Extensive | Basic checksum |
| **Flow control** | Có | Không |
| **Congestion control** | Có | Không |
| **Broadcasting** | Không hỗ trợ | Hỗ trợ |
| **Multicast** | Không hỗ trợ | Hỗ trợ |

### 💡 Khi Nào Dùng UDP?

**Sử dụng UDP cho:**
- **Real-time applications**: Gaming, video calls, live streaming
- **IoT sensors**: Dữ liệu cảm biến định kỳ
- **DNS queries**: Tra cứu nhanh
- **VoIP**: Voice over IP
- **Broadcasting/Multicasting**: Gửi đến nhiều máy
- **Time-sensitive data**: Dữ liệu cũ không còn giá trị

**KHÔNG dùng UDP cho:**
- File transfer (cần đảm bảo toàn vẹn)
- Email, messaging (cần tin cậy)
- Web browsing (HTTP/HTTPS dùng TCP)
- Database transactions

## UDP Datagram Structure

### UDP Header Format
```
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|     Source      |   Destination   |
|      Port       |       Port      |
+--------+--------+--------+--------+
|     Length      |    Checksum     |
+--------+--------+--------+--------+
|                                   |
|          Data octets ...          |
+-----------------------------------+
```

**Header chỉ 8 bytes:**
- Source Port (2 bytes)
- Destination Port (2 bytes)  
- Length (2 bytes)
- Checksum (2 bytes)

## UDP Socket Cơ Bản

### Simple UDP Server
```java
import java.io.IOException;
import java.net.*;

public class UDPServer {
    private static final int PORT = 9876;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        DatagramSocket socket = null;
        
        try {
            // Tạo UDP socket
            socket = new DatagramSocket(PORT);
            System.out.println("=== UDP Server ===");
            System.out.println("Listening on port: " + PORT);
            System.out.println("Waiting for datagrams...\n");
            
            byte[] receiveBuffer = new byte[BUFFER_SIZE];
            
            while (true) {
                // Tạo DatagramPacket để nhận dữ liệu
                DatagramPacket receivePacket = new DatagramPacket(
                    receiveBuffer, 
                    receiveBuffer.length
                );
                
                // Nhận datagram (blocking call)
                socket.receive(receivePacket);
                
                // Lấy thông tin từ packet
                String message = new String(
                    receivePacket.getData(), 
                    0, 
                    receivePacket.getLength()
                );
                InetAddress clientAddress = receivePacket.getAddress();
                int clientPort = receivePacket.getPort();
                
                System.out.println("Received from " + 
                    clientAddress.getHostAddress() + ":" + clientPort);
                System.out.println("Message: " + message);
                System.out.println("Length: " + receivePacket.getLength() + " bytes\n");
                
                // Gửi response về client
                String response = "Echo: " + message;
                byte[] sendBuffer = response.getBytes();
                
                DatagramPacket sendPacket = new DatagramPacket(
                    sendBuffer,
                    sendBuffer.length,
                    clientAddress,
                    clientPort
                );
                
                socket.send(sendPacket);
                System.out.println("Response sent\n");
                
                // Clear buffer
                receiveBuffer = new byte[BUFFER_SIZE];
            }
            
        } catch (SocketException e) {
            System.err.println("Socket error: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("IO error: " + e.getMessage());
        } finally {
            if (socket != null && !socket.isClosed()) {
                socket.close();
                System.out.println("Socket closed");
            }
        }
    }
}
```

### Simple UDP Client
```java
import java.io.IOException;
import java.net.*;
import java.util.Scanner;

public class UDPClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 9876;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        DatagramSocket socket = null;
        
        try {
            // Tạo UDP socket (không cần chỉ định port)
            socket = new DatagramSocket();
            socket.setSoTimeout(5000); // 5 seconds timeout
            
            InetAddress serverAddress = InetAddress.getByName(SERVER_HOST);
            
            Scanner scanner = new Scanner(System.in);
            System.out.println("=== UDP Client ===");
            System.out.println("Enter messages (type 'quit' to exit):\n");
            
            while (true) {
                System.out.print("> ");
                String message = scanner.nextLine();
                
                if (message.equalsIgnoreCase("quit")) {
                    break;
                }
                
                // Gửi datagram
                byte[] sendBuffer = message.getBytes();
                DatagramPacket sendPacket = new DatagramPacket(
                    sendBuffer,
                    sendBuffer.length,
                    serverAddress,
                    SERVER_PORT
                );
                
                socket.send(sendPacket);
                System.out.println("Sent: " + message);
                
                // Nhận response
                byte[] receiveBuffer = new byte[BUFFER_SIZE];
                DatagramPacket receivePacket = new DatagramPacket(
                    receiveBuffer,
                    receiveBuffer.length
                );
                
                try {
                    socket.receive(receivePacket);
                    
                    String response = new String(
                        receivePacket.getData(),
                        0,
                        receivePacket.getLength()
                    );
                    
                    System.out.println("Server: " + response + "\n");
                    
                } catch (SocketTimeoutException e) {
                    System.out.println("No response (timeout)\n");
                }
            }
            
            scanner.close();
            
        } catch (UnknownHostException e) {
            System.err.println("Unknown host: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("IO error: " + e.getMessage());
        } finally {
            if (socket != null && !socket.isClosed()) {
                socket.close();
                System.out.println("Socket closed");
            }
        }
    }
}
```

### Giải Thích Code

**DatagramSocket**
```java
// Server: bind to specific port
DatagramSocket socket = new DatagramSocket(PORT);

// Client: OS assigns random port
DatagramSocket socket = new DatagramSocket();
```

**DatagramPacket**
```java
// Để nhận dữ liệu
DatagramPacket packet = new DatagramPacket(buffer, buffer.length);

// Để gửi dữ liệu
DatagramPacket packet = new DatagramPacket(
    data, data.length, address, port
);
```

**Gửi và Nhận**
```java
socket.send(packet);    // Gửi datagram
socket.receive(packet); // Nhận datagram (blocking)
```

## UDP Broadcast

### Broadcasting to All Hosts
```java
import java.io.IOException;
import java.net.*;

public class UDPBroadcastSender {
    private static final int PORT = 9876;
    private static final String BROADCAST_ADDRESS = "255.255.255.255";
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket();
            
            // Enable broadcast
            socket.setBroadcast(true);
            
            InetAddress broadcastAddress = InetAddress.getByName(BROADCAST_ADDRESS);
            
            System.out.println("=== UDP Broadcast Sender ===");
            System.out.println("Broadcasting to: " + BROADCAST_ADDRESS);
            
            // Gửi 10 broadcast messages
            for (int i = 1; i <= 10; i++) {
                String message = "Broadcast message #" + i;
                byte[] buffer = message.getBytes();
                
                DatagramPacket packet = new DatagramPacket(
                    buffer,
                    buffer.length,
                    broadcastAddress,
                    PORT
                );
                
                socket.send(packet);
                System.out.println("Sent: " + message);
                
                Thread.sleep(1000); // 1 giây 1 message
            }
            
            socket.close();
            System.out.println("\nBroadcast completed");
            
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### Broadcast Receiver
```java
import java.io.IOException;
import java.net.*;

public class UDPBroadcastReceiver {
    private static final int PORT = 9876;
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket(PORT);
            socket.setBroadcast(true);
            
            System.out.println("=== UDP Broadcast Receiver ===");
            System.out.println("Listening on port: " + PORT);
            System.out.println("Waiting for broadcasts...\n");
            
            byte[] buffer = new byte[1024];
            
            while (true) {
                DatagramPacket packet = new DatagramPacket(
                    buffer, buffer.length
                );
                
                socket.receive(packet);
                
                String message = new String(
                    packet.getData(), 0, packet.getLength()
                );
                
                System.out.println("Received: " + message);
                System.out.println("From: " + 
                    packet.getAddress().getHostAddress() + 
                    ":" + packet.getPort() + "\n");
                
                buffer = new byte[1024];
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## UDP với Connect()

### Connected UDP Socket
```java
import java.io.IOException;
import java.net.*;

public class ConnectedUDPClient {
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket();
            InetAddress serverAddress = InetAddress.getByName("localhost");
            int serverPort = 9876;
            
            // "Connect" UDP socket
            // Không thiết lập kết nối thực sự, chỉ filter packets
            socket.connect(serverAddress, serverPort);
            
            System.out.println("UDP Socket 'connected' to " + 
                serverAddress + ":" + serverPort);
            
            // Lợi ích của connect():
            // 1. Không cần chỉ định address/port mỗi lần gửi
            // 2. Chỉ nhận packets từ connected address
            // 3. Nhận ICMP errors
            
            for (int i = 0; i < 5; i++) {
                String message = "Message #" + (i + 1);
                byte[] sendBuffer = message.getBytes();
                
                // Gửi không cần address/port
                DatagramPacket sendPacket = new DatagramPacket(
                    sendBuffer, sendBuffer.length
                );
                socket.send(sendPacket);
                
                System.out.println("Sent: " + message);
                
                // Nhận response
                byte[] receiveBuffer = new byte[1024];
                DatagramPacket receivePacket = new DatagramPacket(
                    receiveBuffer, receiveBuffer.length
                );
                
                socket.receive(receivePacket);
                
                String response = new String(
                    receivePacket.getData(), 0, receivePacket.getLength()
                );
                System.out.println("Received: " + response + "\n");
                
                Thread.sleep(1000);
            }
            
            socket.disconnect();
            socket.close();
            
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## Real-time Gaming Example

### Simple Multiplayer Game Server
```java
import java.io.IOException;
import java.net.*;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

class Player {
    String id;
    InetAddress address;
    int port;
    int x, y;
    long lastUpdate;
    
    public Player(String id, InetAddress address, int port) {
        this.id = id;
        this.address = address;
        this.port = port;
        this.x = 0;
        this.y = 0;
        this.lastUpdate = System.currentTimeMillis();
    }
}

public class GameServer {
    private static final int PORT = 9999;
    private static final int TICK_RATE = 20; // 20 updates/second
    private static ConcurrentHashMap<String, Player> players = 
        new ConcurrentHashMap<>();
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket(PORT);
            
            System.out.println("=== Game Server ===");
            System.out.println("Running on port: " + PORT);
            System.out.println("Tick rate: " + TICK_RATE + " Hz\n");
            
            // Game loop thread
            Thread gameLoop = new Thread(() -> runGameLoop(socket));
            gameLoop.start();
            
            // Receive player inputs
            byte[] buffer = new byte[256];
            
            while (true) {
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);
                
                String message = new String(
                    packet.getData(), 0, packet.getLength()
                );
                
                handlePlayerInput(
                    message, 
                    packet.getAddress(), 
                    packet.getPort()
                );
                
                buffer = new byte[256];
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void handlePlayerInput(String message, 
                                         InetAddress address, int port) {
        String[] parts = message.split(":");
        String command = parts[0];
        
        if (command.equals("JOIN")) {
            String playerId = parts[1];
            Player player = new Player(playerId, address, port);
            players.put(playerId, player);
            System.out.println("Player joined: " + playerId + 
                " (Total: " + players.size() + ")");
            
        } else if (command.equals("MOVE")) {
            String playerId = parts[1];
            int dx = Integer.parseInt(parts[2]);
            int dy = Integer.parseInt(parts[3]);
            
            Player player = players.get(playerId);
            if (player != null) {
                player.x += dx;
                player.y += dy;
                player.lastUpdate = System.currentTimeMillis();
            }
            
        } else if (command.equals("LEAVE")) {
            String playerId = parts[1];
            players.remove(playerId);
            System.out.println("Player left: " + playerId);
        }
    }
    
    private static void runGameLoop(DatagramSocket socket) {
        long lastUpdate = System.currentTimeMillis();
        int frameDelay = 1000 / TICK_RATE;
        
        while (true) {
            long now = System.currentTimeMillis();
            long elapsed = now - lastUpdate;
            
            if (elapsed >= frameDelay) {
                // Broadcast game state to all players
                broadcastGameState(socket);
                
                // Remove inactive players (timeout 10s)
                removeInactivePlayers(10000);
                
                lastUpdate = now;
            }
            
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
    
    private static void broadcastGameState(DatagramSocket socket) {
        if (players.isEmpty()) return;
        
        // Build state message: STATE:id1,x1,y1:id2,x2,y2:...
        StringBuilder state = new StringBuilder("STATE");
        
        for (Player player : players.values()) {
            state.append(":").append(player.id)
                 .append(",").append(player.x)
                 .append(",").append(player.y);
        }
        
        byte[] buffer = state.toString().getBytes();
        
        // Send to each player
        for (Player player : players.values()) {
            try {
                DatagramPacket packet = new DatagramPacket(
                    buffer, buffer.length, player.address, player.port
                );
                socket.send(packet);
            } catch (IOException e) {
                System.err.println("Failed to send to " + player.id);
            }
        }
    }
    
    private static void removeInactivePlayers(long timeout) {
        long now = System.currentTimeMillis();
        
        players.entrySet().removeIf(entry -> {
            Player player = entry.getValue();
            if (now - player.lastUpdate > timeout) {
                System.out.println("Player timeout: " + player.id);
                return true;
            }
            return false;
        });
    }
}
```

### Game Client
```java
import java.io.IOException;
import java.net.*;
import java.util.Scanner;

public class GameClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 9999;
    private static String playerId;
    private static DatagramSocket socket;
    private static InetAddress serverAddress;
    
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("Enter your player ID: ");
        playerId = scanner.nextLine();
        
        try {
            socket = new DatagramSocket();
            serverAddress = InetAddress.getByName(SERVER_HOST);
            
            // Join game
            sendMessage("JOIN:" + playerId);
            System.out.println("Joined game as: " + playerId);
            
            // Start receiver thread
            Thread receiver = new Thread(() -> receiveGameState());
            receiver.setDaemon(true);
            receiver.start();
            
            // Controls
            System.out.println("\nControls:");
            System.out.println("  w = up, s = down");
            System.out.println("  a = left, d = right");
            System.out.println("  q = quit\n");
            
            // Input loop
            while (true) {
                String input = scanner.nextLine().toLowerCase();
                
                if (input.equals("q")) {
                    sendMessage("LEAVE:" + playerId);
                    break;
                }
                
                int dx = 0, dy = 0;
                
                switch (input) {
                    case "w": dy = -1; break;
                    case "s": dy = 1; break;
                    case "a": dx = -1; break;
                    case "d": dx = 1; break;
                    default: continue;
                }
                
                sendMessage("MOVE:" + playerId + ":" + dx + ":" + dy);
            }
            
            socket.close();
            scanner.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void sendMessage(String message) {
        try {
            byte[] buffer = message.getBytes();
            DatagramPacket packet = new DatagramPacket(
                buffer, buffer.length, serverAddress, SERVER_PORT
            );
            socket.send(packet);
        } catch (IOException e) {
            System.err.println("Send error: " + e.getMessage());
        }
    }
    
    private static void receiveGameState() {
        byte[] buffer = new byte[2048];
        
        while (true) {
            try {
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);
                
                String state = new String(
                    packet.getData(), 0, packet.getLength()
                );
                
                if (state.startsWith("STATE:")) {
                    displayGameState(state.substring(6));
                }
                
                buffer = new byte[2048];
                
            } catch (IOException e) {
                break;
            }
        }
    }
    
    private static void displayGameState(String state) {
        System.out.print("\r                                                    \r");
        System.out.print("Players: ");
        
        String[] players = state.split(":");
        for (String playerData : players) {
            String[] parts = playerData.split(",");
            if (parts.length == 3) {
                System.out.print(parts[0] + "(" + parts[1] + "," + 
                    parts[2] + ") ");
            }
        }
        System.out.flush();
    }
}
```

## UDP File Transfer với Error Detection

### Reliable UDP Transfer
```java
import java.io.*;
import java.net.*;
import java.nio.ByteBuffer;
import java.util.*;

class Packet {
    int sequenceNumber;
    byte[] data;
    int length;
    
    public Packet(int seqNum, byte[] data, int length) {
        this.sequenceNumber = seqNum;
        this.data = data;
        this.length = length;
    }
    
    // Serialize packet: [seqNum(4 bytes)][length(4 bytes)][data]
    public byte[] serialize() {
        ByteBuffer buffer = ByteBuffer.allocate(8 + length);
        buffer.putInt(sequenceNumber);
        buffer.putInt(length);
        buffer.put(data, 0, length);
        return buffer.array();
    }
    
    // Deserialize packet
    public static Packet deserialize(byte[] bytes) {
        ByteBuffer buffer = ByteBuffer.wrap(bytes);
        int seqNum = buffer.getInt();
        int length = buffer.getInt();
        byte[] data = new byte[length];
        buffer.get(data);
        return new Packet(seqNum, data, length);
    }
}

public class ReliableUDPSender {
    private static final int PACKET_SIZE = 1024;
    private static final int TIMEOUT = 1000; // 1 second
    private static final int MAX_RETRIES = 3;
    
    public static void main(String[] args) {
        if (args.length < 3) {
            System.out.println("Usage: java ReliableUDPSender <host> <port> <file>");
            return;
        }
        
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        String filename = args[2];
        
        try {
            DatagramSocket socket = new DatagramSocket();
            socket.setSoTimeout(TIMEOUT);
            
            InetAddress address = InetAddress.getByName(host);
            File file = new File(filename);
            FileInputStream fis = new FileInputStream(file);
            
            long fileSize = file.length();
            int totalPackets = (int) Math.ceil(fileSize / (double) PACKET_SIZE);
            
            System.out.println("=== Reliable UDP File Sender ===");
            System.out.println("File: " + filename);
            System.out.println("Size: " + fileSize + " bytes");
            System.out.println("Packets: " + totalPackets + "\n");
            
            // Send file info
            String fileInfo = filename + ":" + fileSize + ":" + totalPackets;
            sendAndWaitAck(socket, address, port, -1, fileInfo.getBytes());
            
            // Send file data
            byte[] buffer = new byte[PACKET_SIZE];
            int seqNum = 0;
            int bytesRead;
            
            while ((bytesRead = fis.read(buffer)) != -1) {
                Packet packet = new Packet(seqNum, buffer, bytesRead);
                sendAndWaitAck(socket, address, port, seqNum, packet.serialize());
                
                System.out.printf("Sent packet %d/%d (%.1f%%)\n", 
                    seqNum + 1, totalPackets, 
                    (seqNum + 1) * 100.0 / totalPackets);
                
                seqNum++;
            }
            
            fis.close();
            socket.close();
            
            System.out.println("\nFile sent successfully!");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void sendAndWaitAck(DatagramSocket socket, 
                                      InetAddress address, int port,
                                      int seqNum, byte[] data) 
                                      throws IOException {
        DatagramPacket sendPacket = new DatagramPacket(
            data, data.length, address, port
        );
        
        byte[] ackBuffer = new byte[16];
        DatagramPacket ackPacket = new DatagramPacket(ackBuffer, ackBuffer.length);
        
        int retries = 0;
        
        while (retries < MAX_RETRIES) {
            try {
                // Send packet
                socket.send(sendPacket);
                
                // Wait for ACK
                socket.receive(ackPacket);
                
                String ack = new String(ackPacket.getData(), 0, ackPacket.getLength());
                int ackSeqNum = Integer.parseInt(ack.split(":")[1]);
                
                if (ackSeqNum == seqNum) {
                    return; // ACK received
                }
                
            } catch (SocketTimeoutException e) {
                retries++;
                System.err.println("Timeout for packet " + seqNum + 
                    ", retry " + retries);
            }
        }
        
        throw new IOException("Failed to send packet " + seqNum + 
            " after " + MAX_RETRIES + " retries");
    }
}
```

### Reliable UDP Receiver
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class ReliableUDPReceiver {
    private static final int PORT = 9999;
    private static final int BUFFER_SIZE = 2048;
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket(PORT);
            
            System.out.println("=== Reliable UDP File Receiver ===");
            System.out.println("Listening on port: " + PORT + "\n");
            
            byte[] buffer = new byte[BUFFER_SIZE];
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            
            // Receive file info
            socket.receive(packet);
            String fileInfo = new String(packet.getData(), 0, packet.getLength());
            String[] parts = fileInfo.split(":");
            String filename = "received_" + parts[0];
            long fileSize = Long.parseLong(parts[1]);
            int totalPackets = Integer.parseInt(parts[2]);
            
            System.out.println("Receiving file: " + filename);
            System.out.println("Size: " + fileSize + " bytes");
            System.out.println("Packets: " + totalPackets + "\n");
            
            // Send ACK for file info
            sendAck(socket, packet.getAddress(), packet.getPort(), -1);
            
            // Receive file data
            FileOutputStream fos = new FileOutputStream(filename);
            Set<Integer> receivedPackets = new HashSet<>();
            
            while (receivedPackets.size() < totalPackets) {
                buffer = new byte[BUFFER_SIZE];
                packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);
                
                Packet dataPacket = Packet.deserialize(packet.getData());
                
                if (!receivedPackets.contains(dataPacket.sequenceNumber)) {
                    fos.write(dataPacket.data, 0, dataPacket.length);
                    receivedPackets.add(dataPacket.sequenceNumber);
                    
                    System.out.printf("Received packet %d/%d (%.1f%%)\n",
                        receivedPackets.size(), totalPackets,
                        receivedPackets.size() * 100.0 / totalPackets);
                }
                
                // Send ACK
                sendAck(socket, packet.getAddress(), packet.getPort(), 
                    dataPacket.sequenceNumber);
            }
            
            fos.close();
            socket.close();
            
            System.out.println("\nFile received successfully!");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static void sendAck(DatagramSocket socket, InetAddress address, 
                               int port, int seqNum) throws IOException {
        String ack = "ACK:" + seqNum;
        byte[] ackData = ack.getBytes();
        DatagramPacket ackPacket = new DatagramPacket(
            ackData, ackData.length, address, port
        );
        socket.send(ackPacket);
    }
}
```

## VoIP Simulator
**Simple Voice Packet Sender** 
```java
import java.io.IOException;
import java.net.*;
import java.util.Random;

public class VoIPSender {
    private static final String HOST = "localhost";
    private static final int PORT = 8000;
    private static final int PACKET_SIZE = 160; // 20ms @ 8kHz
    private static final int PACKETS_PER_SECOND = 50;
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket();
            InetAddress address = InetAddress.getByName(HOST);
            
            System.out.println("=== VoIP Sender ===");
            System.out.println("Sending to: " + HOST + ":" + PORT);
            System.out.println("Packet size: " + PACKET_SIZE + " bytes");
            System.out.println("Rate: " + PACKETS_PER_SECOND + " packets/second\n");
            
            Random random = new Random();
            long startTime = System.currentTimeMillis();
            int packetCount = 0;
            
            // Simulate 10 seconds of voice
            while (packetCount < PACKETS_PER_SECOND * 10) {
                // Simulate voice data
                byte[] voiceData = new byte[PACKET_SIZE];
                random.nextBytes(voiceData);
                
                // Add timestamp and sequence number
                long timestamp = System.currentTimeMillis() - startTime;
                byte[] packet = createVoIPPacket(packetCount, timestamp, voiceData);
                
                DatagramPacket datagramPacket = new DatagramPacket(
                    packet, packet.length, address, PORT
                );
                
                socket.send(datagramPacket);
                packetCount++;
                
                if (packetCount % PACKETS_PER_SECOND == 0) {
                    System.out.println("Sent " + packetCount + " packets");
                }
                
                // 20ms delay between packets
                Thread.sleep(20);
            }
            
            socket.close();
            System.out.println("\nTransmission completed");
            
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    private static byte[] createVoIPPacket(int seqNum, long timestamp, 
                                          byte[] voiceData) {
        // Packet format: [seqNum(4)][timestamp(8)][voiceData]
        byte[] packet = new byte[12 + voiceData.length];
        
        // Sequence number
        packet[0] = (byte) (seqNum >> 24);
        packet[1] = (byte) (seqNum >> 16);
        packet[2] = (byte) (seqNum >> 8);
        packet[3] = (byte) seqNum;
        
        // Timestamp
        for (int i = 0; i < 8; i++) {
            packet[4 + i] = (byte) (timestamp >> (56 - i * 8));
        }
        
        // Voice data
        System.arraycopy(voiceData, 0, packet, 12, voiceData.length);
        
        return packet;
    }
}
```
**Voice Packet Receiver with Jitter Buffer**
```java
import java.io.IOException;
import java.net.*;
import java.util.*;

class VoicePacket implements Comparable<VoicePacket> {
    int sequenceNumber;
    long timestamp;
    byte[] data;
    
    public VoicePacket(int seqNum, long timestamp, byte[] data) {
        this.sequenceNumber = seqNum;
        this.timestamp = timestamp;
        this.data = data;
    }
    
    @Override
    public int compareTo(VoicePacket other) {
        return Integer.compare(this.sequenceNumber, other.sequenceNumber);
    }
}

public class VoIPReceiver {
    private static final int PORT = 8000;
    private static final int JITTER_BUFFER_SIZE = 5; // 100ms buffer
    private static PriorityQueue<VoicePacket> jitterBuffer = 
        new PriorityQueue<>();
    
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket(PORT);
            
            System.out.println("=== VoIP Receiver ===");
            System.out.println("Listening on port: " + PORT);
            System.out.println("Jitter buffer: " + JITTER_BUFFER_SIZE + " packets\n");
            
            byte[] buffer = new byte[1024];
            int expectedSeqNum = 0;
            int packetsReceived = 0;
            int packetsLost = 0;
            long totalLatency = 0;
            
            while (true) {
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);
                
                long receiveTime = System.currentTimeMillis();
                
                // Parse packet
                VoicePacket voicePacket = parseVoIPPacket(packet.getData());
                
                // Calculate latency
                long latency = receiveTime - voicePacket.timestamp;
                totalLatency += latency;
                
                // Add to jitter buffer
                jitterBuffer.offer(voicePacket);
                
                // Check for packet loss
                if (voicePacket.sequenceNumber > expectedSeqNum) {
                    int lost = voicePacket.sequenceNumber - expectedSeqNum;
                    packetsLost += lost;
                    System.err.println("⚠ Lost " + lost + " packets!");
                }
                expectedSeqNum = voicePacket.sequenceNumber + 1;
                
                packetsReceived++;
                
                // Play audio when buffer is full enough
                if (jitterBuffer.size() >= JITTER_BUFFER_SIZE) {
                    VoicePacket playPacket = jitterBuffer.poll();
                    // Here you would play the audio
                    // System.out.println("Playing packet: " + playPacket.sequenceNumber);
                }
                
                // Statistics every 50 packets
                if (packetsReceived % 50 == 0) {
                    double lossRate = packetsLost * 100.0 / 
                        (packetsReceived + packetsLost);
                    double avgLatency = totalLatency / (double) packetsReceived;
                    
                    System.out.printf("Packets: %d | Lost: %d (%.2f%%) | " +
                        "Avg Latency: %.1fms | Buffer: %d\n",
                        packetsReceived, packetsLost, lossRate, 
                        avgLatency, jitterBuffer.size());
                }
                
                buffer = new byte[1024];
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static VoicePacket parseVoIPPacket(byte[] data) {
        // Parse sequence number
        int seqNum = ((data[0] & 0xFF) << 24) |
                    ((data[1] & 0xFF) << 16) |
                    ((data[2] & 0xFF) << 8) |
                    (data[3] & 0xFF);
        
        // Parse timestamp
        long timestamp = 0;
        for (int i = 0; i < 8; i++) {
            timestamp = (timestamp << 8) | (data[4 + i] & 0xFF);
        }
        
        // Extract voice data
        byte[] voiceData = new byte[data.length - 12];
        System.arraycopy(data, 12, voiceData, 0, voiceData.length);
        
        return new VoicePacket(seqNum, timestamp, voiceData);
    }
}
```

## UDP Socket Options
**Configuring UDP Sockets**
```java
import java.io.IOException;
import java.net.*;

public class UDPSocketOptions {
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket();
            
            // 1. SO_TIMEOUT - Read timeout
            socket.setSoTimeout(5000); // 5 seconds
            System.out.println("SO_TIMEOUT: " + socket.getSoTimeout() + "ms");
            
            // 2. SO_RCVBUF - Receive buffer size
            socket.setReceiveBufferSize(65536); // 64KB
            System.out.println("Receive Buffer: " + 
                socket.getReceiveBufferSize() + " bytes");
            
            // 3. SO_SNDBUF - Send buffer size
            socket.setSendBufferSize(65536); // 64KB
            System.out.println("Send Buffer: " + 
                socket.getSendBufferSize() + " bytes");
            
            // 4. SO_REUSEADDR - Reuse address
            socket.setReuseAddress(true);
            System.out.println("SO_REUSEADDR: " + socket.getReuseAddress());
            
            // 5. SO_BROADCAST - Enable broadcast
            socket.setBroadcast(true);
            System.out.println("SO_BROADCAST: " + socket.getBroadcast());
            
            // 6. IP_TOS - Type of Service
            socket.setTrafficClass(0x10); // Low delay
            System.out.println("Traffic Class: " + 
                Integer.toHexString(socket.getTrafficClass()));
            
            // Get local port
            System.out.println("Local Port: " + socket.getLocalPort());
            
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Các nguyên tắc thực hành tốt (Best Practices)
### 1. Luôn thiết lập thời gian chờ (Timeout)
```java
// ✅ GOOD
DatagramSocket socket = new DatagramSocket();
socket.setSoTimeout(5000); // Prevent infinite blocking
```
### 2. Xử lý tình trạng mất gói tin
```java
// Implement retransmission
int retries = 0;
while (retries < MAX_RETRIES) {
    try {
        socket.send(packet);
        socket.receive(ackPacket);
        break; // Success
    } catch (SocketTimeoutException e) {
        retries++;
    }
}
```
### 3. Kiểm tra và xác thực kích thước gói tin
```java
// UDP packets có giới hạn
final int MAX_UDP_SIZE = 65507; // 65535 - 20(IP) - 8(UDP)

if (data.length > MAX_UDP_SIZE) {
    throw new IllegalArgumentException("Packet too large");
}
```
### 4. Sử dụng số thứ tự gói tin (Sequence Numbers)
```java
// Detect packet loss and reordering
class PacketWithSeq {
    int seqNum;
    byte[] data;
}
```
## 🛠️ Troubleshooting

### Problem: Packets bị mất

**Nguyên nhân:**
- Network congestion (tắc nghẽn mạng)
- Buffer overflow (tràn bộ đệm)
- Firewall blocking (tường lửa chặn gói tin)

**Giải pháp:**

```java
// Tăng receive buffer
socket.setReceiveBufferSize(262144); // 256KB
// Implement acknowledgment
// Add sequence numbers
// Use retransmission
```
### Problem: Packets không đúng thứ tự
**Giải pháp:**
```java
// Use sequence numbers và reorder buffer
PriorityQueue<Packet> buffer = new PriorityQueue<>();
```
### Problem: Packets không đúng thứ tự
**Kiểm tra:**
```java
long sendTime = System.currentTimeMillis();
// ... send packet ...
long receiveTime = System.currentTimeMillis();
long rtt = receiveTime - sendTime;
System.out.println("RTT: " + rtt + "ms");
```
## Kết Luận
Trong bài viết này, chúng ta đã học:
- Khái niệm và đặc điểm của UDP
- UDP Socket Programming cơ bản
- Broadcasting và Connected UDP
- Real-time gaming với UDP
- Reliable UDP file transfer
- VoIP simulation với jitter buffer
- Socket options và best practices
## Tài Liệu Tham Khảo

- RFC 768 - User Datagram Protocol
- UDP Network Programming (O'Reilly)
- Real-Time Transport Protocol (RTP)
- WebRTC Standards

---
Happy UDP Coding! 🚀 Hãy thử build một ứng dụng real-time và chia sẻ nhé!