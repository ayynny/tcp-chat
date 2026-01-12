# Day 8-9: TCP Chat Server/Client

## 🎯 Project Goal
Build a multi-client chat application using TCP sockets with a custom protocol.

## 📋 Requirements

### Server Requirements

1. **Connection Management**
   - Accept multiple client connections simultaneously
   - One thread per client
   - Track all connected clients
   - Handle client disconnections gracefully

2. **Message Broadcasting**
   - Receive messages from any client
   - Broadcast to all other connected clients
   - Include sender information with each message

3. **Command Support**
   - `/list` - Show all connected users
   - `/whisper <username> <message>` - Private message
   - `/quit` - Disconnect gracefully

4. **Server Console**
   - Show connection/disconnection events
   - Display all messages (for monitoring)
   - Show number of active connections

### Client Requirements

1. **Connection**
   - Connect to server with username
   - Handle connection errors

2. **Message Sending**
   - Read user input from console
   - Send to server

3. **Message Receiving**
   - Listen for messages from server
   - Display in real-time
   - Show sender username

4. **Concurrent Operations**
   - Send and receive simultaneously
   - Use separate threads for input/output

### Protocol Design

Design a simple text-based protocol. Consider:
- Message framing (how to separate messages)
- Message types (chat, command, system notification)
- Error handling

**Example Protocol Messages**:
```
JOIN:username
MSG:sender:message text
WHISPER:sender:recipient:message text
LIST_USERS:user1,user2,user3
QUIT:username
ERROR:error message
```

## 🤔 Design Challenges

1. **Protocol Design**
   - How to frame messages (newline-delimited? length-prefixed?)
   - How to encode message types?
   - How to handle multi-line messages?
   - How to indicate errors?

2. **Server Architecture**
   - How to represent a connected client?
   - How to store client list (thread-safe?)
   - How to broadcast to all clients?
   - Where to handle protocol parsing?

3. **Client Architecture**
   - How to read from socket and console simultaneously?
   - How to handle server disconnection?
   - Where to handle message formatting?

4. **Thread Safety**
   - What data is shared between client handler threads?
   - How to safely modify client list?
   - How to prevent garbled messages?

5. **Error Handling**
   - What if client crashes?
   - What if network fails?
   - How to handle malformed messages?

## 💻 Example Usage

### Starting Server
```
$ java ChatServer 8080
Server started on port 8080
Waiting for connections...
```

### Client 1 (Alice)
```
$ java ChatClient localhost 8080
Enter username: Alice
Connected to server as Alice

> Hello everyone!
> /list
Active users: Alice, Bob, Carol
> /whisper Bob Hey Bob!
[Whispered to Bob]: Hey Bob!
> /quit
Disconnected from server.
```

### Client 2 (Bob)
```
$ java ChatClient localhost 8080
Enter username: Bob
Connected to server as Bob

[Alice]: Hello everyone!
[Alice whispers]: Hey Bob!
> Hi Alice!
```

### Server Console
```
Server started on port 8080
Waiting for connections...
[CONNECT] Alice connected from /127.0.0.1:54321
Active connections: 1
[CONNECT] Bob connected from /127.0.0.1:54322
Active connections: 2
[Alice]: Hello everyone!
[Alice -> Bob]: Hey Bob!
[Bob]: Hi Alice!
[DISCONNECT] Alice disconnected
Active connections: 1
```

## 🚀 Implementation Phases

### Phase 1: Basic Server (3-4 hours)
- Accept single client connection
- Echo received messages back
- Use BufferedReader/PrintWriter
- Test with telnet

### Phase 2: Multi-Client Server (2-3 hours)
- Accept multiple clients
- One thread per client
- Broadcast messages to all clients
- Handle disconnections

### Phase 3: Basic Client (2-3 hours)
- Connect to server
- Send messages from console
- Receive and display messages
- Handle disconnection

### Phase 4: Protocol & Commands (2-3 hours)
- Design message protocol
- Implement /list, /whisper, /quit
- Add usernames
- Improve error handling

## 📚 Java Concepts

### Networking
```java
ServerSocket
Socket
InputStream / OutputStream
BufferedReader / BufferedWriter
PrintWriter
```

### Threading
```java
Thread (one per client)
ExecutorService (alternative approach)
Runnable / Callable
```

### I/O
```java
try-with-resources
InputStreamReader / OutputStreamWriter
Scanner (for console input)
```

### Collections (Thread-Safe)
```java
CopyOnWriteArrayList<ClientHandler>
Collections.synchronizedList()
ConcurrentHashMap<String, ClientHandler>
```

## 🧪 Testing Scenarios

1. **Single Client**
   - Connect, send messages, disconnect

2. **Multiple Clients**
   - Connect 3+ clients
   - Send messages from each
   - Verify all receive broadcasts

3. **Commands**
   - Test /list with varying client counts
   - Test /whisper between clients
   - Test /quit command

4. **Error Cases**
   - Kill client abruptly (Ctrl+C)
   - Send malformed messages
   - Try duplicate usernames
   - Server shutdown while clients connected

5. **Load Testing**
   - Connect 50+ clients
   - Send many messages rapidly
   - Verify no message loss

## 🐛 Common Issues & Solutions

### Issue: Blocked Reading
```java
// PROBLEM: Reading from socket blocks other operations
socket.getInputStream().read();  // Blocks forever

// SOLUTION: Use separate thread for reading
new Thread(() -> {
    while (true) {
        String msg = reader.readLine();
        handleMessage(msg);
    }
}).start();
```

### Issue: Client List Race Condition
```java
// PROBLEM: Concurrent modification of client list
clients.add(newClient);  // Not thread-safe

// SOLUTION: Use thread-safe collection
CopyOnWriteArrayList<ClientHandler> clients = new CopyOnWriteArrayList<>();
clients.add(newClient);  // Safe
```

### Issue: Socket Not Closing
```java
// PROBLEM: Resources leak
Socket socket = serverSocket.accept();
// ... use socket ...

// SOLUTION: Use try-with-resources
try (Socket socket = serverSocket.accept()) {
    // ... use socket ...
}  // Automatically closed
```

## ✨ Bonus Features

1. **Persistent Chat History**
   - Save all messages to file
   - New clients can request history

2. **Rooms/Channels**
   - Support multiple chat rooms
   - `/join <room>` command
   - Broadcast only within room

3. **Kick/Ban Commands**
   - Admin can kick users
   - Ban by IP address

4. **File Transfer**
   - `/send <username> <file>` command
   - Base64 encode for text protocol

5. **GUI Client**
   - JavaFX or Swing interface
   - Better UX than console

6. **Encryption**
   - Implement basic message encryption
   - Use javax.crypto

## ✅ Success Criteria

Your implementation should:
- ✅ Support multiple simultaneous clients (10+)
- ✅ Broadcast messages to all clients reliably
- ✅ Handle client disconnections without crashing
- ✅ Implement at least 3 commands
- ✅ Use a clear, documented protocol
- ✅ Be thread-safe
- ✅ Provide good error messages
- ✅ Clean up resources properly

## 📖 Protocol Design Tips

**Good Protocol Characteristics**:
- Simple to implement
- Easy to debug (human-readable is helpful)
- Unambiguous message boundaries
- Extensible (can add new message types)
- Error detection

**Example: Newline-Delimited Protocol**
```
Pros: Simple, human-readable
Cons: Can't have newlines in messages
Format: TYPE:arg1:arg2:arg3\n
```

**Example: Length-Prefixed Protocol**
```
Pros: Supports any content
Cons: Slightly more complex
Format: 4-byte length + message bytes
```

Choose what fits your needs!

## 🎓 Learning Outcomes

After this project, you'll understand:
- TCP socket programming in Java
- Client-server architecture
- Protocol design considerations
- Handling multiple concurrent connections
- Thread-safe communication between clients
- Graceful error handling in networked applications

This is a foundational project - many real applications use similar patterns!
# tcp-chat
# tcp-chat
# tcp-chat
