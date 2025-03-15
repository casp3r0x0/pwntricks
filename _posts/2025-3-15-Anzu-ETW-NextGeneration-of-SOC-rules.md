## About 

in this blog we are going to talk about my recent project called Anzu ETW , it is a tool that I developed to allow SOC/DFIR team to create plugins that use ETW or any other telemtry to log or response against cyber threats.  

![](https://www.pwntricks.com/assets/images/4/anzu-in-mythology.jpg)

## what is ETW ?

Event Tracing for Windows (ETW) is a high-performance logging framework in Windows that enables real-time event collection from both user-mode applications and kernel components. It is widely used for performance monitoring, security analysis, and debugging, as it allows event providers (system processes, applications) to generate logs that can be consumed by tools like PerfView, Sysmon, and Windows Performance Analyzer. ETW is efficient, supports dynamic event filtering, and is used by EDR solutions to detect threats like process injection and API abuse. While essential for security monitoring.

## why I created ANZU ETW ?

I created Anzu ETW , when I create the Cortex ransowmare protection bypass you can found here [https://www.pwntricks.com/Bypass-cortex-ransomware-protection](https://www.pwntricks.com/Bypass-cortex-ransomware-protection)  


SOC team can never develop a rule to reponse to such new threats or bypasses (new techniques) so I was think why there isn't any tool that can let SOC/DFIR team to develop custom plugin (rules) that can utlize ETW ? and there was no tool that can provide such an ability. 

## the idea ?

the idea is provide a tool that can make the usage of the ETW for the SOC much more easy and flexible , the EDRs is using ETW but they didn't provide any capabilities to SOC team to develop there own plugins (rules).  

Anzu Project is solving the following issues: 

1. SOC can not take ETW logs due the larg data it generate ,using AnzuETW you can specify exactly what log to be logged using custom plugin you can write in C#
2. it allow SOC team to create plugins that utlize ETW and log and reponse to the threat that including behavior analysis plugins that can be written by the SOC team
3. using AnzuETW can provide you logs that normaly you can not get.
4. using AnzuETW can let the SOC team to develop their own plugins responed to specific scenario or new technique not just rule it can be based on behavior.

## installation


here you can find the Anzu ETW project developed by me in C# :

[https://github.com/casp3r0x0/AnzuETW/](https://github.com/casp3r0x0/AnzuETW/) 

1. Anzu => AnzuETW with console window you can use this as testing 
2. AnzuService => AnzuETW Service that can run in background same as console version
3. CortexRansomwareBypassDetectPlugin => custom plugin that I wrote to reponde to custom threat that bypass the ransomware protection developed by cortex using behavior analsyis (this can not be done without Anzu ^_^)
4. LogNetwork => plguin to log network traffic
5. LogProviderExample => example plugin to log every command line.
6. install.ps1 => to install the service for anzu 

you can download the `LogProviderExample` as template and edit it to create your own plugin.

first you can download the AnzuService project and compile it from the source or download the compiled binary from the following link :

[https://github.com/casp3r0x0/AnzuETW/tree/main/AnzuService/AnzuService/bin/x64/Debug](https://github.com/casp3r0x0/AnzuETW/tree/main/AnzuService/AnzuService/bin/x64/Debug) 

create folder in `C:\` called `anzu` and copy the `install.ps1` and create folder called `plguins` you can change this from the code and recompile also you can use share folder in the network to get the plugins from.

![](https://www.pwntricks.com/assets/images/4/1.png)

run the installation script that will create the anzu service for you. 

![](https://www.pwntricks.com/assets/images/4/2.png)

now you can place the plugins in the plugins folder, please note that the plugins name is not matter, also there is no need for restart after adding a plguin should. 

![](https://www.pwntricks.com/assets/images/4/3.png)

please note that if you develop custom plugin use the `LogProviderExample` project it use `fody` automatically to compile the project into one `dll` file.

copy only the following dll file to the plugins folder.
![](https://www.pwntricks.com/assets/images/4/4.png)

anzu will create a log source in Windows logs , SOC can retrive them into the SIEM solutions, see the following:

![](https://www.pwntricks.com/assets/images/4/5.png)

## how the plugin works ? 

you can develop the plugin just like there is no intgeration Anzu will log every console output as log and save it in the windows event logs. 

you can just ask ChatGPT to create a plugin for you for example : 
```
create a C# code that can monitor using ETW any file read for the chrome cooiks and passwords 
```
then you can use the `LogProviderExample` as template and compile the project to get your own plugin! 

simple right?


## create a behaviour detection for my own cortex ransomware bypass ! 

the detect is also simple I have created the plugin you can find it in the Anzi github repo `CortexRansomwareBypassDetectPlugin` project, the detect is by monitor all the UNC lookups that is done on the machine if any lookup contains `users` likly this a malicious no normal software is doing a dirctory listing through UNC, right ?

the plugin will alert and kill the process!


## proof

![](https://www.pwntricks.com/assets/images/4/6.png)




