# I/O, NIO.2, and Networking Basics

Java provides multiple APIs for input/output operations. This guide covers traditional I/O, modern NIO.2, and networking fundamentals commonly asked in interviews.

## Definitions

- **Stream**: a sequential flow of bytes or characters (read or write).
- **Byte vs character streams**: bytes handle raw binary; characters handle text with encoding.
- **Buffer**: a fixed-size container that data is read into or written from.
- **Channel**: a bidirectional I/O connection used in NIO.
- **Blocking vs non-blocking**: blocking waits for data; non-blocking returns immediately.
- **Selector**: watches multiple channels and tells you which are ready for I/O.
- **Socket**: an endpoint for network communication.

## Illustrations

- **Stream**: like water flowing through a pipe from a source to a sink.
- **Buffer**: a bucket you fill before pouring into another container.
- **Selector**: a receptionist who tells you which phone lines have callers waiting.

## Code Examples

```java
java.nio.file.Path path = java.nio.file.Path.of("notes.txt");
String content = java.nio.file.Files.readString(path);
System.out.println(content);
```

## Interview Questions

1. When would you choose byte streams vs character streams?
2. How do channels and buffers differ from classic streams?
3. What does non-blocking I/O enable?
4. What is a selector used for?
5. Explain a simple TCP client-server flow.

---

## 1) Java I/O Overview

### Stream-Based I/O (java.io)

The original I/O package uses the **stream** abstraction:
- **Byte streams**: `InputStream`, `OutputStream`
- **Character streams**: `Reader`, `Writer`

### NIO (java.nio)

New I/O introduced in Java 1.4:
- **Buffer-based**: Data read into buffers
- **Channel-based**: Bidirectional data flow
- **Non-blocking**: Can check if data is available
- **Selectors**: Monitor multiple channels

### NIO.2 (java.nio.file) - Java 7+

Modern file operations:
- `Path`, `Paths`, `Files` classes
- File system navigation
- File attributes
- Directory watching

---

## 2) Byte Streams

### InputStream Hierarchy

```
InputStream (abstract)
├── FileInputStream
├── ByteArrayInputStream
├── BufferedInputStream (decorator)
├── DataInputStream (decorator)
├── ObjectInputStream (decorator)
└── FilterInputStream (base for decorators)
```

### OutputStream Hierarchy

```
OutputStream (abstract)
├── FileOutputStream
├── ByteArrayOutputStream
├── BufferedOutputStream
├── DataOutputStream
├── ObjectOutputStream
└── PrintStream (System.out)
```

### Basic File Reading (Byte Stream)

```java
// Traditional (avoid - resource leak if exception)
FileInputStream fis = new FileInputStream("file.txt");
int byteData;
while ((byteData = fis.read()) != -1) {
    System.out.print((char) byteData);
}
fis.close();

// Modern (try-with-resources)
try (InputStream is = new FileInputStream("file.txt")) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = is.read(buffer)) != -1) {
        // Process buffer[0..bytesRead-1]
    }
}
```

### Buffered Streams (Performance)

```java
// Always wrap with BufferedInputStream/BufferedOutputStream
try (BufferedInputStream bis = new BufferedInputStream(
        new FileInputStream("large-file.bin"))) {
    // Much faster than unbuffered
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = bis.read(buffer)) != -1) {
        process(buffer, bytesRead);
    }
}

// Writing
try (BufferedOutputStream bos = new BufferedOutputStream(
        new FileOutputStream("output.bin"))) {
    bos.write(data);
    bos.flush();  // Ensure all data is written
}
```

### Data Streams (Primitives)

```java
// Writing primitives
try (DataOutputStream dos = new DataOutputStream(
        new BufferedOutputStream(new FileOutputStream("data.bin")))) {
    dos.writeInt(42);
    dos.writeDouble(3.14);
    dos.writeUTF("Hello");
    dos.writeBoolean(true);
}

// Reading primitives (same order!)
try (DataInputStream dis = new DataInputStream(
        new BufferedInputStream(new FileInputStream("data.bin")))) {
    int i = dis.readInt();
    double d = dis.readDouble();
    String s = dis.readUTF();
    boolean b = dis.readBoolean();
}
```

---

## 3) Character Streams

### Reader/Writer Hierarchy

```
Reader (abstract)
├── InputStreamReader (bridge: bytes → chars)
│   └── FileReader
├── BufferedReader
├── StringReader
└── CharArrayReader

Writer (abstract)
├── OutputStreamWriter (bridge: chars → bytes)
│   └── FileWriter
├── BufferedWriter
├── PrintWriter
├── StringWriter
└── CharArrayWriter
```

### Reading Text Files

```java
// BufferedReader - preferred for text
try (BufferedReader reader = new BufferedReader(
        new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// With explicit encoding
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(
            new FileInputStream("file.txt"), 
            StandardCharsets.UTF_8))) {
    // Read with UTF-8 encoding
}

// Read all lines (Java 8+)
try (BufferedReader reader = Files.newBufferedReader(
        Paths.get("file.txt"))) {
    reader.lines().forEach(System.out::println);
}
```

### Writing Text Files

```java
// BufferedWriter
try (BufferedWriter writer = new BufferedWriter(
        new FileWriter("output.txt"))) {
    writer.write("Line 1");
    writer.newLine();
    writer.write("Line 2");
}

// PrintWriter (convenient methods)
try (PrintWriter writer = new PrintWriter(
        new BufferedWriter(new FileWriter("output.txt")))) {
    writer.println("Line 1");
    writer.printf("Value: %d%n", 42);
}

// With encoding
try (BufferedWriter writer = new BufferedWriter(
        new OutputStreamWriter(
            new FileOutputStream("output.txt"),
            StandardCharsets.UTF_8))) {
    writer.write("UTF-8 encoded text");
}
```

### Character Encoding

**Always specify encoding explicitly!**

```java
// BAD - uses system default (platform-dependent)
new FileReader("file.txt");

// GOOD - explicit UTF-8
new InputStreamReader(new FileInputStream("file.txt"), StandardCharsets.UTF_8);

// BEST - use NIO.2
Files.newBufferedReader(Paths.get("file.txt"), StandardCharsets.UTF_8);
```

Common encodings:
- `StandardCharsets.UTF_8` - Most common, supports all Unicode
- `StandardCharsets.ISO_8859_1` - Latin-1
- `StandardCharsets.US_ASCII` - 7-bit ASCII

---

## 4) NIO.2 File Operations (Modern Approach)

### Path and Paths

```java
// Creating paths
Path path1 = Paths.get("dir/file.txt");
Path path2 = Paths.get("dir", "subdir", "file.txt");
Path path3 = Path.of("file.txt");  // Java 11+

// Path operations
path.getFileName();      // file.txt
path.getParent();        // dir
path.getRoot();          // / or C:\
path.isAbsolute();       // true/false
path.toAbsolutePath();   // Full path
path.normalize();        // Remove . and ..
path.resolve("other");   // Append to path
path.relativize(other);  // Relative path between two

// Comparing
path1.equals(path2);
path1.startsWith("dir");
path1.endsWith(".txt");
```

### Files Class (Utility Methods)

```java
// Reading entire file
String content = Files.readString(path);                    // Java 11+
byte[] bytes = Files.readAllBytes(path);
List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);

// Writing
Files.writeString(path, "content");                         // Java 11+
Files.write(path, bytes);
Files.write(path, lines, StandardCharsets.UTF_8);
Files.write(path, lines, StandardOpenOption.APPEND);

// Copying and moving
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);

// Deleting
Files.delete(path);         // Throws if not exists
Files.deleteIfExists(path); // Returns boolean

// Creating
Files.createFile(path);
Files.createDirectory(path);
Files.createDirectories(path);  // Like mkdir -p
Files.createTempFile("prefix", ".suffix");
Files.createTempDirectory("prefix");

// Checking
Files.exists(path);
Files.notExists(path);
Files.isRegularFile(path);
Files.isDirectory(path);
Files.isReadable(path);
Files.isWritable(path);
Files.isExecutable(path);
Files.isHidden(path);
Files.isSameFile(path1, path2);

// File attributes
long size = Files.size(path);
FileTime modified = Files.getLastModifiedTime(path);
String owner = Files.getOwner(path).getName();
```

### Stream-Based File Reading

```java
// Lines as stream (lazy, memory efficient)
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(line -> !line.isEmpty())
         .map(String::toUpperCase)
         .forEach(System.out::println);
}

// Walk directory tree
try (Stream<Path> paths = Files.walk(startPath)) {
    paths.filter(Files::isRegularFile)
         .filter(p -> p.toString().endsWith(".java"))
         .forEach(System.out::println);
}

// Find files (optimized walk with filter)
try (Stream<Path> paths = Files.find(startPath, 10,
        (path, attrs) -> attrs.isRegularFile() && 
                         path.toString().endsWith(".java"))) {
    paths.forEach(System.out::println);
}

// List directory contents
try (Stream<Path> entries = Files.list(directory)) {
    entries.forEach(System.out::println);
}
```

### File Attributes

```java
// Basic attributes
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);
attrs.size();
attrs.creationTime();
attrs.lastModifiedTime();
attrs.isDirectory();
attrs.isRegularFile();
attrs.isSymbolicLink();

// POSIX attributes (Unix/Linux)
PosixFileAttributes posix = Files.readAttributes(path, PosixFileAttributes.class);
posix.owner();
posix.group();
posix.permissions();

// Setting permissions
Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rwxr-x---");
Files.setPosixFilePermissions(path, perms);
```

---

## 5) NIO Buffers and Channels

### Buffer Basics

```java
// Creating buffers
ByteBuffer buffer = ByteBuffer.allocate(1024);        // Heap buffer
ByteBuffer direct = ByteBuffer.allocateDirect(1024);  // Direct (native) buffer
ByteBuffer wrapped = ByteBuffer.wrap(byteArray);      // Wrap existing array

// Buffer properties
buffer.capacity();   // Total size (fixed)
buffer.position();   // Current read/write position
buffer.limit();      // End of valid data
buffer.remaining();  // limit - position
buffer.hasRemaining();
```

### Buffer Operations

```java
// Writing to buffer
buffer.put((byte) 65);           // Single byte
buffer.put(byteArray);           // Byte array
buffer.putInt(42);               // Primitives
buffer.putDouble(3.14);

// Flip for reading (position→0, limit→position)
buffer.flip();

// Reading from buffer
byte b = buffer.get();           // Single byte
buffer.get(byteArray);           // Into array
int i = buffer.getInt();
double d = buffer.getDouble();

// Reset for writing
buffer.clear();      // position→0, limit→capacity (doesn't erase data)
buffer.compact();    // Move unread data to start

// Rewinding
buffer.rewind();     // position→0, limit unchanged

// Marking
buffer.mark();       // Mark current position
buffer.reset();      // Return to marked position
```

### FileChannel

```java
// Reading with channel
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    while (channel.read(buffer) != -1) {
        buffer.flip();
        while (buffer.hasRemaining()) {
            System.out.print((char) buffer.get());
        }
        buffer.clear();
    }
}

// Writing with channel
try (FileChannel channel = FileChannel.open(path,
        StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    ByteBuffer buffer = ByteBuffer.wrap("Hello".getBytes());
    channel.write(buffer);
}

// Transfer between channels (efficient)
try (FileChannel src = FileChannel.open(source, StandardOpenOption.READ);
     FileChannel dest = FileChannel.open(target, 
         StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    src.transferTo(0, src.size(), dest);
    // or: dest.transferFrom(src, 0, src.size());
}
```

### Memory-Mapped Files

```java
// Map file into memory (very fast for large files)
try (FileChannel channel = FileChannel.open(path, StandardOpenOption.READ)) {
    MappedByteBuffer mapped = channel.map(
        FileChannel.MapMode.READ_ONLY, 0, channel.size());
    
    while (mapped.hasRemaining()) {
        System.out.print((char) mapped.get());
    }
}

// Read-write mapping
try (FileChannel channel = FileChannel.open(path,
        StandardOpenOption.READ, StandardOpenOption.WRITE)) {
    MappedByteBuffer mapped = channel.map(
        FileChannel.MapMode.READ_WRITE, 0, channel.size());
    
    mapped.put(0, (byte) 'X');  // Direct modification
}
```

---

## 6) Serialization

### Basic Serialization

```java
// Class must implement Serializable
public class Person implements Serializable {
    private static final long serialVersionUID = 1L;  // Version control
    
    private String name;
    private int age;
    private transient String password;  // Not serialized
    
    // Constructor, getters, setters...
}

// Serialize (write object)
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("person.ser"))) {
    oos.writeObject(person);
}

// Deserialize (read object)
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("person.ser"))) {
    Person person = (Person) ois.readObject();
}
```

### serialVersionUID

```java
// ALWAYS declare explicitly
private static final long serialVersionUID = 1L;
```

**Why?**
- Auto-generated UID changes if class changes
- Deserialization fails with `InvalidClassException`
- Explicit UID allows controlled evolution

### Customizing Serialization

```java
public class CustomPerson implements Serializable {
    private String name;
    private transient String ssn;  // Don't serialize directly
    
    // Custom serialization
    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject();  // Default serialization
        oos.writeObject(encrypt(ssn));  // Custom handling
    }
    
    private void readObject(ObjectInputStream ois) 
            throws IOException, ClassNotFoundException {
        ois.defaultReadObject();  // Default deserialization
        this.ssn = decrypt((String) ois.readObject());
    }
}
```

### Externalization (Full Control)

```java
public class ExternalPerson implements Externalizable {
    private String name;
    private int age;
    
    // Required no-arg constructor
    public ExternalPerson() {}
    
    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeUTF(name);
        out.writeInt(age);
    }
    
    @Override
    public void readExternal(ObjectInput in) 
            throws IOException, ClassNotFoundException {
        name = in.readUTF();
        age = in.readInt();
    }
}
```

### Serialization Pitfalls

1. **Security vulnerabilities**: Deserialization can execute arbitrary code
2. **Version incompatibility**: Changes to class break deserialization
3. **Performance**: Slower than other formats (JSON, Protocol Buffers)
4. **Inheritance issues**: Parent class must be serializable or have no-arg constructor

**Modern alternative**: Use JSON (Jackson, Gson) or Protocol Buffers.

---

## 7) Directory Watching (WatchService)

```java
// Watch for file system changes
WatchService watcher = FileSystems.getDefault().newWatchService();

// Register directory
Path dir = Paths.get("/path/to/watch");
dir.register(watcher,
    StandardWatchEventKinds.ENTRY_CREATE,
    StandardWatchEventKinds.ENTRY_DELETE,
    StandardWatchEventKinds.ENTRY_MODIFY);

// Process events
while (true) {
    WatchKey key = watcher.take();  // Blocks until event
    
    for (WatchEvent<?> event : key.pollEvents()) {
        WatchEvent.Kind<?> kind = event.kind();
        
        if (kind == StandardWatchEventKinds.OVERFLOW) {
            continue;  // Events may have been lost
        }
        
        @SuppressWarnings("unchecked")
        WatchEvent<Path> ev = (WatchEvent<Path>) event;
        Path filename = ev.context();
        
        System.out.println(kind + ": " + filename);
    }
    
    // Reset key to receive further events
    boolean valid = key.reset();
    if (!valid) {
        break;  // Directory no longer accessible
    }
}
```

---

## 8) Networking Basics

### URL and URLConnection

```java
// Simple URL reading
URL url = new URL("https://example.com/data.txt");
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(url.openStream()))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// With URLConnection (more control)
URLConnection conn = url.openConnection();
conn.setConnectTimeout(5000);
conn.setReadTimeout(10000);
conn.setRequestProperty("User-Agent", "Java Client");

try (InputStream is = conn.getInputStream()) {
    // Read response
}
```

### HttpURLConnection

```java
URL url = new URL("https://api.example.com/users");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();

// Configure request
conn.setRequestMethod("POST");
conn.setRequestProperty("Content-Type", "application/json");
conn.setDoOutput(true);

// Send request body
try (OutputStream os = conn.getOutputStream()) {
    os.write("{\"name\":\"John\"}".getBytes(StandardCharsets.UTF_8));
}

// Read response
int responseCode = conn.getResponseCode();
if (responseCode == HttpURLConnection.HTTP_OK) {
    try (BufferedReader reader = new BufferedReader(
            new InputStreamReader(conn.getInputStream()))) {
        String response = reader.lines().collect(Collectors.joining());
        System.out.println(response);
    }
} else {
    // Read error stream
    try (BufferedReader reader = new BufferedReader(
            new InputStreamReader(conn.getErrorStream()))) {
        String error = reader.lines().collect(Collectors.joining());
        System.err.println(error);
    }
}

conn.disconnect();
```

### HttpClient (Java 11+)

```java
// Modern HTTP client
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)
    .build();

// GET request
HttpRequest getRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/1"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(getRequest, 
    HttpResponse.BodyHandlers.ofString());

System.out.println("Status: " + response.statusCode());
System.out.println("Body: " + response.body());

// POST request
HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"John\"}"))
    .build();

// Async request
CompletableFuture<HttpResponse<String>> futureResponse = 
    client.sendAsync(postRequest, HttpResponse.BodyHandlers.ofString());

futureResponse.thenAccept(r -> {
    System.out.println("Async response: " + r.body());
});
```

### Socket Programming (TCP)

```java
// Server
try (ServerSocket serverSocket = new ServerSocket(8080)) {
    System.out.println("Server listening on port 8080");
    
    while (true) {
        Socket clientSocket = serverSocket.accept();
        System.out.println("Client connected: " + clientSocket.getInetAddress());
        
        // Handle in new thread
        new Thread(() -> handleClient(clientSocket)).start();
    }
}

private void handleClient(Socket socket) {
    try (BufferedReader in = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
         PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {
        
        String message;
        while ((message = in.readLine()) != null) {
            System.out.println("Received: " + message);
            out.println("Echo: " + message);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// Client
try (Socket socket = new Socket("localhost", 8080);
     PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
     BufferedReader in = new BufferedReader(
         new InputStreamReader(socket.getInputStream()))) {
    
    out.println("Hello Server!");
    String response = in.readLine();
    System.out.println("Server response: " + response);
}
```

### UDP Socket (DatagramSocket)

```java
// UDP Server
try (DatagramSocket socket = new DatagramSocket(9876)) {
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
    
    socket.receive(packet);  // Blocks until packet received
    
    String message = new String(packet.getData(), 0, packet.getLength());
    System.out.println("Received: " + message);
    
    // Send response
    InetAddress clientAddress = packet.getAddress();
    int clientPort = packet.getPort();
    byte[] response = "ACK".getBytes();
    DatagramPacket responsePacket = new DatagramPacket(
        response, response.length, clientAddress, clientPort);
    socket.send(responsePacket);
}

// UDP Client
try (DatagramSocket socket = new DatagramSocket()) {
    InetAddress serverAddress = InetAddress.getByName("localhost");
    byte[] data = "Hello UDP".getBytes();
    
    DatagramPacket packet = new DatagramPacket(
        data, data.length, serverAddress, 9876);
    socket.send(packet);
    
    // Receive response
    byte[] buffer = new byte[1024];
    DatagramPacket response = new DatagramPacket(buffer, buffer.length);
    socket.receive(response);
    System.out.println("Response: " + 
        new String(response.getData(), 0, response.getLength()));
}
```

---

## 9) NIO Selectors (Non-blocking I/O)

```java
// Non-blocking server with Selector
Selector selector = Selector.open();
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.configureBlocking(false);
serverChannel.bind(new InetSocketAddress(8080));
serverChannel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    selector.select();  // Blocks until channels ready
    
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> iter = selectedKeys.iterator();
    
    while (iter.hasNext()) {
        SelectionKey key = iter.next();
        iter.remove();
        
        if (key.isAcceptable()) {
            // Accept new connection
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel client = server.accept();
            client.configureBlocking(false);
            client.register(selector, SelectionKey.OP_READ);
            System.out.println("Client connected");
        }
        
        if (key.isReadable()) {
            // Read data
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(256);
            int bytesRead = client.read(buffer);
            
            if (bytesRead == -1) {
                client.close();
            } else {
                buffer.flip();
                String message = StandardCharsets.UTF_8.decode(buffer).toString();
                System.out.println("Received: " + message);
                
                // Echo back
                buffer.flip();
                client.write(buffer);
            }
        }
    }
}
```

### Selection Key Operations

| Operation | Constant | Meaning |
|-----------|----------|---------|
| Accept | `OP_ACCEPT` | Ready to accept connections (server) |
| Connect | `OP_CONNECT` | Connection established (client) |
| Read | `OP_READ` | Data available to read |
| Write | `OP_WRITE` | Ready to write data |

---

## 10) Common I/O Patterns

### Copy File (Multiple Ways)

```java
// NIO.2 (simplest)
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

// FileChannel transfer (efficient for large files)
try (FileChannel src = FileChannel.open(source, StandardOpenOption.READ);
     FileChannel dest = FileChannel.open(target, 
         StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    src.transferTo(0, src.size(), dest);
}

// Stream-based (with buffer)
try (InputStream in = new BufferedInputStream(new FileInputStream(source.toFile()));
     OutputStream out = new BufferedOutputStream(new FileOutputStream(target.toFile()))) {
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = in.read(buffer)) != -1) {
        out.write(buffer, 0, bytesRead);
    }
}
```

### Read Resource from Classpath

```java
// Using ClassLoader
InputStream is = getClass().getClassLoader()
    .getResourceAsStream("config.properties");

// Using Class (relative or absolute)
InputStream is2 = getClass().getResourceAsStream("/config.properties");

// Read as string
try (InputStream is = getClass().getResourceAsStream("/data.txt")) {
    String content = new String(is.readAllBytes(), StandardCharsets.UTF_8);
}
```

### Properties File

```java
// Loading
Properties props = new Properties();
try (InputStream is = Files.newInputStream(Paths.get("config.properties"))) {
    props.load(is);
}

String value = props.getProperty("key", "default");

// Saving
props.setProperty("newKey", "newValue");
try (OutputStream os = Files.newOutputStream(Paths.get("config.properties"))) {
    props.store(os, "Configuration");
}
```

---

## 11) Best Practices

### Always Close Resources

```java
// BAD - resource leak
FileInputStream fis = new FileInputStream("file.txt");
// If exception occurs, stream never closed

// GOOD - try-with-resources
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // Automatically closed
}

// Multiple resources
try (InputStream in = new FileInputStream("in.txt");
     OutputStream out = new FileOutputStream("out.txt")) {
    // Both closed in reverse order
}
```

### Always Specify Encoding

```java
// BAD - platform-dependent
new FileReader("file.txt");
new String(bytes);

// GOOD - explicit encoding
Files.newBufferedReader(path, StandardCharsets.UTF_8);
new String(bytes, StandardCharsets.UTF_8);
```

### Use Buffered Streams

```java
// BAD - inefficient
new FileInputStream("file.txt");

// GOOD - buffered
new BufferedInputStream(new FileInputStream("file.txt"));
```

### Prefer NIO.2 for Files

```java
// Legacy
File file = new File("path");
if (file.exists()) { ... }

// Modern NIO.2
Path path = Paths.get("path");
if (Files.exists(path)) { ... }
```

---

## 12) Interview Questions

### Streams and I/O
1. What is the difference between byte streams and character streams?
2. Why do we need `BufferedInputStream`/`BufferedOutputStream`?
3. What is the decorator pattern in Java I/O?
4. What is the difference between `FileReader` and `InputStreamReader`?
5. How do you ensure a stream is always closed?

### NIO and NIO.2
6. What is the difference between `java.io` and `java.nio`?
7. Explain Buffer operations: `flip()`, `clear()`, `compact()`.
8. What is a `FileChannel`? When would you use it?
9. What is a memory-mapped file? What are its benefits?
10. How does `Files.walk()` differ from `Files.list()`?

### Serialization
11. What is `Serializable`? What is `serialVersionUID`?
12. What does `transient` keyword do?
13. How would you customize serialization?
14. What are the security concerns with Java serialization?
15. What is `Externalizable` vs `Serializable`?

### Networking
16. What is the difference between TCP and UDP?
17. Explain how `Socket` and `ServerSocket` work.
18. What is a `Selector` in NIO? When would you use it?
19. What is non-blocking I/O? How does it differ from blocking I/O?
20. How does `HttpClient` (Java 11+) differ from `HttpURLConnection`?

### Best Practices
21. Why should you always specify character encoding?
22. What are resource leaks? How do you prevent them?
23. When would you use direct buffers vs heap buffers?
24. How do you efficiently copy large files in Java?
25. What is try-with-resources and how does it work?

---

## 13) Quick Reference

### File Operations Comparison

| Operation | java.io | NIO.2 |
|-----------|---------|-------|
| Check exists | `file.exists()` | `Files.exists(path)` |
| Create file | `file.createNewFile()` | `Files.createFile(path)` |
| Delete | `file.delete()` | `Files.delete(path)` |
| Read bytes | Manual stream | `Files.readAllBytes(path)` |
| Read lines | BufferedReader | `Files.readAllLines(path)` |
| Write | FileWriter | `Files.write(path, bytes)` |
| Copy | Manual | `Files.copy(src, dest)` |
| Move | `renameTo()` | `Files.move(src, dest)` |
| List dir | `file.listFiles()` | `Files.list(path)` |

### StandardOpenOption

| Option | Description |
|--------|-------------|
| `READ` | Open for reading |
| `WRITE` | Open for writing |
| `APPEND` | Append to end |
| `CREATE` | Create if not exists |
| `CREATE_NEW` | Fail if exists |
| `TRUNCATE_EXISTING` | Truncate existing file |
| `DELETE_ON_CLOSE` | Delete when stream closed |
| `SYNC` | Sync content and metadata |
| `DSYNC` | Sync content only |

