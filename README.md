Download Link: https://assignmentchef.com/product/solved-cpsc471-project1-challenges-of-protocol-design
<br>



<ol>

 <li><strong>To understand </strong>the challenges of protocol design.</li>

 <li><strong>To discover and appreciate </strong>the challenges of developing complex, real-world network applications.</li>

 <li><strong>Make sense of </strong>real-world sockets programming APIs.</li>

 <li><strong>To utilize </strong>a sockets programming API to construct simplified FTP server and client applications.</li>

</ol>

<h1>Overview</h1>

In this assignment, you will implement (simplified) FTP server and FTP client. The client shall connect to the server and support uploading and downloading of files to/from server. Before continuing, <strong>please read the <em>Preliminaries </em>section, below</strong>. It will save y o u hours o f frustration.

<h1>The Preliminaries</h1>

In class we covered the basics of programming with TCP sockets. Please consult the slides covering the Application Layer, if you need a quick refresher. The purpose of this section is to list some tips which can make this assignment <strong>orders of magnitude </strong>easier and more enjoyable.

<strong>Peculiarities of send() and recv() and How to Get Them Right</strong>

Consider codes for client and server codes below.

<ul>

 <li><em># Server code</em></li>

 <li><strong>from </strong>s o c k e t <strong>import </strong></li>

 <li><em># The port on which to l is ten</em></li>

 <li>s e rve r Po r t = 12000</li>

 <li><em># Create a TCP socket</em></li>

 <li>s e r v e r S o c k e t = s o c k e t ( AF INET ,SOCK STREAM) 9</li>

 <li><em># Bind the socket to the port</em></li>

 <li>s e r v e r S o c k e t . bind ( ( ’ ’ , s e r v e r P o r t ) ) 12</li>

 <li><em># Start listening for incoming connections</em></li>

 <li>s e r v e r S o c k e t . l i s t e n ( 1 )</li>

</ul>

15

16         <strong>print  </strong>”The  <u> </u>s e r v e r  <u> </u> i s  <u> r</u>eady  t<u>o</u>             r e c e i v e ”

17

<ul>

 <li><em># The buffer to s tore the received data</em></li>

 <li>data = ””</li>

</ul>

20

21 <em># Forever accept incoming connections</em> 22 <strong>while </strong>1 :

<ul>

 <li><em># Accept a connection ; get client ’ s socket</em></li>

 <li>connection Socket , addr = s e r v e r S o c k e t . accept ( ) 25</li>

 <li><em># Receive whatever the newly connected client has to send</em></li>

 <li>data = connection Socket . recv ( 40 ) 28</li>

</ul>

29                             <strong>print        </strong>data

30

<ul>

 <li><em># Close the socket</em></li>

 <li>connection Socket . c l o se ( )</li>

</ul>




<ul>

 <li><em># Client code</em></li>

 <li><strong>from </strong>s o c k e t <strong>import </strong></li>

</ul>

3

4 <em># Name and port number of the serve r to</em> 5 <em># which want to connect .</em>

<ul>

 <li>serverName = ” e c s . f u l l e r t o n . edu”</li>

 <li>s e rv e r Po r t = 12000 8</li>

 <li><em># Create a s ocke t</em></li>

 <li>c l i e n t S o c k e t = s o ck e t ( AF INET , SOCK STREAM) 11</li>

 <li><em># Connect to the serve r</em></li>

 <li>c l i e n t S o c k e t . connect ( ( serverName , s e rv e r Po r t ) ) 14</li>

 <li><em># A s t ring we want to send to the serve r</em></li>

 <li>data =  ”Hel l o   world !   <u>T</u>his   i<u> s</u>    a   <u>v</u>ery   l<u>o</u>ng   s t<u> r</u> i n g. ” 17 18 <em># Send that string!</em></li>

</ul>

19 c l i e n t S o c k e t . send ( data )

20

<ul>

 <li><em># Close the socket</em></li>

 <li>c l i e n t S o c k e t . c l o s e ( )</li>

</ul>




In the code above, the client connects to the server and sends string “Hello world! This is a very long string.” Both codes are syntactically correct. Whenever the client connects to the server, we would expect the server to print out the 40 character string sent by the client. In practice, this may not always be the case. The potential problems are outlined below:

<ul>

 <li><strong>Problem 1: send(data) is not guaranteed to send </strong><strong>all bytes of the data: </strong>As we learned in class, each socket has an associated send buffer. The buffer stores application data ready to be sent off. Whenever you call the send(), behind  the  scenes,  the operating system copies the data from the buffer in your program into the send buffer of the socket. Your operating system will then take the data in the send buffer, convert it into a TCP segment, and push the segment down the protocol stack. send()<strong>,  however,  will  return immediately after the socket’s network  buffer  has  been  filled</strong>.  This  means  that in the client code above,  if  the  data  string  was  large,  (i.e.  Larger than the network buffer), the  send() would return before  sending all bytes of</li>

</ul>




<ul>

 <li><strong>Problem 2: data = connectionSocket.recv(40) is not guaranteed to receive </strong><strong>all 40 bytes: </strong>This can happen even if the sender has sent all 40 bytes. Before delving into this problem, lets review the behavior of recv(). Function recv() will block until either a) the other side has closed their socket, or b) some data has been received.</li>

</ul>




Case a) is generally not a problem. It can be easily detected by adding line if not data: immediately after data = connectionSocket.recv(40) in the receiver (which in the above example is the server). The test will evaluate to true if the other side has closed their socket.




In Case b), the variable data will contain the received data. However, data may not be 40

bytes in size (again, even if the sender sent 40 bytes). The reasons can be as follows:




<ol>

 <li><strong>Not all data may arrive at the same time: </strong>Recall that Internet is a packet switched network. Big chunks of data are split into multiple packets. Some of these packets may arrive at the receiver faster than others. Hence, it is possible that 20 bytes of the string arrive at the receiver, while 20 more are still on their way. recv(), however, will return as soon as it gets the first 20 bytes.</li>

</ol>




<ol start="2">

 <li><strong>recv() returns after emptying the receive buffer of the  socket:  </strong>as  we  learned  in class, each socket also has a receive buffer. That buffer stores all arriving data that is ready  to  be  retrieved  by  the    recv()  <strong>returns  after  emptying  the  receive</strong> <strong>buffer</strong>. Because the receive buffer  may  not  contain  all  data  sent  by  the  sender when the  recv()  is  called,  recv()  will  return  before  having  received  the  specified  number of bytes.</li>

</ol>




So how do we cope with the above problems? The answer is that it is up to you, the application developer, to ensure that all bytes are sent and received. This can be done by calling send() and recv() inside the  loop,  until all data has been  sent/received. First,  lets fix  our  client  to  ensure that it sends all of the specified bytes:




<ul>

 <li><em># Clie n t code</em></li>

 <li><strong>from </strong>s o c k e t <strong>import </strong></li>

</ul>

3

4 <em># Name and port number of the serve r to</em> 5 <em># which want to connect .</em>

<ul>

 <li>serverName = e c s . f u l l e r t o n . e d u</li>

 <li>s e rv e r Po r t = 12000</li>

</ul>

8

<ul>

 <li><em># Create a s ocke t</em></li>

 <li>c l i e n t S o c k e t = s o c k e t ( AF IN<u>E</u>T , SOCK STREAM) 11</li>

 <li><em># Connect to the serve r</em></li>

 <li>c l i e n t S o c k e t . connect ( ( serverName , s e rv e r Po r t ) ) 14</li>

 <li><em># A s t ring we want to send to the serve r</em></li>

 <li>data =  ” Hel l o  <u> </u>  world!  <u> </u> This  <u> </u> i s  <u> </u> a  <u> </u> very  <u> </u> long   s t r i n g. ” 17</li>

</ul>

18  <em># byte s Sent = 0</em>

19

20 <em># Keep  sending  bytes until a l l bytes are sent</em> 21 <strong>while   </strong>bytes Sent != <strong>len </strong>( data ) :

<ul>

 <li><em># Send that s t r ing !</em></li>

 <li>bytes Sent += c l i e n t S o c k e t . send ( data [ bytes Sent : ] ) 24</li>

 <li><em># Close the s ocke t</em></li>

 <li>c l i e n t S o c k e t . c l o s e ( )</li>

</ul>




We made three changes. First, we added an integer variable, bytesSent, which keeps track of how many bytes the client has sent. We then added line: while bytesSent != len(data):. This loop will repeatedly call send() until bytesSent is equal to the length of our data. Another words, this line says “keep sending until all bytes of data have been sent.”

Finally, inside the loop, we have line bytesSent += clientSocket.send(data[bytesSent:]). Recall, that send() returns the  number  of bytes it has just sent. Hence, whenever send() sends x bytes, bytesSent will be incremented by x. The parameter, data[bytesSent:], will return all bytes that come after the first bytesSent bytes of data. This ensures that in the next iteration of the loop, we will resume sending at the offset of data where we left off.

With the client code working, lets now turn our attention to the server code. The modified code   is given below:




<ul>

 <li><em># Server code</em></li>

 <li><strong>from </strong>s o c k e t <strong>import </strong></li>

</ul>

3

<ul>

 <li><em># The port on which to l i s t en</em></li>

 <li>s e rv e r Po r t = 12000</li>

</ul>

6

<ul>

 <li><em># Create a TCP s ocke t</em></li>

 <li>s e r v e r S o c k e t = s o c k e t (      AF INET        ,SOCK STREAM) 9</li>

 <li><em># Bind the s ocke t to the port</em></li>

 <li>s e r v e r S o c k e t . bind ( ( ’ ’ , s e r v e r P o r t ) )</li>

</ul>

12

<ul>

 <li><em># Sta rt l i s t ening for incoming connections</em></li>

 <li>s e r v e r S o c k e t . l i s t e n ( 1 ) 15</li>

</ul>

16           <strong>print  </strong>”The  <u> </u> s e r v e r <u> </u>  i s <u>  </u>  ready <u> </u>to  <u>  </u> r e c e i v e ” 17

18 <em># Forever accept incoming connections</em> 19 <strong>while </strong>1 :

<ul>

 <li><em># Accept a connection ; get client ’ s s ocke t</em></li>

 <li>connection Socket , addr = s e r v e r So c k e t . accept ( )</li>

</ul>

22

<ul>

 <li><em># The temporary buffer</em></li>

 <li>tmpBuff= ”” 25</li>

 <li><strong>while len </strong>( data ) != 4 0 :</li>

 <li><em># Receive whatever the newly connected client has to send</em></li>

 <li>tmpBuff = connectionSocket . recv (40 29</li>

 <li><em># The other side unexpectedly clo s e d i t ’ s s ocke t</em></li>

 <li><strong>if not </strong>tmpBuff :</li>

 <li><strong>break</strong></li>

</ul>

33

<ul>

 <li><em># Save the data</em></li>

 <li>data += tmpBuff</li>

</ul>

36

37                                  <strong>print </strong>data 38

39 <em># Close the s ocket</em> 40 connection Socket . c l o se ( )




We made several changes. First, we added loop, while len(data) != 40:, which will spin until the size of data becomes 40. Hence, if recv() is unable to receive the expected 40 bytes after the first call, the loop will ensure that the program calls recv() again in order to receive the remaining bytes. Also, we changed line data = connectionSocket.recv(40) to tmpBuff= connectionSocket.recv(40) and added

test if not tmpBuff: which will evaluate to true if the other side unexpectedly closed it’s socket. We then added line data += tmpBuff which adds the newly received bytes to the accumulator file data buffer. These changes ensure that with every iteration the newly received bytes are appended to the end of the buffer.

At this point we have fully functioning server and client programs which do not suffer from the problems discussed above. However, there is still one important caveat: <strong>In the above</strong> <strong>code, what if the server does not know the amount of data the client will be sending? </strong>

This is often the case in the real world. The answer is that the client will have to tell the server. How? Well, this is where it is up to the programmer to decide on the types and formats of messages that the server and client will be exchanging.




One approach, for example, is to decide that all messages sent from client to server start with a 10 byte header indicating the size of the data in the message followed by the actual data. Hence, the server will always receive the first 10 bytes of data, parse them and determine the size of the data, and then use a loop as we did above, in order to receive the amount of data indicated by the header. You can see such example in the directory Assignment1SampleCodes/Python/sendfile.




Before proceeding, it is recommended that you stop and think about what you just read, experiment with Assignment1SampleCodes/Python/sendfile codes, and make sure you understand   everything.




<h1>Specifications</h1>

<strong> </strong>

<strong>The server shall be invoked  as:</strong>

<strong> </strong>

python serv.py &lt;PORT NUMBER&gt;




&lt;PORT NUMBER&gt; specifies the port at which ftp server accepts connection requests. For example:    python    serv.py   1234




<strong>The ftp client is invoked as:</strong>




<strong> </strong>

cli <em>&lt;</em>server machine&gt; &lt;server  port<em>&gt;</em>

<em> </em>

<em>&lt;</em>server machine<em>&gt; </em>is the domain name of the server (ecs.fullerton.edu). This will be converted into 32 bit IP address using DNS lookup. For example: python cli.py ecs.fullerton.edu       1234




Upon connecting to the server, the client prints out <strong>ftp</strong><em>&gt;</em>, which allows the user to execute the following commands.




ftp&gt; get &lt;file name&gt; (downloads file <em>&lt;</em>file name<em>&gt; </em>from the server) ftp&gt; put &lt;filename&gt; (uploads file <em>&lt;</em>file name<em>&gt; </em>to the server) ftp&gt; ls (lists files on the server)

ftp&gt; quit (disconnects from the server and exits)




<strong><em>Use two connections for each ftp session – control and data</em></strong>.  Control  channel  lasts  throughout the ftp session and  is  used  to  transfer  all  commands  (ls, get, and  put)  from client  to  server and all status/error messages from server to client. The initial channel on which the client connects to server will be the control channel.  Data channel is used for data transfer.  It  is  established  and torn down for every file transfer – when the client wants to  transfer  data  (ls,  get,  or  put),  it generates an ephemeral port to use for connection and then wait for the server  to  connect  to  the client on that port. The connection is then used to upload/download file to/from the server, and is torn down after the transfer iscomplete.




For each command/request sent from the client to server, the server prints out the message indicating SUCCESS/FAILURE of the command. At the end of each transfer, client prints the filename and number of bytes transferred.




<h1>Designing the Protocol</h1>

<strong> </strong>

Before you start coding, please design an application-layer protocol that meets the above specifications.

Please submit the design along with your code. Here are some guidelines to help you get started:

<ul>

 <li>What kinds of messages will be exchanged across the control channel?</li>

 <li>How should the other side respond to the messages?</li>

 <li>What sizes/formats will the messages have?</li>

 <li>What message exchanges have to take place in order to setup a file transfer channel?</li>

 <li>How will the receiving side know when to start/stop receiving the file?</li>

 <li>How to avoid overflowing TCP buffers?</li>

 <li>You may want to use diagrams to model your protocol.</li>

</ul>




<h1>Tips</h1>

<ul>

 <li>All sample files are in directory Assignment1SampleCodes. The following files contain sample Python codes, should you choose to use this language.

  <ul>

   <li>Please see sample file, ephemeral.py, illustrating how to generate an ephemeral port number. This program basically creates a socket and calls bind() in  order  to  bind the socket to port 0; calling bind() with port 0 binds the socket to the first available port.</li>

   <li>Please see file cmds.py which illustrates how to run an ls command from your code and to capture its input.</li>

   <li>Subdirectory sendfile contains files sendfileserv.py and sendfilecli.py, illustrating how to correctly transfer data over sockets (this includes large files).</li>

  </ul></li>

</ul>


