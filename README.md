# Workshops 5

<p>Hello everyone! Today we'll be focusing on Sockets and Broadcasting technologies for client/server systems.</p>

<p> Today you will have to modify the exercise from the second session of workshops, in the way, that it will change from single user system to client/server one.</p>

<p>The idea behind the system is to allow users to set their username, add a task and mark it as a in-progress one or done. It's very simple, but it will help you better understand basic concepts from SDJ2 so far.</p>

[Full solution on Oliwier's github](https://github.com/OliwierWijas/TaskApplication?fbclid=IwAR3aiqjNFYGZf-Q1nbApb4oN9YB61smzZpt6K-nhDzvdFzin-mlowyAWer4)

<p>The exercise consists of 15 steps. Let's get started.</p>

## Exercise
<p>Below is a UML of the classes needed. Please note the UML diagram may not be complete, and you're welcome to add to it as is needed.</p>

##### Before starting make sure that you have completed the previous learning path, that can be found [here](https://github.com/OliwierWijas/OliwierWijas.github.io/blob/main/Workshops2.md)

##### Also remember that even though we give you full implementation of the needed classes, you can still ask questions if something is unclear to you, especially about the new topics including: the Observer pattern, the State pattern and the MVVM pattern.
#### Step 1 - Adding Json to Maven

<p>Before we start coding, we need to make sure all the needed dependencies are added to our maven file.</p>
<p>In this exercise we will need to use Json for sending object over the network from client to server and contrariwise.</p>
<p>Open you pom.xml file and add the following code to your dependencies</p>
```mavan
<dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.10.1</version>
</dependency>
```
