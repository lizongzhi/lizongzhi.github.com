---
layout: post
title: " Windows 服务应用程序 "
category: Windows
tags: [Windows , C++]
description: |
 本文提供一个windows 服务程序的 C 示例 和一个 C++ 示例
---
##C  版##

<pre class="prettyprint linenums">
SERVICE_STATUS          ServiceStatus; 
SERVICE_STATUS_HANDLE   hStatus; 

void  ServiceMain(int argc, char** argv); 
void  ControlHandler(DWORD request); 

// Control Handler
void ControlHandler(DWORD request) 
{ 
   switch(request) 
   { 
      case SERVICE_CONTROL_STOP: 
         ServiceStatus.dwWin32ExitCode = 0; 
         ServiceStatus.dwCurrentState = SERVICE_STOPPED; 
         SetServiceStatus (hStatus, &ServiceStatus);
         return; 
 
      case SERVICE_CONTROL_SHUTDOWN: 
         ServiceStatus.dwWin32ExitCode = 0; 
         ServiceStatus.dwCurrentState = SERVICE_STOPPED; 
         SetServiceStatus (hStatus, &ServiceStatus);
         return; 
        
      default:
         break;
    } 
 
}

void ServiceMain(int argc, char** argv) 
{ 
   int error; 
 
   ServiceStatus.dwServiceType = SERVICE_WIN32; 
   ServiceStatus.dwCurrentState = SERVICE_START_PENDING; 
   ServiceStatus.dwControlsAccepted  = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
   ServiceStatus.dwWin32ExitCode = 0; 
   ServiceStatus.dwServiceSpecificExitCode = 0; 
   ServiceStatus.dwCheckPoint = 0; 
   ServiceStatus.dwWaitHint = 0; 
 
   hStatus = RegisterServiceCtrlHandler("MemoryStatus", (LPHANDLER_FUNCTION)ControlHandler); 
   if (hStatus == (SERVICE_STATUS_HANDLE)0) 
   { 
      // Registering Control Handler failed
      return; 
   }  
   // We report the running status to SCM. 
   ServiceStatus.dwCurrentState = SERVICE_RUNNING; 
   SetServiceStatus (hStatus, &ServiceStatus);
 
   // The worker loop of a service
   while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
   {
	  Run();//执行应用程序
      Sleep(SLEEP_TIME);
   }
   return; 
}

void main(int argc, char* argv[])
{ 
   SERVICE_TABLE_ENTRY ServiceTable[1];
   ServiceTable[0].lpServiceName = "MemoryStatus";
   ServiceTable[0].lpServiceProc = (LPSERVICE_MAIN_FUNCTION)ServiceMain;
	
   // Start the control dispatcher thread for our service
   StartServiceCtrlDispatcher(ServiceTable);
}
</pre>

##C++版##

C++版是MSDN上的一个[示例](http://code.msdn.microsoft.com/windowsdesktop/CppWindowsService-cacf4948)，其中提供了类ServiceBase，只要继承这个基类并重载相应的OnStart,OnStop等方法，修改相应的参数，就可以了。

##调试方法##

<pre class="prettyprint linenums">
void Log(__in char* szInfo)
{
	char szOutput [cchOUTPUT];

	if (SUCCEEDED(StringCchPrintf(
		szOutput,
		cchOUTPUT,
		"%s - %s\n",
		szOUTPUT_TAG,
		szInfo)));
	OutputDebugString(szOutput);
}
</pre>
