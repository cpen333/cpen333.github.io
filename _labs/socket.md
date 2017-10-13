---
layout: lab
title:  Lab 5 - Sockets
date:   2017-10-06 12:00:00
authors: [C. Antonio Sánchez]
categories: [labs, sockets, music, library]

---

# Lab 5 -- Sockets
{:.no_toc}

In this lab we are going to practice interprocess communication using sockets.  A socket is an end-point of a two-way communication channel between programs running over a network.  In our case both programs will be running on the same machine, but the same system could be run across multiple machines.

We are going to develop a music library application that stores a list of songs on a server.  We will also develop a client to communicate with the server, sending it commands and queries to:
- Add a song to the library
- Remove a song from the library
- Search for songs based on a search criteria

Much of the code is provided for this lab, available on GitHub [here (https://github.com/cpen333/lab5)](https://github.com/cpen333/lab5).

This lab contains much more code and content than previous labs.  Try to take it in stride.  We will only be modifying small sections of the application.  In the end, we should have a working client-server application which we can later refer to as an example.

Feel free to discuss approaches and solutions with your classmates, but labs are to be completed individually.  Each student is expected to be able to answer questions about the content, describe their work, and reproduce their code (or parts thereof).

{% include toc.html %}

## System Overview

Our *system* is going to have two applications: a server program that stores a library list of songs; and a client program that can connect to the server to run commands and search queries.  Multiple clients will be able to connect to the server and run commands simultaneously.

![client-server]({{site.url}}/assets/labs/socket/client_server.png)

The basic class diagram for the music library is as follows:

![music library class diagram]({{site.url}}/assets/labs/socket/music_library_class_diagram.png)

The library stores zero or more songs internally, and has public methods for adding, removing, and searching for songs by artist and/or title.

The server maintains the library in local memory.  It listens for client socket connections, and when one is established, communicates with the client by passing *messages* back and forth.  Messages from the client can be converted into commands that act on the library.  The server then sends values back to the client through response messages.

![message class diagram]({{site.url}}/assets/labs/socket/message_class_diagram.png)

Each type of message can be represented by a specialized `Message` class as in the above class hierarchy.  By constructing these message objects, we can better separate the main functionality of our program from the details of socket communication.

For sending messages back and forth, we will develop a *communication protocol*, allowing us to ensure that the message sent by one side is exactly the same as the message received on the other.  We will also develop an *application programming interface* (API) that will allow us to decipher the messages and translate them into commands and values.

The communication protocol and API will be encapsulated in a separate class, isolating the communication details from the rest of the program.  We provide an *interface*, `MusicLibraryApi`, that our client and server programs can use.  We will also provide a specific implementation, `JsonMusicLibraryApi`, that *implements* this interface using a specific protocol.

![api class diagram]({{site.url}}/assets/labs/socket/api_class_diagram.png)

Encapsulation and interfaces help us better separate code related to different functionalities, help us reduce dependencies, and allow us to easily replace components or implementation details in the future.  

### Source Files

The main system code, found in the `src/` directory is divided into several files:
- `Song.h`: class definition and implementation of a `Song` for populating our library
- `MusicLibrary.h`: class definition and implementation of the `MusicLibrary`
- `Message.h`: class definitions and implementations for the various message types
- `MusicLibraryApi.h`: the interface for our API that both the server and client can use for sending messages back and forth
- `JsonMusicLibraryApi.h`: an implementation of the `MusicLibraryApi` that uses the JSON format for encoding/decoding messages
- `JsonConverter.h`: a utility class for converting `Message`s and `Song`s to and from `JSON`-encoded objects
- `music_library_server.cpp`: the main server program
- `music_library_client.cpp`: the main client program

We also provide two files to help with testing in the `test/` subfolder:
- `TestException.h`: basic implementation of an exception to be thrown if a unit test fails
- `music_library_test.cpp`: unit test file for testing and debugging `MusicLibrary` on its own

Skim through the files to get a rough idea of what functionality is available.  You do not need to understand all the details of what is involved, only enough to get your bearings for when we modify parts of the program to make it functional.

## The Music Library

The music library class, `MusicLibrary` contains a *set* of songs in a private member variable.  The class has public member functions that allow us to add or remove songs to/from the library, as well as a method to search for songs by artist and title using [*regular expressions*](https://en.wikipedia.org/wiki/Regular_expression).  

We initialize the library by loading in a selection of songs from files in the `data/` subfolder.  These files contain lists of songs in JSON format downloaded from the [Billboard.com charts](http://www.billboard.com/charts).  Make sure this data folder exists in your *working directory* when running the server.

### Sets

*Sets* are very much like lists or vectors, except they are *unordered* -- i.e. the precise ordering of elements is irrelevant -- and all elements in the set must be unique.  If you attempt to add an element that already exists in the set, the set will remain unmodified.

In C\+\+, we can create a `std::set` by including the `<set>` header.  It is a templated class, allowing us to create a set of any type that supports comparisons with the `<` operator.  Alternatively, for custom objects, we can create a set by writing our own *comparator*, just as we have done for sorting in [Lab 3]({{site.url}}/labs/mutex/#sorting).  For our music library application, we have already implemented the comparison operator for `Song`:
```cpp
class Song {
  //...

  // less-than operator for comparisons, sort by artist then by song
  friend bool operator<(const Song& a, const Song& b) {
    if (a.artist < b.artist) {
      return true;
    } else if (a.artist > b.artist) {
      return false;
    }
    return a.title < b.title;
  }
}
```
When writing a comparison operator or a comparator, we need to ensure that it enforces a unique ordering of elements.  Here we first order by artist then by song title.

Elements can be added or removed from a set using
```cpp
std::set<Song> songs;
// ...
auto element = songs.insert(Song("Weezer","Buddy Holly"));
bool added = element.second;  // check if element was added
//...
size_t removed = songs.erase(Song("Weezer","Buddy Holly"));
```
The `insert(...)` method tries to add a new element to the set.  It returns a `std::pair` that holds an iterator to the corresponding element, and a `bool` indicating if the element added was new.  The `erase(...)` method tries to remove element from the set, returning the number of elements removed.

### Regular Expressions

*Regular expressions* are very powerful tools used to search for and replace specific patterns of text.  They allow for wildcards, optional characters, characters ranges, and much more.  

We will use regular expressions in our `MusicLibrary::find(...)` method to add greater flexibility to the search terms.  You do not need to know the fine details of regular expressions, or anything beyond a cursory knowledge of their existence.  However, they are incredibly useful, and most text editors and IDEs support them.

> **Personal anecdote:**
I once had a combination lock that I hadn't used for quite a while.  It was my favourite lock because it had a picture of a soccer ball painted on the front, so it was very easy to identify my locker.  At some point, I had written the combo in a text file, but that was years ago and I had no idea where I saved it.  All I *could* remember was that the file probably contained the word "soccer", and listed the combo in some way.
>
> Using the `grep` command-line tool, I searched through all text files on my computer for the regular expression pattern
>```
>[Ss]occer.*[0-9]{1,2}[-: ]+[0-9]{1,2}[-: ]+[0-9]{1,2}
>```
>- `[Ss]occer`:  the word soccer starting with either a lowercase or uppercase letter 's'
>- `.*`: any number of characters
>- `[0-9]{1,2}`: one or two digits in the range `0-9`
>- `[-: ]+`: one or more dashes, colons, or spaces used as a separator
>
>The above regular expression therefore is looking for the word "Soccer" or "soccer", followed by any amount of text, followed by a one-or-two digit number, followed by a separator, followed by another one or two digit number, followed by another separator, followed by a final one or two digit number.
>
>An hour later, `grep` spat this out:
>```
>D:\\backup\2005-12-03\ ... \private_notes.txt: >soccer: 14-5-32
>```
>I had found the combination!  But I have since lost the lock.

The most commonly used expressions are:
- `.`: matches any character
- `^`: matches the start of the line
- `$`: matches the end of the line
- `[abc]`: matches any single character in the brackets (i.e. 'a' or 'b' or 'c')
- `[^abc]`: matches any single character *except* those in the brackets
- `*`: matches zero or more of the previous character or expression
- `+`: matches one or more of the previous character or expression
- `(abc)|(def)`: matches the full string `"abc"` *or* `"def"`

#### Regular Expressions in C\+\+

The C\+\+ standard template library provides an implementation of regular expressions, which can be accessed by including the `<regex>` header.  We are going to use two elements of the library:
- `std::regex`: main class that *compiles* a regular expression from the provided pattern
- `bool std::regex_search(const std::string&, const std::regex&)`: searches a string to see if it matches the regular expression

For example, we can search a list of filenames for all files ending with `.json` or `.JSON` with:
```cpp
// .json or .JSON at the end of the line or string
std::regex json_ext("\.(json)|(JSON)$")

// search through list of filenames
for (std::string& filename : filenames) {
  if (std::regex_search(filename, json_ext)) {
    std::cout << "JSON file found: " << filename << std::endl;
  }
}
```
The `\.` in the above regular expression "escapes" the period so we actually search for filenames ending in ".json".  Otherwise the `.` would match any character.

### **Music Library Tasks**

We will modify the `MusicLibrary` class to complete the implementation of the `remove(...)` and `search(...)` functionalities.

1. Implement `bool MusicLibrary::remove(const Song& song)` so that it removes the provided song if it exists, returning true if the song was found in the set and removed successfully, or false if the song was not found.
2. Implement `size_t MusicLibrary::remove(const std::vector<Song>& songs)`, removing all songs matching those in the provided vector and returning the total number of songs removed.
3. Modify the `std::vector<Song> MusicLibrary::find(...)` method to include the *title* regular expression in the search (it currently only uses the artist regular expression).  For a result to be included, *both* artist and title regular expressions should match.
4. **(Optional)** Test the add, remove, and search functionalities using the `test/music_library_test.cpp` unit test program.  This could be useful if you wish to test your code before needing to complete the entire client-server system.  You may need to modify the unit tests to be more comprehensive.

## Sockets

To implement interprocess communication via sockets, we will use the CPEN 333 [course library](https://github.com/cpen333/library).  Make sure the `include/` folder is somewhere on the Include Path (or Header Search Path), so your compiler/IDE can find it.

### The Server

The server implementation, in `music_library_server.cpp`, creates a socket server that listens for connections on a specific port:
```cpp
// start server
cpen333::process::socket_server server(MUSIC_LIBRARY_SERVER_PORT);
server.open();
```
You may need to change this port number if you get an error about the port already being in use.

In client-server applications, the server generally goes through the following steps in a loop:
- *listen* for incoming connections on a specified port
- when a client attempts to connect, *accept* the connection
- create a new thread to service the connection and pass (or *move*) the socket to that thread
- *detach* the service thread so it can continue while the server waits for the next connection

In our application, the service function has the signature
```cpp
void service(MusicLibrary& lib, MusicLibraryApi&& api, int id);
```
This function takes in the music library by reference, the communication API handler by *r-value reference* (meaning it must be `std::move(..)`d into the function), and an integer ID for printing information to the console.

The service routine uses the API handler to wait for messages.  Once a message is returned,
```cpp
// receive message
std::unique_ptr<Message> msg = api.recvMessage();
```
we can check its type and process it accordingly.  The special class `std::unique_ptr<Message>` behaves exactly like a regular pointer to a `Message` object, except it cannot be copied (it can only be moved). We can dereference the pointer using the `*` operator, and access member variables and methods using the `->` operator.  Unique pointers are a form of "smart pointer" that automatically handle memory management using RAII.

Processing of adding songs, searching for songs, and goodbye messages are already implemented for you.  After each operation, a response is sent back to the client through the API handler.

### The Client

The client implementation, in `music_library_client.cpp`, creates a socket and tries to connect to the local machine and port that the server is running on:
```cpp
// start client
cpen333::process::socket socket("localhost", MUSIC_LIBRARY_SERVER_PORT);

// if we open the socket successfully, continue
if (socket.open()) {
  // ... do some stuff
}
```

Once open, we can send messages to the server and receive the corresponding responses.

The client is controlled through a simple text-based menu.  Sending and receiving of messages is handled by a `MusicLibraryApi` handler.  The "add" and "search" functionality is already provided for you.

### **Socket Tasks**

We are going to update the server to allow multiple simultaneous connections, and implement the "remove" functionality.

1. Modify the code in main server program to allow multiple simultaneous connections, all handled in separate detached threads.
2. Modify the server's `service(...)` routine to implement the "remove" functionality.  It should be similar to the "add" functionality.
3. Implement thread-safety using mutual exclusion in the server's `service(...)` routine.  Identify the critical sections related to modifying the music library and localize any locks in order to maximize concurrency.
4. Implement the client's `do_remove(...)` method to send the "remove" command to the server and handle its response.

## The Communication Protocol

To allow our programs to communicate over a network, we need to develop a *communication protocol*.  This is to ensure that content sent from one end of the communication channel can be perfectly re-assembled at the other end.  Remember, if clients/servers are compiled on different machines, the sizes of primitives like `int` may differ, so we cannot directly send raw binary information without explicitly specifying its composition.

The layout of our content will be as follows:

| indicator | size | message |
|--------|------|---------|
| 1-byte (0x55) | 4-byte big-endian integer | ASCII character string |

The first byte, `<indicator>` informs the receiver that the data being sent is of the type expected.  A leading byte like this is useful for two reasons:
- if we support multiple content types, or if we wish to change the communication protocol in the future, we can check the indicator value to see which type/protocol is being used and process the rest of the data accordingly
- if we receive a byte that doesn't match any indicator known to us, we know not to try to process the rest -- it would likely be garbage data anyways since it doesn't conform to our protocol

The next four bytes, `<size>`, encode the byte-size of the remainder of the content (i.e. the *message*), in big-endian format (most-significant byte first).  Big-endian, often referred to as *network byte order*, is the standard for sending raw data over a network.  Sending explicit sizes is useful so we know exactly how much data to expect before we try to decode the message.

The remaining bytes contain the `<message>` content.  In this case, the message consists of an ASCII string, so we know that each byte is a single character.

Now that both sender/receiver agree on how the content is being sent, byte-by-byte, we can be sure that the message will be consistent between the two ends of the communication channel.  The protocol is not concerned with the actual content of the message or how to interpret it, only how to send/receive it.

### The Message API

Messages in our music library application consist of actions to apply to the library on the server, and corresponding responses to send back to the client.  We support the following "commands" for the client to send to the server:

| Command | Description |
|---------|-------------|
| "add"   | add a song to the library |
| "remove" | remove a song from the library |
| "search" | search for songs matching artist and title expressions |
| "goodbye" | close the connection |

The add, remove and search commands have corresponding responses:

| Response | Description |
|---------|-------------|
| "add_response"   | reports success or failure of add |
| "remove_response" | reports success or failure of remove |
| "search_response" | returns results of search query |

To pass these commands and responses around, we will use [JSON](http://www.json.org)-encoded strings.  You do not need to know all the details of JSON, but you will need to know enough to modify existing code that uses it.

#### The JSON Format

JavaScript Object Notation ([JSON](http://www.json.org/)) has become the *de facto* standard format for passing around information in web applications.  It is a human-readable text-based format, is incredibly simple, and much more compact than other formats like XML.

The JSON format only describes a few basic types:
- **Objects:** surrounded by `{}`, act like a map containing attribute-value pairs separated by commas.  The difference between these objects and the maps we've seen in C\+\+ is that the keys are always strings, and the values don't all have to be the same type.
- **Arrays:** surrounded by `[]` and with elements separated by commas, can also hold a variety of value types.
- **Primitives:** strings, numbers, booleans, and null.

The following is an example JSON object representing an "add" message in our music library application:
```json
{
  "msg": "add",
  "song": {
    "artist": "Bob Dylan",
    "title": "Like a Rolling Stone"
  }
}
```
Any whitespace between entries, including newlines, is ignored.  This allows us to format JSON strings however we like.

#### JSON in C\+\+

To work with JSON in C\+\+ we will use the open-source library [JSON for Modern C\+\+](https://github.com/nlohmann/json), developed by Niels Lohmann.  It is a header-only library, meaning that all we have to do is include the header `<json.hpp>` and make sure our compiler can find it.  In the provided source files, this header can be found in the `include/` folder.

The library is rather simple to use.  The JSON class is contained within the `nlohmann` namespace.  For convenience, we will create an *alias* to this:
```cpp
using JSON = nlohmann::json;
```
This will allow us to use `JSON` as the class type in place of the original `nlohmann::json`.

To manually build a `JSON` object, we first create one using the default constructor, then assign values just like a map:
```cpp
  // song object
  JSON stone;
  stone["title"] = "Like a Rolling Stone";
  stone["artist"] = "Bob Dylan";

  // main object
  JSON add;
  add["msg"] = "add";
  add["song"] = stone;  // add JSON object as value

  std::cout << add.dump() << std::endl;
```

```
{"msg":"add","song":{"artist":"Bob Dylan","title":"Like a Rolling Stone"}}
```

JSON objects can be parsed from a string using the static function `JSON::parse()`, and converted back into a string using the member function `dump()`:
```cpp
std::string json_str = "[{\"artist\":\"Bob Dylan\",\"title\":\"Like a Rolling Stone\"},{\"artist\":\"Bob Marley\",\"title\":\"Counter Stone\"}]";

JSON songs = JSON::parse(json_str);   // parsed songs array
std::string songs_str = songs.dump(); // string (should match json_str)
```
We can iterate through JSON arrays using a range-based for-loop:
```cpp
// loop through array of songs
for (auto& jsong : songs) {
  std::string artist = jsong["artist"];
  std::string title = jsong["title"];
  std::cout << artist << " - " << title << std::endl;
}
```

#### Music Library JSON API

Within our JSON-encoded messages, songs are stored as JSON objects with two attributes:
<table>
  <thead>
    <tr>
      <th>JSON Object</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody style="font-size:0.9rem; padding:8px 8px 8px 8px;">
    <tr>
      <td>Song
        <ul class="compact">
          <li><code>"artist"</code>: string</li>
          <li><code>"title"</code>: string</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "artist": "Bob Dylan"
  "title":  "Like a Rolling Stone"
}</code></pre>
        </div>
      </td>
    </tr>
  </tbody>
</table>

The messages that can be passed back and forth between the client and server all contain the attribute `"msg"` which identifies the type of the message, followed by further message-specific attributes as listed below:
<table>
  <thead>
    <tr>
      <th>JSON Message</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody style="font-size:0.9rem; padding:8px 8px 8px 8px;">
    <tr>
      <td>AddMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"add"</code></li>
          <li><code>"song"</code>: Song</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg":  "add",
  "song": {
    "artist": "Bob Dylan",
    "title":  "Like a Rolling Stone"
  }
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>AddResponseMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"add_response"</code></li>
          <li><code>"status"</code>: <code>"OK"</code> or <code>"ERROR"</code></li>
          <li><code>"info"</code>: string</li>
          <li><code>"add"</code>: AddMessage</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg":    "add_response",
  "status": "ERROR",
  "info":   "Song already exists in library",
  "add": {
    "msg": "add",
    "song": {
      "artist": "Bob Dylan",
      "title":  "Like a Rolling Stone"
    }
  }
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>RemoveMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"remove"</code></li>
          <li><code>"song"</code>: Song</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg":  "remove",
  "song": {
    "artist": "Bob Dylan",
    "title":  "Like a Rolling Stone"
  }
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>RemoveResponseMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"remove_response"</code></li>
          <li><code>"status"</code>: <code>"OK"</code> or <code>"ERROR"</code></li>
          <li><code>"info"</code>: string</li>
          <li><code>"remove"</code>: RemoveMessage</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg":    "remove_response",
  "status": "OK",
  "info":   "",
  "remove": {
    "msg": "remove",
    "song": {
      "artist": "Bob Dylan",
      "title":  "Like a Rolling Stone"
    }
  }
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>SearchMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"search"</code></li>
          <li><code>"artist_regex"</code>: string</li>
          <li><code>"title_regex"</code>: string</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg": "search",
  "artist_regex": "Bob",
  "title_regex": "[Ss]tone"
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>SearchResponseMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"search_response"</code></li>
          <li><code>"status"</code>: <code>"OK"</code> or <code>"ERROR"</code></li>
          <li><code>"info"</code>: string</li>
          <li><code>"results"</code>: Song[ ]</li>
          <li><code>"search"</code>: SearchMessage</li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg":    "search_response",
  "status": "OK",
  "info":   "",
  "results": [
    {
      "artist": "Bob Dylan",
      "title":  "Like a Rolling Stone"
    },
    {
      "artist": "Bob Marley",
      "title": "Corner Stone"
    }
  ]
  "search": {
    "msg": "search",
    "artist_regex": "Bob",
    "title_regex": "[Ss]tone"
  }
}</code></pre>
        </div>
      </td>
    </tr>
    <tr>
      <td>GoodbyeMessage
        <ul class="compact">
          <li><code>"msg"</code>: <code>"goodbye"</code></li>
        </ul>
      </td>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>{
  "msg": "goodbye"
}</code></pre>
        </div>
      </td>
    </tr>
  </tbody>
</table>

Each message also has a corresponding C\+\+ class, found in `Message.h`.  Conversions between the JSON objects and classes are handled by the `JsonConverter` class.

### The API Interface

The `MusicLibraryApi` class provides an *interface* for our programs to send and receive messages by passing in or retrieving objects of type `Message`.  The API handler internally deals with all the communication details, such as conversion to and from JSON and reading/writing to the socket.  That way the rest of our program can be agnostic to the communication details.

```cpp
/**
 * Handles communication layer
 */
class MusicLibraryApi {
 public:
  /**
   * Sends a message
   * @param msg message to write
   * @return true if successful, false if error
   */
  virtual bool sendMessage(const Message& msg) = 0;

  /**
   * Reads a message from the socket.  
   * @return parsed message, nullptr if an error occurred
   */
  virtual std::unique_ptr<Message> recvMessage() = 0;

};
```

The interface itself does not have an implementation.  It is what we call a *pure abstract class* because all its methods are virtual and it does not have any member variables.  Interfaces are used to define a *contract* that other implementations must conform to.  We could have one of many *concrete* implementations, each of which could be used in place of the interface.  We provide one such implementation of `MusicLibraryApi`: `JsonMusicLibraryApi`.  We say that `JsonMusicLibraryApi` *implements* `MusicLibraryApi`.

The `JsonMusicLibraryApi` class has one constructor which takes ownership of a `socket` -- i.e. the `socket` must be passed by *r-value reference* using `std::move(...)`.  It then communicates with the client or server by converting our messages to and from JSON-encoded strings, and reading/writing them from/to the socket according to our communication protocol.

### **Communication Tasks**

We are going to complete the implementation of  `JsonMusicLibraryApi`, enabling decoding of message sizes and handling the "remove" functionality.

1. Modify `JsonConverter` in `JsonConverter.h` to implement conversion of messages of type `RemoveMessage` and `RemoveResponseMessage` to/from JSON.  Use these in the methods
    ```cpp
    class JsonConverter {
      //...
      static JSON toJSON(const Message &msg);
      //...
      static std::unique_ptr<Message> parseMessage(const JSON &jmsg);
    }
    ```
    so that our API implementation can handle them.
2. Modify `JsonMusicLibraryApi::recvJSON(...)` to decode the big-endian message size so we can read the remainder of the message.
3. Implement `JsonMusicLibraryApi::readString(...)` to populate a string by reading from the socket in (up to) 256-byte chunks.  Having a fixed buffer size and reading the data in chunks is a common practice when potentially reading arbitrarily long strings.

## Summary

We covered *a lot* of material in this lab:
- Sets
- Smart pointers
- Interfaces
- Regular Expressions
- JSON
- Socket client-server applications
- Communication protocols
- Application Programming Interfaces (APIs)

With these concepts, we can put together some pretty powerful applications.

The client-server music library program should now be fully functional, and easily extensible.  We should be able to connect to our server through the client, update the library by adding or removing songs, and run our regular-expression-based search queries.
