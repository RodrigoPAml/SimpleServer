# Simple Server

## Description
Simple Server is a server and client implementation over TCP in C# with: text based comunication, client identification and safe thread operations

It's ideal for builindg more complex things on top of it and in an agile way.

The text encoding used is ASCII but can be changed in the source code if needed.

## Features

  - Customizable server with thread safe operations and ASCII comunnication
  - Each client is identified with an id provided by the server
  - Customizable client

## Usage of custom server

```C#
static void Main()
{
    ExampleServer server = new ExampleServer(55555);
    server.Run();
} 
```

```C# 
public class ExampleServer : Server
{
    public ExampleServer(int port) : base(port)
    {
    }

    protected override void OnServerStart()
    {
        Logger.Log("[Server]: Started");
    }

    protected override void OnClientConnected(long clientId)
    {
        Logger.Log($"[Server]: Client {clientId} connected");
    }

    protected override void OnClientDisconnected(long clientId, Exception e)
    {
        Logger.Log($"[Server]: Client {clientId} disconnected");
    }

    protected override void OnRecieveMessage(long clientId, string message)
    {
        Logger.Log($"[Server]: Client {clientId} send message \"{message}\"");
    }

    protected override void OnServerStop()
    {
        Logger.Log("[Server]: Stopped");
    }
}
```

## Usage of custom client

```C#
static void Main()
{
    ExampleClient client = new ExampleClient();
  
    client.Connect("127.0.0.1", 55555);
    client.Disconnect();
} 
```

```C#
public class ExampleClient : Client
{
    public ExampleClient() : base(bufferSize)
    {
    }

    protected override void OnConnect()
    {
        Logger.Log($"[Client]: Connected into the server in {Host}:{Port} with id {Id}");
    }

    protected override void OnDisconnect(Exception e)
    {
        Logger.Log($"[Client]: Disconnected from server in {Host}:{Port}");
    }

    protected override void OnRecieveMessage(string message)
    {
        Logger.Log($"[Client]: Recieved message from server: \"{message}\"");
    }
}
```

## Sever Overview
All protected members can be turned public

### Constructor

```C#
  // Port to listen
  // bufferSize for each client to recieve messages
  // If the implemented callbacks of the server will be thread safe between they
  public Server(int port, int bufferSize = 4096, bool syncronizeCallbacks = true)
```

### Running

```C#
  // If the server is running and listening in the port
  protected bool Running { get; }
```

### Port

```C#
  // The server listening port
  protected readonly int Port;
```

### BufferSize

```C#
  // The server buffer size used by each client in bytes
  protected readonly int BufferSize;
```

### SyncronedCallbacks

```C#
  // If the implemented server callbacks are thread safe between they
  protected readonly bool SyncronizedCallbacks;
```

### ConnectedClients

```C#
  // The current id of the connected clients
  protected List<long> ConnectedClients { get; }
```

### Lock

```C#
  // The lock used to syncronize the server callbacks
  // You can use it manually if you disable the automatic use via constructor (syncronizeCallbacks)
  protected object Lock { get; private set; }
```
### Send

```C#
  // Send a message to some client
  protected void Send(long clientId, string message)
```

### Close

```C#
  // Close the connection with some client
  protected void Close(int clientId)
```

### Stop

```C#
  // Stop the server
  protected void Stop()
```

## Client Overview
All protected members can be turned public
  
### Constructor

```C#
  public Client(int bufferSize = 64_000) // The buffer size for recieving messages from server
```

### Connect

```C#
  public void Connect(string host, int port) // Connect into server with ip and port
```

### Disconnect

```C#
  // Disconnect from server, wait thread means join the client thread until everything if finished
  // If called from the inside of the callbacks of the client, this should be false to avoid deadlock
  public void Disconnect(bool waitThread = true) 
```

### Id

```C#
  // The id provided by the server for this client
  protected long Id { get; private set; }
```

### IsConnected

```C#
  // If the client is currently connected
  protected bool IsConnected { get; private set; }
```

### BufferSize

```C#
  // the current buffer size in bytes to recieve messages
  protected int BufferSize { get; private set; }
```

### Host

```C#
  // The connected server ip address
  protected int Port { get; private set; }
```

### Port

```C#
  // The connected server port 
  protected string Host { get; private set; }
```

### Send

```C#
  // Send message to the server
   protected void Send(string content)
```

## Implemeting a client for the server in other language

The layout to send a message to server is a ASCII message in the format "BEG" + message size with 10 chars + message + "END".
When connected into the server it will return an id for the client and then start listening for the client messages

```C#
string data = $"BEG{(content.Length + 16).ToString("D10")}" + content + "END";
```

## Improvements

- SSL TCP Connection
- Possibility to send UDP packets
- Use a ThreadPool instead of one thread per client to improve performance

## Examples

### 1 - Basic Client
In this example the client send a message to the server and then  the server return the message to the same client
### 2 - Chat
This example simulates a chat between the clients
### 3 - Performance
This examples calculates the throughput between two clients, a sender and a reciever

#### Local host tests

With 64000 buffer size and payload
```
14/05/2023 04:50:08 - [Client]: Throughput was 433629333,3333333 bytes per second
14/05/2023 04:50:08 - [Client]: or 433,625 megabytes per second
14/05/2023 04:50:09 - [Client]: Throughput was 434337280 bytes per second
14/05/2023 04:50:09 - [Client]: or 434,32 megabytes per second
14/05/2023 04:50:10 - [Client]: Throughput was 434941538,46153843 bytes per second
14/05/2023 04:50:10 - [Client]: or 434,9230769230769 megabytes per second
14/05/2023 04:50:11 - [Client]: Throughput was 434927407,4074074 bytes per second
14/05/2023 04:50:11 - [Client]: or 434,9259259259259 megabytes per second
14/05/2023 04:50:12 - [Client]: Throughput was 434160000 bytes per second
14/05/2023 04:50:12 - [Client]: or 434,14285714285717 megabytes per second
14/05/2023 04:50:13 - [Client]: Throughput was 434458482,7586207 bytes per second
14/05/2023 04:50:13 - [Client]: or 434,44827586206895 megabytes per second
14/05/2023 04:50:14 - [Client]: Throughput was 434030933,3333333 bytes per second
14/05/2023 04:50:14 - [Client]: or 434 megabytes per second
14/05/2023 04:50:15 - [Client]: Throughput was 434246193,5483871 bytes per second
14/05/2023 04:50:15 - [Client]: or 434,2258064516129 megabytes per second
14/05/2023 04:50:16 - [Client]: Throughput was 434494000 bytes per second
14/05/2023 04:50:16 - [Client]: or 434,46875 megabytes per second
14/05/2023 04:50:17 - [Client]: Throughput was 434860606,06060606 bytes per second
14/05/2023 04:50:17 - [Client]: or 434,8484848484849 megabytes per second
14/05/2023 04:50:18 - [Client]: Throughput was 434832941,1764706 bytes per second
14/05/2023 04:50:18 - [Client]: or 434,8235294117647 megabytes per second
14/05/2023 04:50:19 - [Client]: Throughput was 435274971,4285714 bytes per second
14/05/2023 04:50:19 - [Client]: or 435,25714285714287 megabytes per second
14/05/2023 04:50:20 - [Client]: Throughput was 435603555,5555556 bytes per second
14/05/2023 04:50:23 - [Client]: or 435,5833333333333 megabytes per second
14/05/2023 04:50:24 - [Client]: Throughput was 438027487,17948717 bytes per second
14/05/2023 04:50:24 - [Client]: or 438,02564102564105 megabytes per second
```

With only 1024 buffer size and payload we can see the difference
```
14/05/2023 04:51:41 - [Client]: or 118,20689655172414 megabytes per second
14/05/2023 04:51:42 - [Client]: Throughput was 118237866,66666667 bytes per second
14/05/2023 04:51:42 - [Client]: or 118,23333333333333 megabytes per second
14/05/2023 04:51:43 - [Client]: Throughput was 118218619,87096775 bytes per second
14/05/2023 04:51:43 - [Client]: or 118,19354838709677 megabytes per second
14/05/2023 04:51:44 - [Client]: Throughput was 118123840 bytes per second
14/05/2023 04:51:44 - [Client]: or 118,09375 megabytes per second
14/05/2023 04:51:45 - [Client]: Throughput was 118111821,57575758 bytes per second
14/05/2023 04:51:45 - [Client]: or 118,0909090909091 megabytes per second
14/05/2023 04:51:46 - [Client]: Throughput was 118154631,52941176 bytes per second
14/05/2023 04:51:46 - [Client]: or 118,1470588235294 megabytes per second
14/05/2023 04:51:47 - [Client]: Throughput was 118151753,14285715 bytes per second
14/05/2023 04:51:47 - [Client]: or 118,14285714285714 megabytes per second
14/05/2023 04:51:48 - [Client]: Throughput was 118155264 bytes per second
14/05/2023 04:51:48 - [Client]: or 118,13888888888889 megabytes per second
14/05/2023 04:51:49 - [Client]: Throughput was 118186675,8918919 bytes per second
14/05/2023 04:51:49 - [Client]: or 118,16216216216216 megabytes per second
14/05/2023 04:51:50 - [Client]: Throughput was 118236914,5263158 bytes per second
14/05/2023 04:51:50 - [Client]: or 118,23684210526316 megabytes per second
14/05/2023 04:51:51 - [Client]: Throughput was 118259449,43589744 bytes per second
14/05/2023 04:51:51 - [Client]: or 118,25641025641026 megabytes per second
14/05/2023 04:51:52 - [Client]: Throughput was 118267494,4 bytes per second
14/05/2023 04:51:52 - [Client]: or 118,25 megabytes per second
14/05/2023 04:51:53 - [Client]: Throughput was 118227893,07317074 bytes per second
14/05/2023 04:51:53 - [Client]: or 118,21951219512195 megabytes per second
14/05/2023 04:51:54 - [Client]: Throughput was 118235769,90476191 bytes per second
14/05/2023 04:51:54 - [Client]: or 118,21428571428571 megabytes per second
```

### 4 - Sending a object
In this example the client send a class object to the server

### 5 - Thread safe test

In this example many clients increment a value in the server to test if the operations are thread safe

You can change the bool in the server class to see the difference

