# 创建windows服务

```c++
#define _SILENCE_STDEXT_HASH_DEPRECATION_WARNINGS // 如果编译出错，可加上这句

#include <Windows.h>
#include <stdio.h>
#include <fstream>

SERVICE_STATUS ServiceStatus;
SERVICE_STATUS_HANDLE hStatus;
LPWSTR SERVICE_NAME = L"gt";

int WriteToLog(char* str) {
	ofstream ofs;
	ofs.open("c:/gt/log.txt", std::ios::app);
	if (ofs.is_open()) {
		ofs << str << endl;
		ofs.close();
	}

	return 0;
}

void ControlHandler(DWORD request) {
	switch (request)
	{
	case SERVICE_CONTROL_STOP:
		WriteToLog("Monitoring stoppend.");
		ServiceStatus.dwCurrentState = SERVICE_STOPPED;
		ServiceStatus.dwWin32ExitCode = 0;
		SetServiceStatus(hStatus, &ServiceStatus);
		break;
	case SERVICE_CONTROL_SHUTDOWN:
		WriteToLog("Monitoring shutdown.");
		ServiceStatus.dwCurrentState = SERVICE_STOPPED;
		ServiceStatus.dwWin32ExitCode = 0;
		SetServiceStatus(hStatus, &ServiceStatus);
		break;
	default:
		break;
	}
}

// 服务的入口点。它运行在一个单独的线程当中，这个线程是由控制分派器创建的
void WINAPI ServerMain(int argc, char** args) {
	ServiceStatus.dwServiceType = SERVICE_WIN32; // 指示服务类型，SERVICE_WIN32：创建 Win32 服务
	ServiceStatus.dwCurrentState = SERVICE_START_PENDING;   // 指定服务的当前状态
    // 这个域通知 SCM 服务接受哪个域
	ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
	ServiceStatus.dwWin32ExitCode = 0;
	ServiceStatus.dwCheckPoint = 0;
	ServiceStatus.dwWaitHint = 0;

	// 为服务注册控制处理器,两个参数传递给此函数：服务名和指向 ControlHandlerfunction 的指针。 
	// 注册完控制处理器之后，获得状态句柄（hStatus）。
    // 通过调用 SetServiceStatus 函数，用 hStatus 向 SCM 报告服务的状态。 
	hStatus = RegisterServiceCtrlHandler(SERVICE_NAME, (LPHANDLER_FUNCTION)ControlHandler);
	if (hStatus == (SERVICE_STATUS_HANDLE)0) return; // Registering Controll Handler failed;

	ServiceStatus.dwCurrentState = SERVICE_RUNNING;
	SetServiceStatus(hStatus, &ServiceStatus);

	for (int i = 0; i < 10; i++) {
		WriteToLog("loop.");
		Sleep(6000);
	}

	ServiceStatus.dwCurrentState = SERVICE_STOPPED;
	ServiceStatus.dwWin32ExitCode = -1;
	SetServiceStatus(hStatus, &ServiceStatus);
}

int main() {
	SERVICE_TABLE_ENTRY arSrv[] = {
		// 服务名、服务入口函数
		{SERVICE_NAME,  (LPSERVICE_MAIN_FUNCTION)ServerMain},
		// 分派表最后一项必须名和入口都是NULL
		{NULL, NULL}
	};

	StartServiceCtrlDispatcher(arSrv); // 分派服务
}
```

```bash
#                          =后要有个空格
sc create MyService binpath= e:/xxx/MyService.exe # 创建服务MyService
# 可以通过 services.msc 查看到创建的服务
sc start MyService   # 启动服务
sc stop MyService    # 停止服务
sc delete MyService  # 删除服务
# 修改服务启动类型。start参数的值可以是demand（手动）、disabled（禁用），auto（自动）。
sc config 服务名 start= auto|demand|disabled 
```

