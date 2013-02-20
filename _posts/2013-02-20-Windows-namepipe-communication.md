---
layout: post
title: " Windows命名管道通信"
category: Windows
tags: [Windows, 网络, 进程间通信, C++]
description: |
  Windows中进程间通信的方法有许多种，命名管道就是其中之一。本文提供一个异步命名管道通信的示例
---

  命名管道是通过网络来完成进程之间的通信的，命名管道依赖于底层网络接口，其中包括有 DNS 服务，TCP/IP 协议等等机制，但是其屏蔽了底层的网络协议细节，
对于匿名管道而言，其只能实现在父进程和子进程之间进行通信，而对于命名管道而言，其不仅可以在本地机器上实现两个进程之间的通信，还可以跨越网络实现两个进程之间的通信。

命名管道使用了 Windows 安全机制，因而命名管道的服务端可以控制哪些客户有权与其建立连接，而哪些客户端是不能够与这个命名管道建立连接的。
利用命名管道机制实现不同机器上的进程之间相互进行通信时，可以将命名管道作为一种网络编程方案时，也就是看做是 Socket 就可以了，它实际上是建立了一个客户机/服务器通信体系，并在其中可靠的传输数据。
命名管道的通信是以连接的方式来进行的，服务器创建一个命名管道对象，然后在此对象上等待连接请求，一旦客户连接过来，则两者都可以通过命名管道读或者写数据。          

命名管道提供了两种通信模式：字节模式和消息模式。在字节模式下，数据以一个连续的字节流的形式在客户机和服务器之间流动，
而在消息模式下，客户机和服务器则通过一系列的不连续的数据单位，进行数据的收发，每次在管道上发出一个消息后，它必须作为一个完整的消息读入。

#服务器端#

##创建命名管道##

<pre class="prettyprint linenums" id="Cpp">
HANDLE hPipe = CreateNamedPipeW(PIPENAME,
		PIPE_ACCESS_DUPLEX | FILE_FLAG_OVERLAPPED,
		PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE | PIPE_WAIT,
		20,
		BUFSIZE,
		BUFSIZE,
		PIPE_TIMEOUT,
		NULL);
</pre>

##监听线程##

<pre class="prettyprint linenums lang-cpp">
bool ConnectPipe(OVERLAPPED* overLapped)
{
	if(!ConnectNamedPipe(hPipe, overLapped) && GetLastError() != 997/*WSA_IO_PENDING*/){
		Log("Connect Pipe Failed" , GetLastError());
		return false;
	}
	return true;
}
</pre>

<pre class="prettyprint linenums">
UINT WINAPI ListenThread(void* pParam)
{
	//异步IO
	HANDLE event = CreateEvent(NULL, TRUE, FALSE, NULL);

	OVERLAPPED overLapped;
	ZeroMemory(&overLapped, sizeof(OVERLAPPED));
	overLapped.hEvent = event;
	if(!ConnectPipe(&overLapped))
		return 0;

	while(!exit) {

		DWORD result = WaitForMultipleObjects(1, &event, TRUE, 800);

		if(result == WAIT_FAILED){
			break;
		}
		if(result == WAIT_TIMEOUT){
			continue;
		}
		ResetEvent(event);

		DWORD cbBytesRead;
		char chRequest[PACKAGESIZE];
		while(!exit)
		{
			ReadFile (hPipe, chRequest, PACKAGESIZE, &cbBytesRead, NULL);
			if(PACKAGESIZE != cbBytesRead){
				break;
			}
			if(chRequest[0] == HEARTTICK){
				WriteFile(hPipe, chRequest,PACKAGESIZE,&cbBytesRead,NULL);
				if(PACKAGESIZE != cbBytesRead){
					break;
				}	
			}
		}
		DisconnectNamedPipe(hPipe);
		ZeroMemory(&overLapped, sizeof(OVERLAPPED));
		overLapped.hEvent = event;
		if(!ConnectPipe(&overLapped))
			return 0;
	}
	DisconnectNamedPipe(hPipe);
	hPipe = INVALID_HANDLE_VALUE;
	return 1;
}
</pre>

#客户端#

##打开管道##

<pre class="prettyprint linenums">
HANDLE hFile = CreateFileW(PIPENAME, GENERIC_WRITE|GENERIC_READ ,0, NULL,
		OPEN_EXISTING, 0, NULL);
</pre>

##读，写管道##
<pre class="prettyprint linenums">
WriteFile(hFile,&data[0], PACKAGESIZE, &byteToWrite, NULL);
	
ReadFile (hFile,&data[0], PACKAGESIZE, &byteToWrite, NULL);
</pre>

#参考资料#
《WINDOWS网络编程技术》
		
 