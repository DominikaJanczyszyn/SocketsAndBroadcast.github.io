# Workshops 5

<p>Hello everyone! Today we'll be focusing on Sockets and Broadcasting technologies for client/server systems.</p>

<p>Today, you'll modify the exercise from the second session of workshops, transitioning it from a single-user system to a client/server one.</p>

<p>The idea behind the system is to allow users to set their username, add a task, and mark it as in-progress or done. It's very simple, but it will help you better understand basic concepts from SDJ2 so far.</p>

[Full solution on Oliwier's GitHub](https://github.com/OliwierWijas/TaskApplication?fbclid=IwAR3aiqjNFYGZf-Q1nbApb4oN9YB61smzZpt6K-nhDzvdFzin-mlowyAWer4)

<p>The exercise consists of 15 steps. Let's get started.</p>

## Exercise
<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as needed.</p>

#### <span style="color: green;"> Before starting make sure that you have completed the previous learning path, that can be found [here](https://github.com/OliwierWijas/OliwierWijas.github.io/blob/main/Workshops2.md)

#### Also remember that even though we give you the full implementation of the needed classes, you can still ask questions if something is unclear to you, especially about the new topics including: the Observer pattern, the State pattern, and the MVVM pattern.

### Step 1 - Adding Json to Maven

<p>Before we start coding, we need to make sure all the needed dependencies are added to our Maven file.</p>
<p>In this exercise, we will need to use Json for sending objects over the network from client to server and vice versa.</p>
<p>Open your pom.xml file and add the following code to your dependencies</p>

```xml
<dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.10.1</version>
</dependency>

```

### Step 2 - The Broadcaster Class
<p>In our system we need a class that will be sending packages to all of clients. In that excercise that class will be called - <code>The Broadcaster class</code>
<p>Implement <code>The Broadcaster class</code> from the class diagram. Follow the instruction that Ole gave you when implementing the <code>broadcast(String message)</code> method.</p>

<blockquote>
<details>
<summary>Display solution for the Broadcaster class</summary>
      
```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class Broadcaster {
  private final InetAddress group;
  private final int port;

  public Broadcaster(String groupAddress, int port) throws IOException {
    this.group = InetAddress.getByName(groupAddress);
    this.port = port;
  }

  public synchronized void broadcast(String message) throws IOException {
    try(DatagramSocket socket = new DatagramSocket()) {
      byte[] content = message.getBytes();
      DatagramPacket packet = new DatagramPacket(content, content.length, group, port);
      socket.send(packet);
    }
  }
}
```
</details>
</blockquote>

### Step 3 - Implementing the SharedArrayList Class

<p>In this step, we'll implement the SharedArrayList class, which will manage all the relevant data about tasks in the system.</p>
<p>To ensure accessibility by our server and consistent data management, we'll create the SharedArrayList class.</p>
<p>This class should contain an ArrayList to store all the tasks and methods to facilitate data manipulation.</p>
<p>First, let's include methods from the ModelManager class that are relevant to task management:</p>
<ul>
  <li><code>addTask(Task task)</code>: Adds a new task to the list.</li>
  <li><code>startTask(Task task)</code>: Marks a task as in-progress.</li>
  <li><code>finishTask(Task task)</code>: Marks a task as done.</li>
</ul>
<p>We'll also implement a method called <code>getTasks()</code> to retrieve all the tasks in the system.</p>
<p>All methods in SharedArrayList should be synchronized to allow multiple clients to access them simultaneously without causing errors.</p>
<p>Additionally, we'll design SharedArrayList as a singleton class to ensure there is only one instance throughout the system, maintaining data consistency.</p>
<p>Try to program it yourself before looking at the solution!</p>

<blockquote>
<details>
<summary>Display solution for the SharedArrayList class</summary>
      
```java
public class SharedArrayList
{
  private ArrayList<Task> tasks;
  private static SharedArrayList instance;

  private SharedArrayList() {
    this.tasks = new ArrayList<>();
  }

  public static synchronized SharedArrayList getInstance() {
    if (instance == null) {
      instance = new SharedArrayList();
    }
    return instance;
  }

  public synchronized ArrayList<Task> getTasks()
  {
    return tasks;
  }

  public synchronized void addTask(Task task) {
    this.tasks.add(task);
  }

  public synchronized void startTask(Task task) {
    for (int i = 0; i < tasks.size(); i++)
    {
      if (tasks.get(i).equals(task)) {
        tasks.get(i).startTask();
        break;
      }
    }
  }

  public synchronized void finishTask(Task task) {
    for (int i = 0; i < tasks.size(); i++)
    {
      if (tasks.get(i).equals(task)) {
        tasks.get(i).finishTask();
        break;
      }
    }
  }
}
```
</details>
</blockquote>

### Step 4 - Implementing the Communicator Class

<p>Now that we have a class responsible for sending packages - <code>The Broadcaster class</code> and a class responsible for storing the data - <code>The SharedArrayList class</code>, we can move on to the class that will be listening for incoming messages (packets) from the clients.</p>

<p>We will call it <code>The Communicator class</code>.</p>

<p>First of all, we should think about names for our messages (protocols) that will indicate which action the Server should perform.</p>

<p>We can determine the names by looking at the methods that could be performed within the system. For example:</p>

<ul>
  <li><code>ADD</code>: Adds a new task to the list.</li>
  <li><code>START</code>: Marks a task as in-progress.</li>
  <li><code>FINISH</code>: Marks a task as done.</li>
  <li><code>GET</code>: Retrieve all the tasks in the system.</li>
  <li><code>EXIT</code>: Disconnect the user from the server.</li>
</ul>

<p>The Communicator should listen to incoming messages in a <code>while(true)</code> loop. When it 'hears' one of the first four messages, it should use JSON to send or retrieve task objects from <code>the SharedArrayList class</code> and use <code>the Broadcaster class</code> to send a response. In case of the last message, it should simply break from the loop, indicating the end of the listening process.</p>

<p>While coding, try to implement it yourself using the instructions provided by us and Ole. Refer to the solution only if needed.</p>

<blockquote>
<details>
<summary>Display solution for the Communicator class</summary>
      
```java
public class Communicator implements Runnable {
  private final Socket socket;
  private final Broadcaster broadcaster;
  private final Gson gson;
  private final SharedArrayList sharedArrayList;

  private final static String GET = "GET";
  private final static String ADD = "ADD";
  private final static String START = "START";
  private final static String FINISH = "FINISH";
  private final static String EXIT = "EXIT";

  public Communicator(Socket socket, Broadcaster broadcaster) {
    this.socket = socket;
    this.broadcaster = broadcaster;
    this.gson = new GsonBuilder().registerTypeAdapter(State.class, new StateInterfaceAdapter()).create();
    this.sharedArrayList = SharedArrayList.getInstance();
  }

  private synchronized void communicate() throws IOException {
    try {
      InputStream inputStream = socket.getInputStream();
      BufferedReader input = new BufferedReader(new InputStreamReader(inputStream));
      OutputStream outputStream = socket.getOutputStream();
      PrintWriter output = new PrintWriter(outputStream);

      loop: while (true) {
        String jsonRequest = input.readLine();
        switch (jsonRequest) {
          case GET: {
            output.println(gson.toJson(sharedArrayList.getTasks()));
            output.flush();
            System.out.println(socket.getLocalAddress() + ": Tasks arrayList request.");
            break;
          }
          case ADD: {
            String message = input.readLine();
            Task task = gson.fromJson(message, Task.class);
            sharedArrayList.addTask(task);
            System.out.println(socket.getLocalAddress() + ": Adding task request.");
            broadcaster.broadcast(gson.toJson(sharedArrayList.getTasks()));
            break;
          }
          case START: {
            String message = input.readLine();
            Task task = gson.fromJson(message, Task.class);
            sharedArrayList.startTask(task);
            System.out.println(socket.getLocalAddress() + ": Starting task request.");
            broadcaster.broadcast(gson.toJson(sharedArrayList.getTasks()));
            break;
          }
          case FINISH: {
            String message = input.readLine();
            Task task = gson.fromJson(message, Task.class);
            sharedArrayList.finishTask(task);
            System.out.println(socket.getLocalAddress() + ": Finishing task request.");
            broadcaster.broadcast(gson.toJson(sharedArrayList.getTasks()));
            break;
          }
          case EXIT: {
            System.out.println(socket.getLocalAddress() + ": Exiting request.");
            break loop;
          }
        }
      }
    }
    finally {
      socket.close();
    }
  }

  @Override public void run() {
    try {
      communicate();
    }
    catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```
</details>
</blockquote>

### Step 5 - Implementing the ServerStart Class

<p>We have implemented all the essential functionalities within the server domain. Now, let's implement the class responsible for starting the server - <code>ServerStart</code>.</p>

<p>To start the server, we need to:</p>

1. Create a ServerSocket with a specified port.
2. Create a Broadcaster object with a specified address. Note that you can use any address between 224.0.0.0 - 239.255.255.255, and any port between 1024 - 9999.
3. Create a Communicator object for handling client communication.
4. Start a new thread to handle client communication.

<blockquote>
<details>
<summary>Display solution for the ServerStart class</summary>
      
```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class ServerStart {
  public static void main(String[] args) throws IOException {
    ServerSocket serverSocket = new ServerSocket(8080);
    Broadcaster broadcaster = new Broadcaster("230.0.0.0", 8888);
    while (true) {
      System.out.println("Server ready for input.");
      Socket socket = serverSocket.accept();
      Communicator communicator = new Communicator(socket, broadcaster);
      Thread communicatorThread = new Thread(communicator);
      communicatorThread.start();
    }
  }
}
```
</details>
</blockquote>








