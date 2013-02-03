AHKsock - A simple AHK implementation of Winsock (TCP/IP)
=========================================================

AHKsock is a high-level wrapper which I have written to facilitate the use of
the Winsock APIs in AHK. It will allow you to create clients and servers that
can communicate with each other, purely written in AHK! Forum thread is [here]
(http://www.autohotkey.com/board/topic/53827-ahksock-a-simple-ahk-implementation
-of-winsock-tcpip/).

AHKsock includes three [examples](examples/):
* In Example 1, the server listens for clients and sends some data to any client
that connects. This is a very simple example to show the basics of AHKsock.
* In Example 2, the server sends a file to any client that connects. There are
three types of servers. The first one uses the TransmitFile() function provided
by Windows while the second one reads from the file itself and transfer to a
single client at a time. The third type can serve multiple clients
simultaneously.
* In Example 3, two peers connect to each other to start chatting. This is a
great example of a symmetrical connection. Whichever is started first becomes
the server, while the second becomes the client. However, they both behave in
exactly the same way once connected to each other.
* In Example 4, we can look up IPs from hostnames and vice-versa.

Additional resources on Winsock and sockets in general:
* [Winsock Programmer's FAQ](http://tangentsoft.net/wskfaq/)
* [WinSock Development Information](http://sockets.com/)
* [MSDN Windows Sockets 2 Reference](http://msdn.microsoft.com/en-us/library/ms740673)
