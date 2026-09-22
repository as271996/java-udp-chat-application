# Java UDP Chat Application

A desktop-based multi-user chat application built in Java using **UDP datagrams, Java Swing, and multithreading**.

The project follows a client-server model where multiple desktop clients connect to a central UDP server and exchange messages in real time.

The application includes:

- a Java Swing client for entering a username, server IP, and port
- a central UDP server for managing connected users and broadcasting messages
- online-user tracking
- client connection monitoring and timeout detection
- server-side commands for managing active users

This project was developed to explore core networking concepts such as UDP communication, datagram-based messaging, concurrent send/receive operations, and client-server coordination.

## Features

- Multi-user chat over UDP
- Central server for client registration and message broadcasting
- Java Swing desktop client
- Login screen for username, server IP address, and port
- Real-time message sending and receiving
- Online-user list maintained by the server
- Automatic client identification using unique IDs
- Connection-health monitoring using periodic server checks
- Automatic removal of unresponsive clients after repeated missed responses
- Graceful client disconnect handling
- Server-side administrative commands, including:
  - `/clients` to list connected users
  - `/kick` to remove a user
  - `/raw` to toggle raw server output
  - `/help` to display available commands
  - `/quit` to stop the server
- Server-originated broadcast messages to all connected clients

## UDP Client-Server Architecture

The application uses a centralized UDP-based client-server model.

```text
               ┌───────────────────┐
               │       Server      │
               │                   │
               │ Client Registry   │
               │ Message Broadcast │
               │ Health Monitoring │
               └─────────┬─────────┘
                         │
                    UDP Datagrams
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Client A │    │ Client B │    │ Client C │
    │  Swing   │    │  Swing   │    │  Swing   │
    └──────────┘    └──────────┘    └──────────┘
```

### Server Responsibilities

The server:

- listens for incoming UDP datagrams
- registers connected clients and assigns unique identifiers
- maintains the list of active users
- broadcasts chat messages to connected clients
- monitors client responsiveness
- removes disconnected or unresponsive clients
- provides administrative commands for server-side user management

### Client Responsibilities

The client:

- connects to the server using an IP address and port
- sends registration and chat messages through UDP datagrams
- receives broadcast messages from the server
- displays chat messages through a Java Swing interface
- displays the list of connected users
- handles disconnect operations

## Tech Stack

**Language:** Java 8  
**Desktop UI:** Java Swing  
**Networking:** UDP / DatagramSocket / DatagramPacket  
**Concurrency:** Java Multithreading  
**Architecture:** Client-Server  
**Build Tool / IDE:** Ant, NetBeans

## UDP Message Flow

### Client Registration

When a client starts:

1. The client opens a `DatagramSocket`.
2. It sends a connection message containing the username:

```text
/c/<username>/e/
```

3. The server generates a unique client ID.
4. The server stores the client's:
   - username
   - IP address
   - UDP port
   - unique ID
5. The generated ID is sent back to the client.

```text
Client
   │
   │ /c/username/e/
   ▼
Server
   │
   │ Generate unique ID
   │ Register client
   ▼
Client
   │
   └── /c/client-id/e/
```

### Chat Messaging

When a user sends a message:

1. The client prefixes the message with the username.
2. The message is wrapped using the application's message protocol:

```text
/m/<username>: <message>/e/
```

3. The UDP datagram is sent to the server.
4. The server receives the message and broadcasts it to every connected client.
5. Each client extracts the message content and displays it in the chat window.

```text
                UDP Message
Client A ─────────────────────► Server
                                  │
                     ┌────────────┼────────────┐
                     │            │            │
                     ▼            ▼            ▼
                 Client A     Client B     Client C
```

### Online User Updates

The server periodically sends the current connected-user list to all clients.

The user-list message uses the following format:

```text
/u/user1/n/user2/n/user3/e/
```

Clients parse this message and update the **Online Users** window.

### Connection Monitoring

Because UDP does not maintain a persistent connection, the server periodically checks whether clients are still active.

1. The server sends an identification request:

```text
/i/server
```

2. Each active client replies with its assigned ID:

```text
/i/<client-id>/e/
```

3. The server tracks missed responses.
4. After repeated failures, the client is considered timed out and is removed from the active-client list.

### Disconnect

When a client closes the application, it sends:

```text
/d/<client-id>/e/
```

The server then removes that client from its active-user registry.

## Project Structure

```text
java-udp-chat-application/
└── java-chatApp-master/
    └── java-chatApp-master/
        ├── src/
        │   └── com/
        │       └── Chat/
        │           └── ChatApp/
        │               ├── Client.java
        │               ├── ClientWindow.java
        │               ├── Login.java
        │               ├── OnlineUsers.java
        │               │
        │               └── server/
        │                   ├── Server.java
        │                   ├── ServerClient.java
        │                   ├── ServerMain.java
        │                   └── UniqueIdentifier.java
        │
        ├── build.xml
        ├── manifest.mf
        └── nbproject/
```

## Main Classes

### Client

`Client.java` handles the low-level UDP communication for each desktop client.

It is responsible for:

- creating the `DatagramSocket`
- connecting to the configured server address and port
- sending UDP datagrams
- receiving incoming datagrams
- storing the unique client ID assigned by the server
- closing the socket when the client disconnects

### Client Window

`ClientWindow.java` provides the main Java Swing chat interface and manages the client-side messaging workflow.

It handles:

- sending chat messages
- continuously listening for server messages
- parsing the application's message protocol
- displaying incoming chat messages
- updating the online-user list
- responding to server identification requests
- notifying the server when the client disconnects

### Login

`Login.java` provides the initial connection screen where the user enters:

- username
- server IP address
- server port

After validation, it launches the main chat window.

### Online Users

`OnlineUsers.java` displays the list of users currently connected to the server.

### Server

`Server.java` contains the core server-side networking logic.

It is responsible for:

- receiving UDP datagrams from clients
- registering new clients
- assigning unique client IDs
- maintaining the active-client registry
- broadcasting chat messages
- publishing online-user updates
- monitoring client responsiveness
- disconnecting timed-out clients
- processing server administration commands

### Server Client

`ServerClient.java` represents an individual connected client and stores information such as:

- username
- IP address
- UDP port
- unique client ID
- connection-monitoring attempts

### Server Main

`ServerMain.java` starts the UDP server on the configured port.

### Unique Identifier

`UniqueIdentifier.java` generates unique IDs used by the server to identify connected clients.

## Running the Application

The application contains both the UDP server and desktop client in the same Java project.

### Prerequisites

Make sure the following are installed:

- Java 8 or later
- Apache Ant, or NetBeans IDE with Ant support

### 1. Clone the Repository

```bash
git clone https://github.com/as271996/java-udp-chat-application.git
cd java-udp-chat-application
```

The NetBeans project is located inside:

```text
java-chatApp-master/java-chatApp-master
```

### 2. Build the Project

Navigate to the NetBeans project directory:

```bash
cd java-chatApp-master/java-chatApp-master
```

Build the application using Ant:

```bash
ant clean
ant jar
```

The generated JAR is created under:

```text
dist/java-chatApp-master.jar
```

### 3. Start the UDP Server

The server requires a port number as a command-line argument.

For example:

```bash
java -jar dist/java-chatApp-master.jar 8192
```

This starts the UDP server on port `8192`.

The server entry point is:

```text
com.Chat.ChatApp.server.ServerMain
```

You can choose another available port instead of `8192`.

### 4. Start a Client

Launch the Swing client using:

```bash
java -cp dist/java-chatApp-master.jar com.Chat.ChatApp.Login
```

The login window asks for:

```text
Username
Server IP Address
Server Port
```

For a server running on the same machine, use:

```text
Server IP: 127.0.0.1
Server Port: 8192
```

Use the same port that was supplied when starting the server.

### 5. Run Multiple Clients

Launch the client command multiple times to simulate multiple users:

```bash
java -cp dist/java-chatApp-master.jar com.Chat.ChatApp.Login
```

Each client can connect with a different username.

```text
                 UDP Server
                  Port 8192
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Client A    Client B    Client C
```

Once connected, users can exchange broadcast messages and view the list of currently online users.

### Server Commands

While the server is running, the following commands can be entered in the server console:

```text
/clients
/kick <username-or-id>
/raw
/help
/quit
```

Text entered directly into the server console without a `/` prefix is broadcast to all connected clients as a server message.

## Limitations and Future Improvements

This project was built as a networking-focused desktop application and can be extended further in several areas:

- Add user authentication and persistent user accounts
- Encrypt communication between clients and the server
- Add message delivery acknowledgements and retry handling
- Improve reliability for lost or out-of-order UDP packets
- Replace the custom string-based message protocol with a structured format such as JSON
- Add private messaging in addition to broadcast chat
- Add persistent server-side chat history
- Improve synchronization and thread safety around shared client state
- Support configurable message sizes instead of the current fixed UDP buffer
- Improve validation for usernames, IP addresses, and port numbers
- Add richer server administration and client-management controls
- Add automated unit and integration tests
- Replace the NetBeans/Ant setup with Maven or Gradle
- Modernize the Java Swing user interface
- Add better error handling and user-facing connection status

## Author

**Amit Singh**

Java networking project focused on UDP communication, client-server coordination, multithreading, and desktop application development.

- GitHub: [github.com/as271996](https://github.com/as271996)
- LinkedIn: [linkedin.com/in/amit-singh-sp27](https://www.linkedin.com/in/amit-singh-sp27)

## Project Context

This project was built to explore network programming through a lightweight multi-user desktop chat application.

It demonstrates:

- UDP-based client-server communication using `DatagramSocket` and `DatagramPacket`
- registration and unique client identification
- broadcast messaging across multiple connected users
- online-user tracking
- client health monitoring and timeout detection
- multithreaded message handling
- server-side administrative commands
- Java Swing desktop UI development
