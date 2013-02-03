# Documentation

[AHKsock_Listen()](#ahksock_listensport-sfunction--false)  
[AHKsock_Connect()](#ahksock_connectsname-sport-sfunction)  
[AHKsock_Send()](#ahksock_sendisocket-ptrdata-ilength)  
[AHKsock_ForceSend()](#ahksock_forcesendisocket-ptrdata-ilength)  
[AHKsock_Close()](#ahksock_closeisocket---1-itimeout--5000)  
[AHKsock_GetAddrInfo()](#ahksock_getaddrinfoshostname-byref-siplist-bone--false)  
[AHKsock_GetNameInfo()](#ahksock_getnameinfosip-byref-shostname-sport--0-byref-sservice--)  
[AHKsock_SockOpt()](#ahksock_sockoptisocket-soption-ivalue---1)  
[AHKsock_Settings()](#ahksock_settingsssetting-svalue--)  
[AHKsock_ErrorHandler()](#ahksock_errorhandlersfunction--)  
[Notes on sockets and the structure of the event-handling function](#notes-on-sockets-and-the-structure-of-the-event-handling-function)  
[Notes on closing sockets and AHKsock_Close](#notes-on-closing-sockets-and-ahksock_close)  
[Notes on receiving and sending data](#notes-on-receiving-and-sending-data)  
[Notes on testing a stream processor](#notes-on-testing-a-stream-processor)  


### AHKsock_Listen(sPort, sFunction = False)

Tells AHKsock to listen on the port in sPort, and call the function in sFunction when events occur. If sPort is a port on
which AHKsock is already listening, the action taken depends on sFunction:  
* If sFunction is False, AHKsock will stop listening on the port in sPort.
* If sFunction is "()", AHKsock will return the name of the current function AHKsock calls when
a client connects on the port in sPort.
* If sFunction is a valid function, AHKsock will set that function as the new function to call
when a client connects on the port in sPort.

Returns blank on success. On failure, it returns one of the following positive integer:  
* 2: sFunction is not a valid function.
* 3: The WSAStartup() call failed. The error is in ErrorLevel.
* 4: The Winsock DLL does not support version 2.2.
* 5: The getaddrinfo() call failed. The error is in ErrorLevel.
* 6: The socket() call failed. The error is in ErrorLevel.
* 7: The bind() call failed. The error is in ErrorLevel.
* 8: The WSAAsyncSelect() call failed. The error is in ErrorLevel.
* 9: The listen() call failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

See the section titled "STRUCTURE OF THE EVENT-HANDLING FUNCTION AND MORE INFO ABOUT SOCKETS" for more info about how the
function in sFunction interacts with AHKsock.

### AHKsock_Connect(sName, sPort, sFunction)

Tells AHKsock to connect to the hostname or IP address in sName on the port in sPort, and call the function in sFunction
when events occur.

Although the function will return right away, the connection attempt will still be in progress. Once the connection attempt
is over, successful or not, sFunction will receive the CONNECTED event. Note that it is important that once AHKsock_Connect
returns, the current thread must stay (or soon after must become) interruptible so that sFunction can be called once the
connection attempt is over.

AHKsock_Connect can only be called again once the previous connection attempt is over. To check if AHKsock_Connect is ready
to make another connection attempt, you may keep polling it by calling AHKsock_Connect(0,0,0) until it returns False.

Returns blank on success. On failure, it returns one of the following positive integer:  
* 1: AHKsock_Connect is still processing a connection attempt. ErrorLevel contains the name and the port of that
connection attempt, separated by a tab.
* 2: sFunction is not a valid function.
* 3: The WSAStartup() call failed. The error is in ErrorLevel.
* 4: The Winsock DLL does not support version 2.2.
* 5: The getaddrinfo() call failed. The error is in ErrorLevel.
* 6: The socket() call failed. The error is in ErrorLevel.
* 7: The WSAAsyncSelect() call failed. The error is in ErrorLevel.
* 8: The connect() call failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

See the section titled "STRUCTURE OF THE EVENT-HANDLING FUNCTION AND MORE INFO ABOUT SOCKETS" for more info about how the
function in sFunction interacts with AHKsock.

### AHKsock_Send(iSocket, ptrData, iLength)

Sends the data of length iLength to which ptrData points to the connected socket in iSocket.

Returns the number of bytes sent on success. This can be less than the number requested to be sent in the iLength parameter,
i.e. between 1 and iLength. This would occur if no buffer space is available within the transport system to hold the data to
be transmitted, in which case the number of bytes sent can be between 1 and the requested length, depending on buffer
availability on both the client and server computers. On failure, it returns one of the following negative integer:  
* -1: WSAStartup hasn't been called yet.
* -2: Received WSAEWOULDBLOCK. This means that calling send() would have blocked the thread.
* -3: The send() call failed. The error is in ErrorLevel.
* -4: The socket specified in iSocket is not a valid socket. This means either that the socket in iSocket hasn't been
created using AHKsock_Connect or AHKsock_Listen, or that the socket has already been destroyed.
* -5: The socket specified in iSocket is not cleared for sending. You haven't waited for the SEND event before calling,
either ever, or not since you last received WSAEWOULDBLOCK.

You may start sending data to the connected socket in iSocket only after the socket's associated function receives the first
SEND event. Upon receiving the event, you may keep calling AHKsock_Send to send data until you receive the error -2, at
which point you must wait once again until you receive another SEND event before sending more data. Not waiting for the SEND
event results in receiving error -5 when calling AHKsock_Send.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_ForceSend(iSocket, ptrData, iLength)

This function is exactly the same as AHKsock_Send, but with three differences:  
* If only part of the data could be sent, it will automatically keep trying to send the remaining part.
* If it receives WSAEWOULDBLOCK, it will wait for the socket's SEND event and try sending the data again.
* If the data buffer to send is larger than the socket's send buffer size, it will automatically send the data in
smaller chunks in order to avoid a performance hit. See http://support.microsoft.com/kb/823764 for more info.

Therefore, AHKsock_ForceSend will return only when all the data has been sent. Because this function relies on waiting for
the socket's SEND event before continuing to send data, it cannot be called in a critical thread. Also, for the same reason,
it cannot be called from a socket's associated function (not specifically iSocket's associated function, but any socket's
associated function).

Another limitation to consider when choosing between AHKsock_Send and AHKsock_ForceSend is that AHKsock_ForceSend will not
return until all the data has been sent (unless an error occurs). Although the script will still be responsive (new threads
will still be able to launch), the thread from which it was called will not resume until it returns. Therefore, if sending
a large amount of data, you should either use AHKsock_Send, or use AHKsock_ForceSend by feeding it smaller pieces of the
data, allowing you to update the GUI if necessary (e.g. a progress bar).

Returns blank on success, which means that all the data to which ptrData points of length iLength has been sent. On failure,
it returns one of the following negative integer:  
* -1: WSAStartup hasn't been called yet.
* -3: The send() call failed. The error is in ErrorLevel.
* -4: The socket specified in iSocket is not a valid socket. This means either that the socket in iSocket hasn't been
created using AHKsock_Connect or AHKsock_Listen, or that the socket has already been destroyed.
* -5: The current thread is critical.
* -6: The getsockopt() call failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_Close(iSocket = -1, iTimeout = 5000)

Closes the socket in iSocket. If no socket is specified, AHKsock_Close will close all the sockets on record, as well as
terminate use of the Winsock 2 DLL (by calling WSACleanup). If graceful shutdown cannot be attained after the timeout
specified in iTimeout (in milliseconds), it will perform a hard shutdown before calling WSACleanup to free resources. See
the section titled "NOTES ON CLOSING SOCKETS AND AHKsock_Close" for more information.

Returns blank on success. On failure, it returns one of the following positive integer:  
* 1: The shutdown() call failed. The error is in ErrorLevel. AHKsock_Close forcefully closed the socket and freed the
associated resources.

Note that when AHKsock_Close is called with no socket specified, it will never return an error.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_GetAddrInfo(sHostName, ByRef sIPList, bOne = False)

Retrieves the list of IP addresses that correspond to the hostname in sHostName. The list is contained in sIPList, delimited
by newline characters. If bOne is True, only one IP (the first one) will be returned.

Returns blank on success. On failure, it returns one of the following positive integer:  
* 1: The WSAStartup() call failed. The error is in ErrorLevel.
* 2: The Winsock DLL does not support version 2.2.
* 3: Received WSAHOST_NOT_FOUND. No such host is known.
* 4: The getaddrinfo() call failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_GetNameInfo(sIP, ByRef sHostName, sPort = 0, ByRef sService = "")

Retrieves the hostname that corresponds to the IP address in sIP. If a port in sPort is supplied, it also retrieves the
service that corresponds to the port in sPort.

Returns blank on success. On failure, it returns on of the following positive integer:  
* 1: The WSAStartup() call failed. The error is in ErrorLevel.
* 2: The Winsock DLL does not support version 2.2.
* 3: The IP address supplied in sIP is invalid.
* 4: The getnameinfo() call failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_SockOpt(iSocket, sOption, iValue = -1)

Retrieves or sets a socket option. Supported options are:  
* SO_KEEPALIVE: Enable/Disable sending keep-alives. iValue must be True/False to enable/disable. Disabled by default.
* SO_SNDBUF:    Total buffer space reserved for sends. Set iValue to 0 to completely disable the buffer. Default is 8 KB.
* SO_RCVBUF:    Total buffer space reserved for receives. Default is 8 KB.
* TCP_NODELAY:  Enable/Disable the Nagle algorithm for send coalescing. Set iValue to True to disable the Nagle algorithm,
set iValue to False to enable the Nagle algorithm, which is the default.

It is usually best to leave these options to their default (especially the Nagle algorithm). Only change them if you
understand the consequences. See MSDN for more information on those options.

If iValue is specified, it sets the option to iValue and returns blank on success. If iValue is left as -1, it returns the
value of the option specified. On failure, it returns one of the following negative integer:  
* -1: The getsockopt() failed. The error is in ErrorLevel.
* -2: The setsockopt() failed. The error is in ErrorLevel.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### AHKsock_Settings(sSetting, sValue = "")

Changes the AHKsock setting in sSetting to sValue. If sValue is blank, the current value for that setting is returned. If
sValue is the word "Reset", the setting is restored to its default value. The possible settings are:  
* Message: Determines the Windows message numbers used to monitor network events. The message number in iMessage and the
next number will be used. Default value is 0x8000. For example, calling AHKsock_Settings("Message", 0x8005)
will cause AHKsock to use 0x8005 and 0x8006 to monitor network events.
* Buffer:  Determines the size of the buffer (in bytes) used when receiving data. This is thus the maximum size of bData
when the RECEIVED event is raised. If the data received is more than the buffer size, multiple recv() calls
(and thus multiple RECEIVED events) will be needed. Note that you shouldn't use this setting as a means of
delimiting frames. See the "NOTES ON RECEIVING AND SENDING DATA" section for more information about receiving
and sending data. Default value is 64 KB, which is the maximum for TCP.

If you do call AHKsock_Settings to change the values from their default ones, it is best to do so at the beginning of the
script. The message number used cannot be changed as long as there are active connections.

### AHKsock_ErrorHandler(sFunction = """")

Sets the function in sFunction to be the new error handler. If sFunction is left at its default value, it returns the name
of the current error handling function.

An error-handling function is optional, but may be useful when troubleshooting applications. The function will be called
anytime there is an error that arises in a thread which wasn't called by the user but by the receival of a Windows message
which was registered using OnMessage.

The function in sFunction must be of the following format:
MyErrorHandler(iError, iSocket)

The possible values for iError are:  
*  1: The connect() call failed. The error is in ErrorLevel.
*  2: The WSAAsyncSelect() call failed. The error is in ErrorLevel.
*  3: The socket() call failed. The error is in ErrorLevel.
*  4: The WSAAsyncSelect() call failed. The error is in ErrorLevel.
*  5: The connect() call failed. The error is in ErrorLevel.
*  6: FD_READ event received with an error. The error is in ErrorLevel. The socket is in iSocket.
*  7: The recv() call failed. The error is in ErrorLevel. The socket is in iSocket.
*  8: FD_WRITE event received with an error. The error is in ErrorLevel. The socket is in iSocket.
*  9: FD_ACCEPT event received with an error. The error is in ErrorLevel. The socket is in iSocket.
* 10: The accept() call failed. The error is in ErrorLevel. The listening socket is in iSocket.
* 11: The WSAAsyncSelect() call failed. The error is in ErrorLevel. The listening socket is in iSocket.
* 12: The listen() call failed. The error is in ErrorLevel. The listening socket is in iSocket.
* 13: The shutdown() call failed. The error is in ErrorLevel. The socket is in iSocket.

For the failures which affect ErrorLevel, ErrorLevel will contain either the reason the DllCall itself failed (ie. -1, -2,
An, etc... as laid out in the AHK docs for DllCall) or the Windows Sockets Error Code as defined at:
http://msdn.microsoft.com/en-us/library/ms740668

### Notes on sockets and the structure of the event-handling function

The functions used in the sFunction parameter of AHKsock_Listen and AHKsock_Connect must be of the following format:

MyFunction(sEvent, iSocket = 0, sName = 0, sAddr = 0, sPort = 0, ByRef bData = 0, bDataLength = 0)

The variable sEvent contains the event for which MyFunction was called. The event raised is associated with one and only one
socket; the one in iSocket. The meaning of the possible events that can occur depend on the type of socket involved. AHKsock
deals with three different types of sockets:  
* Listening sockets: These sockets are created by a call to AHKsock_Listen. All they do is wait for clients to request
a connection. These sockets will never appear as the iSocket parameter because requests for connections are
immediately accepted, and MyFunction immediately receives the ACCEPTED event with iSocket set to the accepted socket.
* Accepted sockets: These sockets are created once a listening socket receives an incoming connection attempt from a
client and accepts it. They are thus the sockets that servers use to communicate with clients.
* Connected sockets: These sockets are created by a successful call to AHKsock_Connect. These are the sockets that
clients use to communicate with servers.

More info about sockets:  
* You may have multiple client sockets connecting to the same listening socket (ie. on the same port).
* You may have multiple listening sockets for different ports.
* You cannot have more than one listening socket for the same port (or you will receive a bind() error).
* Every single connection between a client and a server will have its own client socket on the client side, and its own
server (accepted) socket on the server side.

For all of the events that the event-handling function receives,  
* sEvent contains the event that occurred (as described below),
* iSocket contains the socket on which the event occurred,
* sName contains a value which depends on the type of socket in iSocket:
  * If the socket is an accepted socket, sName is empty.
  * If the socket is a connected socket, sName is the same value as the sName parameter that was used when
AHKsock_Connect was called to create the socket. Since AHKsock_Connect accepts both hostnames and IP addresses,
sName may contain either.
* sAddr contains the IP address of the socket's endpoint (i.e. the peer's IP address). This means that if the socket in
iSocket is an accepted socket, sAddr contains the IP address of the client. Conversely, if it is a connected socket,
sAddr contains the server's IP.
* sPort contains the server port on which the connection was accepted.

Obviously, if your script only calls AHKsock_Listen (acting as a server) or AHKsock_Connect (acting as a client) you don't
need to check if the socket in iSocket is an accepted socket or a connected socket, since it can only be one or the other.
But if you do call both AHKsock_Listen and AHKsock_Connect with both of them using the same function (e.g. MyFunction), then
you will need to check what type of socket iSocket is by checking the sName parameter.

Of course, it would be easier to simply have two different functions, for example, MyFunction1 and MyFunction2, with one
handling the server part and the other handling the client part so that you don't need to check what type of socket iSocket
is when each function is called. However, this might not be necessary if both server and client are "symmetrical" (i.e. the
conversation doesn't actually change whether or not we're on the server side or the client side). See Example 3 for an
example of this, where only one function is used for both server and client sockets.

The variable sEvent can be one of the following values if iSocket is an accepted socket:  
<table>
<tr><td>sEvent</td><td>Event Description</td></tr>
<tr><td>ACCEPTED</td><td>A client connection was accepted (see the "Listening sockets" section above for more details).</td></tr>
<tr><td>CONNECTED</td><td>&lt;Does not occur on accepted sockets&gt;</td></tr>
<tr><td>DISCONNECTED</td><td>The client disconnected (see AHKsock_Close for more details).</td></tr>
<tr><td>SEND</td><td>You may now send data to the client (see AHKsock_Send for more details).</td></tr>
<tr><td>RECEIVED</td><td>You received data from the client. The data received is in bData and the length is in bDataLength.</td></tr>
<tr><td>SENDLAST</td><td>The client is disconnecting. This is your last chance to send data to it. Once this function returns, disconnection will occur. This event only occurs on the side which did not initiate shutdown (see AHKsock_Close for more details).</td></tr>
</table>

The variable sEvent can be one of the following values if iSocket is a connected socket:  
<table>
<tr><td>sEvent</td><td>Event Description</td></tr>
<tr><td>ACCEPTED</td><td>&lt;Does not occur on connected sockets&gt;</td></tr>
<tr><td>CONNECTED</td><td>The connection attempt initiated by calling AHKsock_Connect has completed (see AHKsock_Connect for more details). If it was successful, iSocket will equal the client socket. If it failed, iSocket will equal -1. To get the error code that the failure returned, set an error handling function with AHKsock_ErrorHandler, and read ErrorLevel when iError is equal to 1.</td></tr>
<tr><td>DISCONNECTED</td><td>The server disconnected (see AHKsock_Close for more details).</td></tr>
<tr><td>SEND</td><td>You may now send data to the server (see AHKsock_Send for more details).</td></tr>
<tr><td>RECEIVED</td><td>You received data from the server. The data received is in bData and the length is in bDataLength.</td></tr>
<tr><td>SENDLAST</td><td>The server is disconnecting. This is your last chance to send data to it. Once this function returns, disconnection will occur. This event only occurs on the side which did not initiate shutdown (see AHKsock_Close for more details).</td></tr>
</table>

More information: The event-handling functions described in here are always called with the Critical setting on. This is
necessary in order to ensure proper processing of messages. Note that as long as the event-handling function does not
return, AHKsock cannot process other network messages. Although messages are buffered, smooth operation might suffer when
letting the function run for longer than it should.

### Notes on closing sockets and AHKsock_Close

There are a few things to note about the AHKsock_Close function. The most important one is this: because the OnExit
subroutine cannot be made interruptible if running due to a call to Exit/ExitApp, AHKsock_Close will not be able to execute
a graceful shutdown if it is called from there. 

A graceful shutdown refers to the proper way of closing a TCP connection. It consists of an exchange of special TCP messages
between the two endpoints to acknowledge that the connection is about to close. It also fires the SENDLAST event in the
socket's associated function to notify that this is the last chance it will have to send data before disconnection. Note
that listening sockets cannot (and therefore do not need to) be gracefully shutdown as it is not an end-to-end connection.
(In practice, you will never have to manually call AHKsock_Close on a listening socket because you do not have access to
them. The socket is closed when you stop listening by calling AHKsock_Listen with no specified value for the second
parameter.)

In order to allow the socket(s) connection(s) to gracefully shutdown (which is always preferable), AHKsock_Close must be
called in a thread which is, or can be made, interruptible. If it is called with a specified socket in iSocket, it will
initiate a graceful shutdown for that socket alone. If it is called with no socket specified, it will initiate a graceful
shutdown for all connected/accepted sockets, and once done, deregister itself from the Windows Sockets implementation and
allow the implementation to free any resources allocated for Winsock (by calling WSACleanup). In that case, if any
subsequent AHKsock function is called, Winsock will automatically be restarted.

Therefore, before exiting your application, AHKsock_Close must be called at least once with no socket specified in order to
free Winsock resources. This can be done in the OnExit subroutine, either if you do not wish to perform a graceful shutdown
(which is not recommended), or if you have already gracefully shutdown all the sockets individually before calling
Exit/ExitApp. Of course, it doesn't have to be done in the OnExit subroutine and can be done anytime before (which is the
recommended method because AHKsock will automatically gracefully shutdown all the sockets on record).

This behaviour has a few repercussions on your application's design. If the only way for the user to terminate your
application is through AHK's default Exit menu item in the tray menu, then upon selecting the Exit menu item, the OnExit sub
will fire, and your application will not have a chance to gracefully shutdown connected sockets. One way around this is to
add your own menu item which will in turn call AHKsock_Close with no socket specified before calling ExitApp to enter the
OnExit sub. See AHKsock Example 1 for an example of this.

This is how the graceful shutdown process occurs between two connected peers:  
* a> Once one of the peers (it may be the server of the client) is done sending all its data, it calls AHKsock_Close to
shutdown the socket. (It is not a good idea to have the last peer receiving data call AHKsock_Close. This will result
in AHKsock_Send errors on the other peer if more data needs to be sent.) In the next steps, we refer to the peer that
first calls AHKsock_Close as the invoker, and the other peer simply as the peer.
* b> The peer receives the invoker's intention to close the connection and is given one last chance to send any remaining
data. This is when the peer's socket's associated function receives the SENDLAST event.
* c> Once the peer is done sending any remaining data (if any), it also calls AHKsock_Close on that same socket to shut it
down, and then close the socket for good. This happens once the peer's function that received the SENDLAST event
returns from the event. At this point, the peer's socket's associated function receives the DISCONNECTED event.
* d> This happens in parallel with c>. After the invoker receives the peer's final data (if any), as well as notice that
the peer has also called AHKsock_Close on the socket, the invoker finally also closes the socket for good. At this
point, the socket's associated function also receives the DISCONNECTED event.

When AHKsock_Close is called with no socket specified, this process occurs (in parallel) for every connected socket on
record.

### Notes on receiving and sending data

It's important to understand that AHKsock uses the TCP protocol, which is a stream protocol. This means that the data
received comes as a stream, with no apparent boundaries (i.e. frames or packets). For example, if a peer sends you a string,
it's possible that half the string is received in one RECEIVED event and the other half is received in the next. Of course,
the smaller the string, the less likely this happens. Conversely, the larger the string, the more likely this will occur.

Similarly, calling AHKsock_Send will not necessarily send the data right away. If multiple AHKsock_Send calls are issued,
Winsock might, under certain conditions, wait and accumulate data to send before sending it all at once. This process is
called coalescing. For example, if you send two strings to your peer by using two individual AHKsock_Send calls, the peer
will not necessarily receive two consecutive RECEIVED events for each string, but might instead receive both strings through
a single RECEIVED event.

One efficient method of receiving data as frames is to use length-prefixing. Length-prefixing means that before sending a
frame of variable length to your peer, you first tell it how many bytes will be in the frame. This way, your peer can
divide the received data into frames that can be individually processed. If it received less than a frame, it can store the
received data and wait for the remaining data to arrive before processing the completed frame with the length specified.
This technique is used in in AHKsock Example 3, where peers send each other strings by first declaring how long the string
will be (see the StreamProcessor function of Example 3).

### Notes on testing a stream processor

As you write applications that use length-prefixing as described above, you might find it hard to test their ability to
properly cut up and/or put together the data into frames when testing them on the same machine or on a LAN (because the
latency is too low and it is thus harder to stress the connection).

In this case, what you can do to properly test them is to uncomment the comment block in AHKsock_Send, which will sometimes
purposely fail to send part of the data requested. This will allow you to simulate what could happen on a connection going
through the Internet. You may change the probability of failure by changing the number in the If statement.

If your application can still work after uncommenting the block, then it is a sign that it is properly handling frames split
across multiple RECEIVED events. This would also demonstrate your application's ability to cope with partially sent data.
