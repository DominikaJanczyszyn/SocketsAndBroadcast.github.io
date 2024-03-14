# Workshops 5

<p>Hello everyone! Today we'll be focusing on Sockets and Broadcasting technologies for client/server systems.</p>

<p>Today, you'll modify the exercise from the second session of workshops, transitioning it from a single-user system to a client/server one.</p>

<p>The idea behind the system is to allow users to set their username, add a task, and mark it as in-progress or done. It's very simple, but it will help you better understand basic concepts from SDJ2 so far.</p>

[Full solution on Oliwier's GitHub](https://github.com/OliwierWijas/TaskApplication?fbclid=IwAR3aiqjNFYGZf-Q1nbApb4oN9YB61smzZpt6K-nhDzvdFzin-mlowyAWer4)

<p>The exercise consists of 15 steps. Let's get started.</p>

## Exercise
<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as needed.</p>

##### Before starting make sure that you have completed the previous learning path, that can be found [here](https://github.com/OliwierWijas/OliwierWijas.github.io/blob/main/Workshops2.md)

##### Also remember that even though we give you the full implementation of the needed classes, you can still ask questions if something is unclear to you, especially about the new topics including: the Observer pattern, the State pattern, and the MVVM pattern.

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
###Step 2 - The Braodcaster Class
<p>Implement the Broadcaster class from the class diagram. Follow the instuction that Ole gave you when implementing the broadcast(String message) method.</p>

<blockquote>
<details>
<summary>Display solution for the Broadcaster class</summary>
      
```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class Broadcaster
{
  private final InetAddress group;
  private final int port;

  public Broadcaster(String groupAddress, int port) throws IOException
  {
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



