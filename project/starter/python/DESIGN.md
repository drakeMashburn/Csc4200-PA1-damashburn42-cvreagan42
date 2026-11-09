# PA1 Design — ETP/1 Client and Server

_One to two pages. Short, specific answers beat long general ones. Every section
should be true of the code you submit. Delete these italic prompts when you write._

## 1. Architecture

_Your modules and what calls what. A small diagram (ASCII is fine) or a list:
which file holds the protocol (encode/decode), which holds the transport, which
holds the server's session logic, which holds the client._

* Protocol: etp.py
* Transport: transport_tcp.py
* Server's Session Logic: server.py
* Client's Session Logic: client.py

The file client.py calls etp.py and transport.py while server.py calls etp.py and transport_tcp.py

* client.py
    etp.py
        * encode()
        * read_message()
    transport.py
        * send_all()
        * connect()

* server.py
    etp.py
        * encode()
        * read_message()
        * valid_name()
    transport_tcp.py
        * close()
        * send_all()
        * listen()
        * accept()
## 2. The transport seam
=======
## 2. The transport seam (drake)

_Which file is your transport? List the six operations as you implemented them.
Confirm that no other file calls the socket API. In two or three sentences: what
would you have to change, and what would stay the same, when you swap TCP for your
PA2 transport in the final project?

a: transport_tcp.py

b: listen, accept, connect, send all, recv exact, close.

c: as of 11.9.2026 at 8:24am, I can confirm that no other files besides transport.tcp.py import the soecket model

d: From our minmimal understanding, it would appear that a lot of the tcp security and checks come from the socket moduel. This would imply that we would have to creat our own barebones socket moduel for prog 2. All it can do is send something, receive something, and connect. 
Our 6 functionalities would remain the same but there would be more to the client and server end to ensure that integrity is maintained



## 3. Framing

_How does your code read one ETP message from a byte stream? Where and how do you
handle a header that arrives in pieces, or two messages that arrive in one read?
Where do you check VERSION and LENGTH, and why before reading the payload?_

The code reads an ETP message in two stages. First, it reads exactly 8 bytes, which is the fixed-size
ETP header. After unpacking the header, it reads exactly the number of payload bytes specified in the field.

If the header arrives in multiple TCP segments, recv_exact() continues calling recv() until all 8 header bytes have been collected. It uses the LENGTH field from the ETP header to determine exactly how many payload bytes belong to the current message. 

Any additional bytes belong to the next message and are processed by a later read. VERSION is checked to verify that the message uses the expected protocol format. LENGTH is checked before reading the payload because it determines how many bytes must be received. The receiver must know the payload size before it can reconstruct the message correctly.

## 4. Concurrency

_Threads, processes or an event loop? What happens to everyone else when one
client stalls half-way through a header? Is any state shared between sessions
(for example, two clients PUT and GET the same file at once), and how is it
protected?_

The server uses a multi-threaded connection model. If one client stalls halfway through sending an ETP header, only that client's worker thread blocks. Other clients continue to be served normally because each connection is handled by its own thread created with threading.

These are not shared between sessions and all threads share the same access to the server file's directory. The implementation does not use explicit locking to synchronize file access. Protection relies on the operating system's file handling rather than 
application-level synchronization.

## 5. Errors and disconnects
=======
## 5. Errors and disconnects (Drake)

_How do fatal and non-fatal errors differ in your code? What happens when a client
disconnects in the middle of a GET of a large file? (C: how did you deal with
SIGPIPE?) How does your client choose its exit code?_

for client side error handling, any Fatal error will simply return and end the program (a return -1 type situation)
for server side handling, any fatal error will send a blank payload with the message type of etp.typeError and close the connection

in terms of non fatal error handling,
Server side. the entire session is done through 2 while loops. A while loop strictly for transitioning from await hello to hello, and the rest of the session, any non fatal error will just result in a (continue) clause being hit, starting another cycle of the loop. Of course, the server will still send
an error message to the client.
Client side, the runs function returns an error and the main function returns and closes the connect.

## 6. Threat model

_One paragraph. Who could attack an ETP/1 server, and how? Name one attack your
server resists (and how), and one it does not. Chapter 5 §5.9 and the
specification's Security Considerations are good starting points._

Anyone who could connect to the ETP server could attempt to send malformed or malicious protocol messages. For example,
an attacker could send an invalid ETP header or a malformed LENGTH field in an attempt to confuse the receiver. One attack the server resists is the server validates the VERSION field when reading ETP messages. If a client sends a message using an unsupported protocol version, the message is rejected rather than processed. The server is vulnerable to a denial-of-service attack in which a client connects and slowly sends data or stalls while transmitting a header. The implementation does not currently enforce connection timeouts or thread limits.

## 7. Testing
=======
## 7. Testing (drake)

_What did you test beyond `etp_selfcheck.py`? How? Name one bug your testing
found, how you found it, and the fix._

No other testing was done besides etp_selfcheck.py. Most of the bugs came from a misunderstanding of our errors. They are constant byte literals. A lot of error handling came from the following logic
try:
    if(errorCase == true)
        raise etp.ERROR
except ERROR:
    handle

You cannot raise a byte literal like that. We essentially had to reimplement error handling from the ground up using if elif blocks. (and perform try except blocks using errors actually defined in standard libraries)
