---
layout: post
title: " Win7 Session隔离"
category: Windows
tags: [Windows, C++]
description: |
  Win7与WinXp实行不同的安全机制，Win7中session 0 无法访问到其他Session中的程序，本文提供了一种突破隔离的方法。
---

在Windows XP、Windows Server 2003及更早期的Windows版本里，控制台登录（在计算机本地进行登录叫做控制台登录）的所有服务都运行在相同的Session中，
此Session被称作"Session0"。所有的应用程序和服务都在Session0中运行，低权限的应用程序可直接与高特权级的系统服务进行通信，
这就为操作系统带来了一定的安全隐患。我们将操作系统比作一条船，将系统服务和应用程序比作乘船的人。
如果乘船的人中有一个人做出破坏行为，就极有可能导致整条船都沉没。

Windows 7采用了Session0隔离的安全模型。无论用户是否通过控制台进行登录，只有系统服务运行在Session0当中，而其余的应用程序都运行在Session1当中，
这就将应用程序与系统服务之间进行了相对隔离。当某个恶意程序试图攻击操作系统时，也无法直接与Windows 7的系统服务进行通信。还是将操作系统比作船，
应用程序和服务比作乘船的人。当某人破坏船只时，也只是应用程序那条船出现问题，而不会导致所有船只都出现问题甚至沉没

首个登录Windows 7的用户会直接登录到Session1上，在此之后登录的用户都会按序号登录到不同的Session当中，但只有系统服务是运行在Session0当中。
由于只有系统服务运行在Session0上，这样就保护了操作系统免受应用程序中所运行的恶意代码攻击。用户可以直接从任务管理器中验证当前是否正登录在Session1中.


<pre class="prettyprint linenums">
typedef BOOL (STDMETHODCALLTYPE FAR * LPFNCREATEENVIRONMENTBLOCK)
( LPVOID  *lpEnvironment,HANDLE hToken,BOOL bInherit );
typedef BOOL (STDMETHODCALLTYPE FAR * LPFNDESTROYENVIRONMENTBLOCK)
( LPVOID lpEnvironment );
typedef BOOL (WINAPI *WTSQueryUserTokenPROC)(ULONG SessionId, PHANDLE phToken );
</pre>

<pre class="prettyprint linenums" id="Cpp">
bool LockWorkStation()
{
	STARTUPINFO si = {0};
	PROCESS_INFORMATION pi = {0};

	DWORD dwSessionID = WTSGetActiveConsoleSessionId();

	HANDLE hToken = NULL;

	WTSQueryUserTokenPROC WTSQueryUserToken = NULL;

	HMODULE hInstWtsapi32 = LoadLibrary("Wtsapi32.dll");
	if(NULL != hInstWtsapi32)
	{
		WTSQueryUserToken = (WTSQueryUserTokenPROC)GetProcAddress(hInstWtsapi32,"WTSQueryUserToken");
	}
	LPFNCREATEENVIRONMENTBLOCK  lpfnCreateEnvironmentBlock  = NULL;
	LPFNDESTROYENVIRONMENTBLOCK lpfnDestroyEnvironmentBlock = NULL;
	HMODULE hUserEnvLib = NULL;
	hUserEnvLib = LoadLibrary( _T("userenv.dll") );
	if ( NULL != hUserEnvLib ) {
		lpfnCreateEnvironmentBlock  = (LPFNCREATEENVIRONMENTBLOCK)GetProcAddress( hUserEnvLib, "CreateEnvironmentBlock" );
		lpfnDestroyEnvironmentBlock = (LPFNDESTROYENVIRONMENTBLOCK)GetProcAddress( hUserEnvLib, "DestroyEnvironmentBlock");
	}

	// 获得当前Session的用户令牌
	if (WTSQueryUserToken(dwSessionID, &hToken) == FALSE)
	{
		Log("WTSQueryUserToken",GetLastError());
		return false;
	}
	// 复制令牌
	HANDLE hDuplicatedToken = NULL;
	if (DuplicateTokenEx(hToken,MAXIMUM_ALLOWED, NULL,
		SecurityIdentification, TokenPrimary,
		&hDuplicatedToken) == FALSE)
	{
		Log("hDuplicatedToken",GetLastError());
		return false;
	}

	// 创建用户Session环境
	LPVOID lpEnvironment = NULL;
	if (lpfnCreateEnvironmentBlock(&lpEnvironment,hDuplicatedToken, FALSE) == FALSE)
	{
		Log("CreateEnvironmentBlock" , GetLastError());
		return false;
	}

	//获得lockworkstation 程序的路径
	char lpszClientPath[MAX_PATH];
	GetModuleFileName(NULL, lpszClientPath, MAX_PATH);

	string path(lpszClientPath);
	int position = path.find_last_of("\\");
	string lws_name = path.substr(0,position) + LOCKWORKSTATION_APP_NAME;

	//创建新的用户线程执行lockworkstation
	if (CreateProcessAsUser(hDuplicatedToken, 
		lws_name.c_str(), NULL, NULL, NULL, FALSE,                    
		NORMAL_PRIORITY_CLASS | CREATE_NEW_CONSOLE |CREATE_UNICODE_ENVIRONMENT,
		lpEnvironment, NULL, &si, &pi) == FALSE)
	{
		Log("CreateProcessAsUser" , GetLastError());
		return false;
	}
	return false;
}
</pre>

##注意##
需要安装服务时具备较高的优先级 "NT AUTHORITY\\SYSTEM"