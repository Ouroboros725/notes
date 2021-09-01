https://www.baeldung.com/a-guide-to-java-sockets


## **1. Overview**[](https://www.baeldung.com/a-guide-to-java-sockets#Overview)

The term  _socket_ _programming_  refers to writing programs that execute across multiple computers in which the devices are all connected to each other using a network.

There are two communication protocols that one can use for socket programming:  **User Datagram Protocol (UDP) and Transfer Control Protocol (TCP)**.

The main difference between the two is that UDP is connectionless, meaning there is no session between the client and the server while TCP is connection-oriented, meaning an exclusive connection must first be established between client and server for communication to take place.

This tutorial presents  **an introduction to sockets programming over TCP/IP**  networks and demonstrates how to write client/server applications in Java. UDP is not a mainstream protocol and as such may not be often encountered.

## **2. Project Setup**[](https://www.baeldung.com/a-guide-to-java-sockets#Setup)

Java provides a collection of classes and interfaces that take care of low-level communication details between the client and the server.

These are mostly contained in the  _java.net_  package, so we need to make the following import:

```java
import java.net.*;
```

We also need the  _java.io_  package which gives us input and output streams to write to and read from while communicating:

```java
import java.io.*;
```

For the sake of simplicity, we'll run our client and server programs on the same computer. If we were to execute them on different networked computers, the only thing that would change is the IP address, in this case, we will use  _localhost_  on  _127.0.0.1_.

## **3. Simple Example**[](https://www.baeldung.com/a-guide-to-java-sockets#Example)

Let's get our hands dirty with the most  **basic of examples involving a client and a server**. It's going to be a two-way communication application where the client greets the server and the server responds.

Let's create the server application in a class called  _GreetServer.java_  with the following code.

We include the  _main_  method and the global variables to draw attention to how we'll be running all servers in this article. In the rest of the examples in the articles, we'll omit this kind of more repetitive code:

```java
public class GreetServer {
    private ServerSocket serverSocket;
    private Socket clientSocket;
    private PrintWriter out;
    private BufferedReader in;

    public void start(int port) {
        serverSocket = new ServerSocket(port);
        clientSocket = serverSocket.accept();
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        String greeting = in.readLine();
            if ("hello server".equals(greeting)) {
                out.println("hello client");
            }
            else {
                out.println("unrecognised greeting");
            }
    }

    public void stop() {
        in.close();
        out.close();
        clientSocket.close();
        serverSocket.close();
    }
    public static void main(String[] args) {
        GreetServer server=new GreetServer();
        server.start(6666);
    }
}
```

Let's also create a client called  _GreetClient.java_  with this code:

```java
public class GreetClient {
    private Socket clientSocket;
    private PrintWriter out;
    private BufferedReader in;

    public void startConnection(String ip, int port) {
        clientSocket = new Socket(ip, port);
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
    }

    public String sendMessage(String msg) {
        out.println(msg);
        String resp = in.readLine();
        return resp;
    }

    public void stopConnection() {
        in.close();
        out.close();
        clientSocket.close();
    }
}
```

**Let's start the server;**  in your IDE you do this by simply running it as a Java application.

And now let's send a greeting to the server using a unit test, which confirms that the server actually sends a greeting in response:

```java
@Test
public void givenGreetingClient_whenServerRespondsWhenStarted_thenCorrect() {
    GreetClient client = new GreetClient();
    client.startConnection("127.0.0.1", 6666);
    String response = client.sendMessage("hello server");
    assertEquals("hello client", response);
}
```

Don't worry if you don't entirely understand what is happening here, as this example is meant to give us a feel of what to expect later on in the article.

In the following sections, we will dissect  **socket communication**  using this simple example and dive deeper into the details with more examples.

## **4. How Sockets Work**[](https://www.baeldung.com/a-guide-to-java-sockets#Work)

We will use the above example to step through different parts of this section.

By definition, a  _socket_  is one endpoint of a two-way communication link between two programs running on different computers on a network.  [A socket is bound to a port number](https://www.baeldung.com/cs/port-vs-socket)  so that the transport layer can identify the application that data is destined to be sent to.

### **4.1. The Server**[](https://www.baeldung.com/a-guide-to-java-sockets#1-the-server)

Usually, a server runs on a specific computer on the network and has a socket that is bound to a specific port number. In our case, we use the same computer as the client and started the server on port  _6666_:

```java
ServerSocket serverSocket = new ServerSocket(6666);
```

The server just waits, listening to the socket for a client to make a connection request. This happens in the next step:

```java
Socket clientSocket = serverSocket.accept();
```

When the server code encounters the  _accept_  method, it blocks until a client makes a connection request to it.

If everything goes well, the server  _accepts_  the connection. Upon acceptance, the server gets a new socket,  _clientSocket_, bound to the same local port,  _6666_, and also has its remote endpoint set to the address and port of the client.

At this point, the new  _Socket_  object puts the server in direct connection with the client, we can then access the output and input streams to write and receive messages to and from the client respectively:

```java
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
```

From here onwards, the server is capable of exchanging messages with the client endlessly until the socket is closed with its streams.

However, in our example the server can only send a greeting response before it closes the connection, this means that if we ran our test again, the connection would be refused.

To allow continuity in communication, we will have to read from the input stream inside a  _while_  loop and only exit when the client sends a termination request, we will see this in action in the following section.

For every new client, the server needs a new socket returned by the  _accept_  call. The  _serverSocket_ is used to continue to listen for connection requests while tending to the needs of the connected clients. We have not allowed for this yet in our first example.

### **4.2. The Client**[](https://www.baeldung.com/a-guide-to-java-sockets#2-the-client)

The client must know the hostname or IP of the machine on which the server is running and the port number on which the server is listening.

To make a connection request, the client tries to rendezvous with the server on the server's machine and port:

```java
Socket clientSocket = new Socket("127.0.0.1", 6666);
```

The client also needs to identify itself to the server so it binds to a local port number, assigned by the system, that it will use during this connection. We don't deal with this ourselves.

The above constructor only creates a new socket when the server has  _accepted_  the connection, otherwise, we will get a connection refused exception. When successfully created we can then obtain input and output streams from it to communicate with the server:

```java
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
```

The input stream of the client is connected to the output stream of the server, just like the input stream of the server is connected to the output stream of the client.

## **5. Continuous Communication**[](https://www.baeldung.com/a-guide-to-java-sockets#Communication)

Our current server blocks until a client connects to it and then blocks again to listen to a message from the client, after the single message, it closes the connection because we have not dealt with continuity.

So it is only helpful in ping requests, but imagine we would like to implement a chat server, continuous back and forth communication between server and client would definitely be required.

We will have to create a while loop to continuously observe the input stream of the server for incoming messages.

Let's create a new server called  _EchoServer.java_  whose sole purpose is to echo back whatever messages it receives from clients:

```java
public class EchoServer {
    public void start(int port) {
        serverSocket = new ServerSocket(port);
        clientSocket = serverSocket.accept();
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        
        String inputLine;
        while ((inputLine = in.readLine()) != null) {
        if (".".equals(inputLine)) {
            out.println("good bye");
            break;
         }
         out.println(inputLine);
    }
}
```

Notice that we have added a termination condition where the while loop exits when we receive a period character.

We will start  _EchoServer_  using the main method just as we did for the  _GreetServer_. This time, we start it on another port such as  _4444_  to avoid confusion.

The  _EchoClient_  is similar to  _GreetClient_, so we can duplicate the code. We are separating them for clarity.

In a different test class, we shall create a test to show that multiple requests to the  _EchoServer_  will be served without the server closing the socket. This is true as long as we are sending requests from the same client.

Dealing with multiple clients is a different case, which we shall see in a subsequent section.

Let's create a  _setup_  method to initiate a connection with the server:

```java
@Before
public void setup() {
    client = new EchoClient();
    client.startConnection("127.0.0.1", 4444);
}
```

We will equally create a  _tearDown_  method to release all our resources, this is best practice for every case where we use network resources:

```java
@After
public void tearDown() {
    client.stopConnection();
}
```

Let's then test our echo server with a few requests:

```java
@Test
public void givenClient_whenServerEchosMessage_thenCorrect() {
    String resp1 = client.sendMessage("hello");
    String resp2 = client.sendMessage("world");
    String resp3 = client.sendMessage("!");
    String resp4 = client.sendMessage(".");
    
    assertEquals("hello", resp1);
    assertEquals("world", resp2);
    assertEquals("!", resp3);
    assertEquals("good bye", resp4);
}
```

This is an improvement over the initial example, where we would only communicate once before the server closed our connection;  **now we send a termination signal to tell the server when we're done with the session**.

## **6. Server With Multiple Clients**[](https://www.baeldung.com/a-guide-to-java-sockets#Multiple)

Much as the previous example was an improvement over the first one, it is still not that great a solution. A server must have the capacity to service many clients and many requests simultaneously.

**Handling multiple clients is what we are going to cover in this section.**

Another feature we will see here is that the same client could disconnect and reconnect again, without getting a connection refused exception or a connection reset on the server. Previously we were not able to do this.

This means that our server is going to be more robust and resilient across multiple requests from multiple clients.

How we will do this is to create a new socket for every new client and service that client's requests on a different thread. The number of clients being served simultaneously will equal the number of threads running.

The main thread will be running a while loop as it listens for new connections.

Enough talk, let's create another server called  _EchoMultiServer.java._  Inside it, we will create a handler thread class to manage each client's communications on its socket:

```java
public class EchoMultiServer {
    private ServerSocket serverSocket;

    public void start(int port) {
        serverSocket = new ServerSocket(port);
        while (true)
            new EchoClientHandler(serverSocket.accept()).start();
    }

    public void stop() {
        serverSocket.close();
    }

    private static class EchoClientHandler extends Thread {
        private Socket clientSocket;
        private PrintWriter out;
        private BufferedReader in;

        public EchoClientHandler(Socket socket) {
            this.clientSocket = socket;
        }

        public void run() {
            out = new PrintWriter(clientSocket.getOutputStream(), true);
            in = new BufferedReader(
              new InputStreamReader(clientSocket.getInputStream()));
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                if (".".equals(inputLine)) {
                    out.println("bye");
                    break;
                }
                out.println(inputLine);
            }

            in.close();
            out.close();
            clientSocket.close();
    }
}
```

Notice that we now call  _accept_  inside a  _while_  loop. Every time the  _while_  loop is executed, it blocks on the  _accept_  call until a new client connects, then the handler thread,  _EchoClientHandler_, is created for this client.

What happens inside the thread is what we previously did in the  _EchoServer_  where we handled only a single client. So the  _EchoMultiServer_  delegates this work to  _EchoClientHandler_  so that it can keep listening for more clients in the  _while_  loop.

We will still use  _EchoClient_  to test the server, this time we will create multiple clients each sending and receiving multiple messages from the server.

Let's start our server using its main method on port  _5555_.

For clarity, we will still put tests in a new suite:

```java
@Test
public void givenClient1_whenServerResponds_thenCorrect() {
    EchoClient client1 = new EchoClient();
    client1.startConnection("127.0.0.1", 5555);
    String msg1 = client1.sendMessage("hello");
    String msg2 = client1.sendMessage("world");
    String terminate = client1.sendMessage(".");
    
    assertEquals(msg1, "hello");
    assertEquals(msg2, "world");
    assertEquals(terminate, "bye");
}

@Test
public void givenClient2_whenServerResponds_thenCorrect() {
    EchoClient client2 = new EchoClient();
    client2.startConnection("127.0.0.1", 5555);
    String msg1 = client2.sendMessage("hello");
    String msg2 = client2.sendMessage("world");
    String terminate = client2.sendMessage(".");
    
    assertEquals(msg1, "hello");
    assertEquals(msg2, "world");
    assertEquals(terminate, "bye");
}
```

We could create as many of these test cases as we please, each spawning a new client and the server will serve all of them.

## **7. Conclusion**[](https://www.baeldung.com/a-guide-to-java-sockets#Conclusion)

In this tutorial, we've focused on  **an introduction to sockets programming over TCP/IP**  and wrote a simple Client/Server application in Java.

The full source code for the article can be found – as usual – in the  [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)  project.