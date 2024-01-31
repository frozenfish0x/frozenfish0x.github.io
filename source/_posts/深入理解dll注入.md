---
layout: 逆向
title: 深入理解dll注入
date: 2022-12-1 04:17:37
tags:
---

DLL注入是指向运行中的其它进程强制插入特定的DLL文件。从技术细节来说，DLL注入命令其它进程自行调用LoadLibrary()API，加载用户指定的DLL文件。DLL注入与一般DLL加载的区别在于，加载的目标进程是其自身或其他进程。
<!--more-->
可以看到，test.dll已被强制插入进程(本来notepad并不会加载test.dll)。加载到某一进程中的test.dll与已经加载到某一进程中的dll一样，拥有访问notepad.exe进程内存的权限。

DLL被加载到进程后会自动运行DLLMain()函数，用户可以把想执行的代码放到DLLMain()函数，
每当加载DLL时，添加的代码就会得到执行。利用这种特性可以修复程序Bug以及添加新功能
DllMain 函数是DLL模块的默认 入口点。当Windows加载DLL模块时调用这一函数。
系统首先调用全局对象的 构造函数，然后调用 全局函数DLLMain。
DLLMain 函数不仅在将DLL链接加载到进程时被调用，在DLL模块与进程分离时（以及其它时候）也被调用。
 <!--more-->
BOOL APIENTRY DllMain( HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
　switch (ul_reason_for_call)
　{
　　case DLL_PROCESS_ATTACH:
      //添加想要执行的代码
      //当dll被进程加载时DLLMain被调用
　　　//printf(" process attach of dll");
　　　break;
　　case DLL_THREAD_ATTACH:
      //添加想要执行的代码
      //当有线程被创建时，DLLMain被调用
　　　printf(" thread attach of dll");
　　　break;
　　case DLL_THREAD_DETACH:
      //添加想要执行的代码
      //当有线程结束时，DLLMain被调用
　　　printf(" thread detach of dll");
　　　break;
　　case DLL_PROCESS_DETACH:
      //添加想要执行的代码
      //当dll被进程卸载时，DLLMain被调用
　　　printf(" process detach of dll");
　　　break;
　}
　return TRUE;
}




一、DLL注入示例


使用LoadLibrary()API加载某个DLL时，该DLL中的DLLMain()函数会被调用执行。DLL注入的工作原理就是从外部促使目标进程调用LoadLibrary()API,所以会强制调用执行DLL的DLLMain函数。并且被注入的DLL拥有目标进程内存的访问权限，用户可以随意操作。





二、实现DLL注入的方法


1、创建远程线程(CreatRemoteThread)



2、使用注册表(AppInit_DLLs值)



3、消息钩取（SetWindowsHookEx()API）





三、创建远程线程(CreatRemoteThread)


3.1、效果示例



运行process explorer（或者火绒剑，任务管理器）获取notepad.exe进程的pid。



可以看见process explorer.exe的pid为2788。






运行InjectDll.exe将myhack.dll注入到notepad.exe进程当中。可以看到dll文件已经被注入到里面。






要想在process explorer中看见注入的dll文件，需要依次选择view->Lower Pane view->DLLS选项。






进行注入时需要注意：

1.LoadLibraryA 和 LoadLibraryW 不同字符表示
之前一直没有成功，没有使用L,但是使用了LoadLibraryW，导致加载dll失败，
如果不使用L，请用LoadLibraryA
 
2.注册的时候注意DLL完整路径，除非被注入程序和dll在同一个文件夹
InjectDll.exe 3480 D:\test\myhack.dll


同时可以看见文件内已经多了一个html文件，此文件是dll中所指定的文件。






3.2、分析示例源码



在DLLMmain()函数中可以看到，这个dll被加载(DLL_PROCESS_ATTACH)时,先输出一个字符串（"<myhack.dll> Injection!!!"），然后再创建线程调用函数（ThreadProc）。在ThreadProc函数中通过调用URLDownloadToFile来下载指定网站的index.html文件。前面提到过，向进程注入dll后会调用dll的DLLMain函数。所以当dll文件注入到exe进程后，会调用URLDownloadToFile下载文件。

//myhack.cpp
#include "windows.h"
#include "tchar.h"
 
#pragma comment(lib, "urlmon.lib")
 
#define DEF_URL       (L"http://www.naver.com/index.html")
#define DEF_FILE_NAME   (L"index.html")
 
HMODULE g_hMod = NULL;
 
DWORD WINAPI ThreadProc(LPVOID lParam)
{
    TCHAR szPath[_MAX_PATH] = {0,};
 
    if( !GetModuleFileName( g_hMod, szPath, MAX_PATH ) )
        return FALSE;
     
    TCHAR *p = _tcsrchr( szPath, '\\' );
    if( !p )
        return FALSE;
    //下载指定网站的index.html文件
    _tcscpy_s(p+1, _MAX_PATH, DEF_FILE_NAME);
 
    URLDownloadToFile(NULL, DEF_URL, szPath, 0, NULL); 
 
    return 0;
}
 
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    HANDLE hThread = NULL;
 
    g_hMod = (HMODULE)hinstDLL;
 
    switch( fdwReason )
    {
    case DLL_PROCESS_ATTACH :  //加载时
        OutputDebugString(L"<myhack.dll> Injection!!!"); //输出调试字符串
        hThread = CreateThread(NULL, 0, ThreadProc, NULL, 0, NULL); //创建线程
        CloseHandle(hThread);
        break;
    }
 
    return TRUE;
}


main函数的主要功能时检查输入程序的参数，然后调用InjectDLL函数。InjectDLL函数是用来进行dll注入的核心，其作用是使目标进程自行调用LoadLibrary这个api。

//InjectDLL.cpp
#include "windows.h"
#include "tchar.h"
 
BOOL SetPrivilege(LPCTSTR lpszPrivilege, BOOL bEnablePrivilege) 
{
    TOKEN_PRIVILEGES tp;
    HANDLE hToken;
    LUID luid;
 
    if( !OpenProcessToken(GetCurrentProcess(),
                          TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, 
                          &hToken) )
    {
        _tprintf(L"OpenProcessToken error: %u\n", GetLastError());
        return FALSE;
    }
 
    if( !LookupPrivilegeValue(NULL,           // lookup privilege on local system
                              lpszPrivilege,  // privilege to lookup 
                              &luid) )        // receives LUID of privilege
    {
        _tprintf(L"LookupPrivilegeValue error: %u\n", GetLastError() ); 
        return FALSE; 
    }
 
    tp.PrivilegeCount = 1;
    tp.Privileges[0].Luid = luid;
    if( bEnablePrivilege )
        tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    else
        tp.Privileges[0].Attributes = 0;
 
    // Enable the privilege or disable all privileges.
    if( !AdjustTokenPrivileges(hToken, 
                               FALSE, 
                               &tp, 
                               sizeof(TOKEN_PRIVILEGES), 
                               (PTOKEN_PRIVILEGES) NULL, 
                               (PDWORD) NULL) )
    { 
        _tprintf(L"AdjustTokenPrivileges error: %u\n", GetLastError() ); 
        return FALSE; 
    } 
 
    if( GetLastError() == ERROR_NOT_ALL_ASSIGNED )
    {
        _tprintf(L"The token does not have the specified privilege. \n");
        return FALSE;
    } 
 
    return TRUE;
}
 
BOOL InjectDll(DWORD dwPID, LPCTSTR szDllPath)
{
    HANDLE hProcess = NULL, hThread = NULL;
    HMODULE hMod = NULL;
    LPVOID pRemoteBuf = NULL;
    DWORD dwBufSize = (DWORD)(_tcslen(szDllPath) + 1) * sizeof(TCHAR);
    LPTHREAD_START_ROUTINE pThreadProc;
 
    // #1. 使用 dwPID 获取目标进程(notepad.exe)句柄（PROCESS_ALL_ACCESS权限），然后就可以用 hProcess 控制进程.
    if ( !(hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID)) )
    {
        _tprintf(L"OpenProcess(%d) failed!!! [%d]\n", dwPID, GetLastError());
        return FALSE;
    }
 
    // #2. 在目标进程(notepad.exe) 内存中分配 szDllName 大小的内存，返回 pRemoteBuf 作为该缓冲区的地址.
    pRemoteBuf = VirtualAllocEx(hProcess, NULL, dwBufSize, MEM_COMMIT, PAGE_READWRITE);
 
    // #3. 将 myhack.dll 路径写入刚刚分配的缓冲区.
    WriteProcessMemory(hProcess, pRemoteBuf, (LPVOID)szDllPath, dwBufSize, NULL);
 
    // #4. 获取 LoadLibraryW() API 地址，kernel32.dll在每个进程中的加载地址相同（这个特性就是我们要利用的）.
    hMod = GetModuleHandle(L"kernel32.dll");
    pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hMod, "LoadLibraryW");
     
    // #5. 在 notepad.exe 中运行线程
    hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, pRemoteBuf, 0, NULL); //CreateRemoteThread()驱使进程调用LoadLibrary()，进而加载指定的DLL文件
    WaitForSingleObject(hThread, INFINITE);   
 
    CloseHandle(hThread);
    CloseHandle(hProcess);
 
    return TRUE;
}
 
int _tmain(int argc, TCHAR *argv[])
{
    if( argc != 3)
    {
        _tprintf(L"USAGE : %s <pid> <dll_path>\n", argv[0]);
        return 1;
    }
 
    // change privilege
    if( !SetPrivilege(SE_DEBUG_NAME, TRUE) )
        return 1;
 
    // inject dll
    if( InjectDll((DWORD)_tstol(argv[1]), argv[2]) )
        _tprintf(L"InjectDll(\"%s\") success!!!\n", argv[2]);
    else
        _tprintf(L"InjectDll(\"%s\") failed!!!\n", argv[2]);
 
    return 0;
}


下面来详细分析一下injectDll函数。



调用 OpenProcess这个API，借助程序运行时以参数形势传递过来的dwPID值，获取exe进程的句柄（PROCESS_ALL_ACCESS）。得到PROCESS_ALL_ACCESS之后，就可以用获取的句柄控制对应进程。

//获取目标的进程句柄
hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID)


需要把即将加载的dll文件的路径通知目标进程。因为任何内存空间都无法进行写入操作，所以先使用VirtualAllocEx() API在目标进程的内存空间中分配一块缓冲区，且指定的缓冲区大小为dll文件路径字符串的长度。

pRemoteBuf = VirtualAllocEx(hProcess, NULL, dwBufSize, MEM_COMMIT, PAGE_READWRITE);


提示：
VirtualAllocEx()函数的返回值为分配所得缓冲区的地址。
该地址不是程序自身进程(Inject.exe)的内存地址，
而是hProcess句柄所指目标进程（notepad.exe）的内存地址。


使用WriteProcessMemory将DLL路径字符串（xxx\xxx\xxx.dll）写入到分配所得缓冲区地址。WriteProcessMemory所写的内存空间也是hProcess句柄所指的目标进程的内存空间。

 WriteProcessMemory(hProcess, pRemoteBuf, (LPVOID)szDllPath, dwBufSize, NULL);
  
 //调用API
 //Windows操作系统提供了调试API，借助其可以访问其它进程的内存空间。
 //例如：VirtualAllocEx()、 WriteProcessMemory等
调用LoadLibrary前需要先获取其地址。LoadLibraryW()是LoadLibrary()的Unicode字符串版本。



我们的目标明明是获取加载到 notepad.exe 进程的kernel32.dll的 LoadLibraryW的起始地址，但代码却用来获取加载到 InjectDll.exe进程的kernel32.dll的 LoadLibraryW的起始地址。如果加载到 notepad.exe 进程中的kernel32.dl的地址与加载到 InjectDll.exe 进程中的kernel32.dll的地址相同，那么上面的代码就不会有什么问题。但是如果kernell32.d在每个进程中加载的地址都不同，那么上面的代码就错了，执行时会发生引用错误。



根据 Os 类型、语言、版本不同，kerne32.dll加载的地址也不网。并且 Vista /7中应用了新的 ASLR 功能，每次启动时。系统 DLL 加载的地址都会改0。但是在系统运行期间它都会被映射（ Mapping ）到号进程的相同地址。



Windows 作系统中， DLL 首次进入内存称为“加载”( Loading )，以后其他进程需要使用相网 DLL 时不必再次加载，只要将加载过的 DLL 代码与资源映射一下即可，这种映射技术有利于提高肉存的使用效率。



像上面这样， OS 核心 DUL 会被加载到自身固有的地址， DLL 注人利用的就是 Windows Os 这一特性（该特性也可能会被恶意使用，成为 Windows 安全漏洞）。导人InjectDll.exe进程中LoadlibraryW地址与导人notepad.exe进程中的LoadLibraryWO地址是相同的。

//在windows中，kernel32.dll在每个进程中的加载地址是相同的。
hMod = GetModuleHandle(L"kernel32.dll");
pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hMod, "LoadLibraryW");


在目标进程中运行远程线程，pThreadProc是exe进程内存中LoadlibraryW的地址，pRemoteBuf是exe进程内存中dll字符串的地址。

hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, pRemoteBuf, 0, NULL);


CreateRemoteThread用来在目标进程中执行其创建的线程，其函数原型如下：



除第一个参数 hProcess 外，其他参数与 CreateThread (）函数完全一样。 hProcess 参数是要执行线程的目标进程（或称“远程进程”、“宿主进程”）的句柄。 IpStartAddress 与 IpParameter 参数分别给出线程函数地址与线程参数地址。需要注意的是，这2个地址都应该在目标进程虚拟内存空间中（这样目标进程才能认识它们）。

HANDLE CreateRemoteThread(
  // 进程句柄
  hProcess,                          
  //  线程安全描述字，指向SECURITY_ATTRIBUTES结构的指针
  LPSECURITY_ATTRIBUTES lpThreadAttributes, 
  SIZE_T dwStackSize,                       //  线程栈大小，以字节表示
  LPTHREAD_START_ROUTINE lpStartAddress,    // 指向在远程进程中执行的函数地址
  LPVOID lpParameter,                       // 传入参数
  DWORD dwCreationFlags,                    // 创建线程的其它标志
  LPDWORD lpThreadId                        // 线程身份标志，如果为NULL,则不返回
);
 
 
HANDLE CreateThread(
  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  SIZE_T                  dwStackSize,
  LPTHREAD_START_ROUTINE  lpStartAddress,
  __drv_aliasesMem LPVOID lpParameter,
  DWORD                   dwCreationFlags,
  LPDWORD                 lpThreadId
);


查看ThreadProc与LoadLibrary。两函数都有一个4字节的参数，并返回一个4字节的值。也就是说，二者形态结构完全一样灵感即源于此。调用 CreateRemoteThread 时，只要将 LoadLibrary函数的地址传递给第四个参数 IpStartAddress ，把要注人的 DLL 的路径字符串地址传递给第五个参数 IpParameter 即可（必须是目标进程的虚拟内存空间中的地址）。由于前面已经做好了一切准备，现在调用该函数使目标进程加载指定的 DLL 文件就行了。

CreateRemoteThread （函数最主要的功能就是驱使目标进程调用LoadLibrary函数，进而加载指定的 DLL 文件。

//调用 CreateRemoteThread 创建远程线程所需要的过程函数的标准形式为
DWORD WINAPI ThreadProc(
  _In_ LPVOID lpParameter
);
 
//Win32编程加载DLL的API为：
HMODULE WINAPI LoadLibrary(
  _In_ LPCTSTR lpFileName
);




四、DLL卸载


DLL卸载（DLL Ejection）：将强制插入进程的DLL弹出的技术。

原理：驱使目标进程调用FreeLibrary() API。

提示：
FreeLibrary卸载dll的方法只适用于CreateRemoteThread注入


先注入dll到目标进程






注入成功后，卸载dll






分析一下EjectDll.exe

#include "windows.h"
#include "tlhelp32.h"
#include "tchar.h"
 
#define DEF_PROC_NAME  (L"notepad.exe")
#define DEF_DLL_NAME   (L"myhack.dll")
 
DWORD FindProcessID(LPCTSTR szProcessName)
{
    DWORD dwPID = 0xFFFFFFFF;
    HANDLE hSnapShot = INVALID_HANDLE_VALUE;
    PROCESSENTRY32 pe;
 
    // Get the snapshot of the system
    pe.dwSize = sizeof( PROCESSENTRY32 );
    hSnapShot = CreateToolhelp32Snapshot( TH32CS_SNAPALL, NULL );
 
    // find process
    Process32First(hSnapShot, &pe);
    do
    {
        if(!_tcsicmp(szProcessName, (LPCTSTR)pe.szExeFile))
        {
            dwPID = pe.th32ProcessID;
            break;
        }
    }
    while(Process32Next(hSnapShot, &pe));
 
    CloseHandle(hSnapShot);
 
    return dwPID;
}
 
BOOL SetPrivilege(LPCTSTR lpszPrivilege, BOOL bEnablePrivilege) 
{
    TOKEN_PRIVILEGES tp;
    HANDLE hToken;
    LUID luid;
 
    if( !OpenProcessToken(GetCurrentProcess(),
                          TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, 
                          &hToken) )
    {
        _tprintf(L"OpenProcessToken error: %u\n", GetLastError());
        return FALSE;
    }
 
    if( !LookupPrivilegeValue(NULL,           // lookup privilege on local system
                              lpszPrivilege,  // privilege to lookup 
                              &luid) )        // receives LUID of privilege
    {
        _tprintf(L"LookupPrivilegeValue error: %u\n", GetLastError() ); 
        return FALSE; 
    }
 
    tp.PrivilegeCount = 1;
    tp.Privileges[0].Luid = luid;
    if( bEnablePrivilege )
        tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    else
        tp.Privileges[0].Attributes = 0;
 
    // Enable the privilege or disable all privileges.
    if( !AdjustTokenPrivileges(hToken, 
                               FALSE, 
                               &tp, 
                               sizeof(TOKEN_PRIVILEGES), 
                               (PTOKEN_PRIVILEGES) NULL, 
                               (PDWORD) NULL) )
    { 
        _tprintf(L"AdjustTokenPrivileges error: %u\n", GetLastError() ); 
        return FALSE; 
    } 
 
    if( GetLastError() == ERROR_NOT_ALL_ASSIGNED )
    {
        _tprintf(L"The token does not have the specified privilege. \n");
        return FALSE;
    } 
 
    return TRUE;
}
 
BOOL EjectDll(DWORD dwPID, LPCTSTR szDllName)
{
    BOOL bMore = FALSE, bFound = FALSE;
    HANDLE hSnapshot, hProcess, hThread;
    HMODULE hModule = NULL;
    MODULEENTRY32 me = { sizeof(me) };
    LPTHREAD_START_ROUTINE pThreadProc;
 
    // dwPID = notepad 进程 ID
    // 使用 TH32CS_SNAPMODULE 参数，获取加载到 notepad 进程的 DLL名称
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, dwPID);
 
    bMore = Module32First(hSnapshot, &me);
    for( ; bMore ; bMore = Module32Next(hSnapshot, &me) )
    {
        if( !_tcsicmp((LPCTSTR)me.szModule, szDllName) || 
            !_tcsicmp((LPCTSTR)me.szExePath, szDllName) ) 
        {
            bFound = TRUE;
            break;
        }
    }
 
    if( !bFound )
    {
        CloseHandle(hSnapshot);
        return FALSE;
    }
 
    if ( !(hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID)) ) //使用进程ID来获取目标进程的进程句柄
    {
        _tprintf(L"OpenProcess(%d) failed!!! [%d]\n", dwPID, GetLastError());
        return FALSE;
    }
    
    //获取加载到EjectDll.exe进程的kernel32.FreeLibrary地址（这个地址在所有进程中是一样的）
    hModule = GetModuleHandle(L"kernel32.dll");
    pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "FreeLibrary");
     
    //在目标进程中运行线程，pThreadProc是FreeLibrary地址，me.modBaseAddr是要卸载的DLL的加载地址
    hThread = CreateRemoteThread(hProcess, NULL, 0, 
                                 pThreadProc, me.modBaseAddr, 
                                 0, NULL);
    WaitForSingleObject(hThread, INFINITE);   
 
    CloseHandle(hThread);
    CloseHandle(hProcess);
    CloseHandle(hSnapshot);
 
    return TRUE;
}
 
int _tmain(int argc, TCHAR* argv[])
{
    DWORD dwPID = 0xFFFFFFFF;
  
    // find process
    dwPID = FindProcessID(DEF_PROC_NAME);
    if( dwPID == 0xFFFFFFFF )
    {
        _tprintf(L"There is no <%s> process!\n", DEF_PROC_NAME);
        return 1;
    }
 
    _tprintf(L"PID of \"%s\" is %d\n", DEF_PROC_NAME, dwPID);
 
    // change privilege
    if( !SetPrivilege(SE_DEBUG_NAME, TRUE) )
        return 1;
 
    // eject dll
    if( EjectDll(dwPID, DEF_DLL_NAME) )
        _tprintf(L"EjectDll(%d, \"%s\") success!!!\n", dwPID, DEF_DLL_NAME);
    else
        _tprintf(L"EjectDll(%d, \"%s\") failed!!!\n", dwPID, DEF_DLL_NAME);
 
    return 0;
}


获取进程中加载的DLL信息。

hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, dwPID);
//使用 Create Toolhelp32Snapshot0 API 可以获取加载到进程的模块（ DLL ）信息。
//将获取的 hSnapshot 句柄传递给Module32First(Module32NextO函数后，
//即可设置与MODULEENTRY32结构体相关的模块信息.


获取目标进程的句柄。

hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID)
//使用pid获取目标进程的进程句柄


获取FreeLibrary API地址

hModule = GetModuleHandle(L"kernel32.dll");
 pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "FreeLibrary");
 //若要驱使 notepad 进程自己调用 FreeLibrary API ，需要先得到 FreeLibrary的地址。
 //然而代码获取的不是加载到 notepad.exe 进程中的Kernel32!FreeLibrary 地址，
// 而是加载到 EjectDl . exei 程中的Kernel32! FreeLibrary 地址。


在目标进程中运行线程

hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, me.modBaseAddr, 0, NULL);
//pThreadProc 参数是 FreeLibrary API 的地址,
//me.modBaseAddr 参数是要卸载的 DLL 的加载地址。
//将线程函数指定为 FreeLibrary 函数，并把 DLL 加载地址传递给线程参数，
//就在目称世中成功调用了 FreeLibraryO ) API 。




五、AppInit_DLLss


计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows






填入dll文件路径






修改LoadAppInit_DLLs






重启系统使修改生效，使用火绒剑，process explorer查看是否注入成功。可以看见已经被注注入成功了。并且是注入了所有加载了user32.dll的进程。但是由于此dll的目标是notepad.exe进程，所以只要当运行这个exe之后才会有所动作。









myhack2.dll的源码比较简单。主要目的为加载进程为notepad.exe的程序，然后隐藏并连接指定网站。

// myhack2.cpp
 
#include "windows.h"
#include "tchar.h"
 
#define DEF_CMD  L"c:\\Program Files\\Internet Explorer\\iexplore.exe" 
#define DEF_ADDR L"http://www.naver.com"
#define DEF_DST_PROC L"notepad.exe"
 
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    TCHAR szCmd[MAX_PATH]  = {0,};
    TCHAR szPath[MAX_PATH] = {0,};
    TCHAR *p = NULL;
    STARTUPINFO si = {0,};
    PROCESS_INFORMATION pi = {0,};
 
    si.cb = sizeof(STARTUPINFO);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;
 
    switch( fdwReason )
    {
    case DLL_PROCESS_ATTACH : 
        if( !GetModuleFileName( NULL, szPath, MAX_PATH ) )
            break;
    
        if( !(p = _tcsrchr(szPath, '\\')) )
            break;
 
        if( _tcsicmp(p+1, DEF_DST_PROC) )
            break;
 
        wsprintf(szCmd, L"%s %s", DEF_CMD, DEF_ADDR);
        if( !CreateProcess(NULL, (LPTSTR)(LPCTSTR)szCmd, 
                            NULL, NULL, FALSE, 
                            NORMAL_PRIORITY_CLASS, 
                            NULL, NULL, &si, &pi) )
            break;
 
        if( pi.hProcess != NULL )
            CloseHandle(pi.hProcess);
 
        break;
    }
    
    return TRUE;
}




六、SetWindowsHooKEX（）


钩子过程（hook procedure）是系统调用的回调函数。



安装钩子时，钩子过程需要在DLL内部，该DLL的示例句柄（instance handle）即hMod。



线程ID如果为0，则钩子为“全局钩子”。



用SetWindowsHookEx()设置好钩子后，在某个进程中生成指定消息时，操作系统会将相关DLL文件强制注入相应进程。

HHOOK SetWindowsHookEx(
 
　　int idHook, // 钩子的类型，即它处理的消息类型
 
　　HOOKPROC lpfn, //钩子子程的地址指针。如果dwThreadId参数为0
 
　　// 或是一个由别的进程创建的线程的标识，
 
　　// lpfn必须指向DLL中的钩子子程。
 
　　// 除此以外，lpfn可以指向当前进程的一段钩子子程代码。
 
　　//钩子函数的入口地址，当钩子钩到任何消息后便调用这个函数。
 
　　HINSTANCE hMod, 
    //应用程序实例的句柄。标识包含lpfn所指的子程的DLL。
 
　　// 如果dwThreadId 标识当前进程创建的一个线程，
 
　　// 而且子程代码位于当前进程，hMod必须为NULL。
 
　　// 可以很简单的设定其为本应用程序的实例句柄。
 
　　DWORD dwThreadId //与安装的钩子子程相关联的线程的标识符。
 
　　// 如果为0，钩子子程与所有的线程关联，即为全局钩子。
 
　　);


6.1、效果示意



首先运行HookMain.exe






再运行notepad.exe，之后再使用查看，发现dll文件已经被注入。






输入q，拆除钩子。拆除后，dll文件消失，可以正常输入。






6.2、分析源码



HookMain.exe主要过程为：首先加载KeyHook.dll文件，然后调用HookStart()函数开始钩取，用户输入"q"时，调用HookStop()函数终止钩取。

//HookMain.exe
 
#include "stdio.h"
#include "conio.h"
#include "windows.h"
 
#define DEF_DLL_NAME        "KeyHook.dll"
#define DEF_HOOKSTART       "HookStart"
#define DEF_HOOKSTOP        "HookStop"
 
typedef void (*PFN_HOOKSTART)();
typedef void (*PFN_HOOKSTOP)();
 
void main()
{
    HMODULE         hDll = NULL;
    PFN_HOOKSTART   HookStart = NULL;
    PFN_HOOKSTOP    HookStop = NULL;
    char            ch = 0;
 
    // 加载KeyHook.dll
    hDll = LoadLibraryA(DEF_DLL_NAME);
    if( hDll == NULL )
    {
        printf("LoadLibrary(%s) failed!!! [%d]", DEF_DLL_NAME, GetLastError());
        return;
    }
 
    // 获取导出函数地址
    HookStart = (PFN_HOOKSTART)GetProcAddress(hDll, DEF_HOOKSTART);
    HookStop = (PFN_HOOKSTOP)GetProcAddress(hDll, DEF_HOOKSTOP);
 
    // 开始
    HookStart();
 
    // “q”退出
    printf("press 'q' to quit!\n");
    while( _getch() != 'q' )    ;
 
    // 结束
    HookStop();
     
    // 卸载 KeyHook.dll 
    FreeLibrary(hDll);
}


KeyHook.dll在调用HookStart()时，SetWindowsHookEx()函数就会将KeyboardProc()添加到键盘钩链。



安装好键盘“钩子”后，无论哪个进程，只要发生键盘输人事件， OS 就会强制将 KeyHook . dl 人相应进程。加载了 KeyHook.dll 的进程中，发生键盘事件时会首先调用执行KeyHookKeyboardProc 。



KeyboardProc 函数中发生键盘输入事件时，就会比较当前进程的名称与“ notepad.exe ”字符串，若相同，则返回1，终止 KeyboardProc（）函数，这意味着截获且删除消息。这样，键盘消息就不会传递到 notepad.exe 程序的消息队列。 安装好键盘“钩子”后，无论哪个进程，只要发生键盘输人事件， OS 就会强制将 KeyHook . dl 人相应进程。加载了 KcyHook . dll 的进程中，发生键盘事件时会首先调用执行 KeyHfookKeyboardProc 。



KeyboardProd 函数中发生键盘输人事件时，就会比较当前进程的名称与“ notepad . exe ”字符串，若相同，则返网1，终止 KcyboardProc （函数，这意味着截获且删除消息。这样，键盘消息就环会传透到 notapadexe 程序的消息队列。notepad.exe 未能接收到任何键盘消息，故无法输出。



除此之外（即当前进程名称非“ notepad . exe ”时），执行 return CallNextHookEx ( g_hHook , nCode , wParam, lParam)，消息会被传递到另一个应用程序或钩链的另一个“钩子”函数。

// KeyHook.dll
 
#include "stdio.h"
#include "windows.h"
 
#define DEF_PROCESS_NAME       "notepad.exe"
 
HINSTANCE g_hInstance = NULL;
HHOOK g_hHook = NULL;
HWND g_hWnd = NULL;
 
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD dwReason, LPVOID lpvReserved)
{
    switch( dwReason )
    {
        case DLL_PROCESS_ATTACH:
            g_hInstance = hinstDLL;
            break;
 
        case DLL_PROCESS_DETACH:
            break; 
    }
 
    return TRUE;
}
 
LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
    char szPath[MAX_PATH] = {0,};
    char *p = NULL;
 
    if( nCode >= 0 )
    {
        // bit 31 : 0 => press, 1 => release
        if( !(lParam & 0x80000000) ) //释放键盘按键时
        {
            GetModuleFileNameA(NULL, szPath, MAX_PATH);
            p = strrchr(szPath, '\\');
 
            // 比较当前进程名称，如果是 notepad.exe 则消息不会传给应用程序
            if( !_stricmp(p + 1, DEF_PROCESS_NAME) )
                return 1;
        }
    }
 
    //反之，调用 CallNextHookEx() 消息传给应用程序
    return CallNextHookEx(g_hHook, nCode, wParam, lParam);
}
 
#ifdef __cplusplus
extern "C" {
#endif
    __declspec(dllexport) void HookStart()
{
        g_hHook = SetWindowsHookEx(WH_KEYBOARD, KeyboardProc, g_hInstance, 0);
    }
 
    __declspec(dllexport) void HookStop()
{
        if( g_hHook )
        {
            UnhookWindowsHookEx(g_hHook);
            g_hHook = NULL;
        }
    }
#ifdef __cplusplus
}
#endif


6.3、调试方法



先调试HookMain().exe，OD打开该程序。






查找核心代码

1、逐行跟踪调试
2、检索相关API
3、搜索相关字符串


由于之前打开过此程序，所以直接搜索"press 'q' to quit!\n"。









点击后跳转到该位置，在401000地址处下断点。然后运行到此处。

先在401006地址处调用LoadLibrary,然后由40104B地址处的CALL指令调用KeyHook.HookStart()函数。






F7跟踪进入。



图中的代码是被加载到HookMain.exe 进程中的 KeyHook.dll的HookStart()函数。



在100010EF地址处可以看到 CALL SetWindowsHookExW() 指令，其上方10010E8与100010ED地址处的2条 PUSH 指令用于把 SetWindowsHookExW() API 的第1、2两个参数压入栈。


Set WindowsHookExW() API 的第一个参数（ idHook）值为WH_KEYBOARD(2)，第二个参数( Ipfn ）值为10001020，该值即是钩子过程的地址。后面调试 KeyHook . dlI 时再仔细看该地址。HookMain.exe的main()函数（401000）的其余代码接收到用户输人的“ q ”命令后终止钩取。






调试KeyHook.dll

使用OD打开notepad.exe,F9运行






在OD中设置如下的选项






运行HookMain.exe






随后在notepad中随意输入一个字母，此时dll被加载到10000000处






根据系统环境不同，有时不会先显示 KeyHook.dll，而是先加載其他 DLL 库。

此时按（F9）运行键，直到KeyHook.dll加载完成。

有些系统无法正常运行该功能，此时使用OllyDbg2.0即可保证运行顺畅。



点击dll跳转到KeyHook.dll的EP地址处。并且由于之前在调试HookMain.exe时候已经知道钩子的地址是10001020，所以直接在此处下断点。






下好断点好，再重新运行一下程序。然后在记事本当中尝试输入数据，记事本无接收数据的意向，并且OD已经跳转到断点处。






1、OD运行notepad.exe
2、开启Break on new module(中断于新模块)选项
3、运行HookMain.exe
4、进行键盘输入，触发键盘消息事件
5、dll被注入
6、OD中设置钩子进程（KeyboardProc）断点