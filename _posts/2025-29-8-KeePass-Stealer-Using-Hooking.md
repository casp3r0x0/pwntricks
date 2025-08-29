## About

in this blog post we are going to build our keepass stealer that will utilize hooking technique to steal the master password, without user knowledge.


## What is hooking ? 

Function hooking is when a program intercepts calls to a function in order to change or monitor its behavior. It's like putting a "middleman" between the original function and whatever is calling it, so you can see what's happening or modify the results before they reach the program.

this technique is used by EDRs to monitor WinAPI calls but today we are utlizing it to steal the master password from keepass. 

> we will use library called EasyHook to Implement the Hooks.

## what to hook ?

we can use tools like [apimonitor](http://www.rohitab.com/apimonitor) to get what is the nature of WinAPIs are the application is using so we can know what to hook exactly.


## RtlDecryptMemory 

after using API Monitor we can notice the application is using RtlDecryptMemory (SystemFunction041) to decrypts memory contents.

function prototype : 

```
NTSTATUS RtlDecryptMemory(
  [in, out] PVOID Memory,
  [in]      ULONG MemorySize,
  [in]      ULONG OptionFlags
);
```

link to microsoft docs: [https://learn.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-rtldecryptmemory](https://learn.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-rtldecryptmemory)

We can notice that the function is in : `Advapi32.dll` based on microsoft documentation.

## Implementation 

you can find my full implementation in Csharp in the following github repo :

[https://github.com/casp3r0x0/KeePassStealer](https://github.com/casp3r0x0/KeePassStealer) 

### project structure: 
- test --> DLL project that will be injected in keepass.exe process (hooking mechansim)
- injector -->  exe  project that will inject into the target process 

All you need is to run injector.exe 

### Injector.exe plan: 

1. find keepass process 
2. save the dll from memory to %temp%
3. Inject into target process
5. password will be logged into %temp%/AAAA.txt
4. done.

> Note you can kill KeePass.exe first so the user re-enter his password.


### Proof
![](https://www.pwntricks.com/assets/images/6/1.png)



