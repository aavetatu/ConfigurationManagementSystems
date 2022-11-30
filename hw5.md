# Homework 5

1.12.2022

## a) Make a SLS-file and store a state which makes "suolaikkuna.txt" file on your Windows machine

I had git and salt-minion installed on my windows machine before starting the task

First I needed to open Windows PowerShell with administrator priviledges

I needed to make a directory for storing the files and states for this homework

  cd /
  md Temp
  cd .\Temp\
  
Next I tried to make a file manually before trying it with salt states

  salt-call --local state.single file.managed 'C:\Temp\helloworld.txt'
   ls


    Directory: C:\Temp


  Mode                 LastWriteTime         Length Name
  ----                 -------------         ------ ----
  -a----         12/1/2022   1:15 AM              0 helloworld.txt
 
I created a new directory insind Temp to store salt state

  cd .\hw5\
  cat init.sls
  C:\Temp\suolaikkuna.txt:
    file.managed
    
And I ran the state

  salt-call --file-root=C:\Temp --local state.apply hw5
  cd C:\Temp\
  ls


      Directory: C:\Temp


  Mode                 LastWriteTime         Length Name
  ----                 -------------         ------ ----
  d-----         12/1/2022   1:23 AM                hw5
  -a----         12/1/2022   1:15 AM              0 helloworld.txt
  -a----         12/1/2022   1:32 AM              0 suolaikkuna.txt
  
I succeeded on creating a new file with salt state

## b)

## c)

### Sources

https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/
