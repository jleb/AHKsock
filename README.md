AHKsock - A simple AHK implementation of Winsock (TCP/IP)
=========================================================

AHKsock is a high-level wrapper which I have written to facilitate the use of
the Winsock APIs in AHK. It will allow you to create clients and servers that
can communicate with each other, purely written in AHK! Forum thread is [here]
(http://www.autohotkey.com/board/topic/53827-ahksock-a-simple-ahk-implementation
-of-winsock-tcpip/).

AHKsock includes three [examples](examples/):
* In Example 1, the server listens for clients and sends some data to any client
that connects.
* In Example 2, the server sends a file to any client that connects. There are
three types of servers which support either a single client or multiple clients.
* In Example 3, two peers connect to each other to start chatting.
* In Example 4, we can look up IPs or hostnames from the other
