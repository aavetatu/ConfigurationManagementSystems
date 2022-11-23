# Homework 4

23.11.2022

## a) Make a new "Hello World" command and install it to the minion

I started with updating the packages and installing salt-master and salt-minion to the same machine

	$ sudo apt-get update
	$ sudo apt-get -y install salt-master salt-minion

Next I needed to print "Hello world!" to the terminal manually before making it a script to see if it actually works

	$ echo "Hello World!"
	Hello World!

I made a new directory to store all the scripts used in this homework and opened it

	$ mkdir scripts
	$ cd scripts/
		
I worte the new script, printed it with cat and ran it with bash

	$ micro helloworld.sh
	$ cat helloworld.sh 
	echo "Hello World!"
	$ bash helloworld.sh 
	Hello World!

The command I just made can be ran in bash, but it is not executable yet, and I can only run it if I'm in the same directory as the command

I new from previous experience that I needed to make the file executable by giving executing permission to every user on the machine, but it is always good to check the permissions first to avoid unnecessary mistakes

	$ ls -l helloworld.sh 
	-rw-r--r-- 1 tatu tatu 20 Nov 23 13:31 helloworld.sh

Permissions looked fine except the fact that it couldn't been executed. I used chmod to make the file executable and checked permissions of the file again to confirm the modifications

	$ chmod ugo+x helloworld.sh 
	$ ls -l helloworld.sh 
	-rwxr-xr-x 1 tatu tatu 20 Nov 23 13:31 helloworld.sh

Helloworld.sh was now executable, but it couldn't be ran yet because I hadn't specified it needs to run in bash. I edited the file again to make the computer know the file needs to be run in bash

	$ micro helloworld.sh
	$ cat helloworld.sh 
	#!/usr/bin/bash
	echo "Hello World!"

Now the file was ready to go except it was still in my home directory and not available to ran from accross the system. I needed to copy it to a directory where the machine can find it

	$ sudo cp helloworld.sh /usr/local/bin/

Next step was to make sure I could run the command

	$ helloworld.sh 
	Hello World!

I succesfully ran the command but I was still in the same directory as the command which could affect the test result so I moved to a different directory and tried again

	$ cd /etc/
	$ pwd
	/etc
	$ helloworld.sh 
	Hello World!

The command was now ready to be used but I still needed to install it to the same computer with salt

For salt I needed to make a new directory where I can store salt states

	$ sudo mkdir /srv/salt
	$ cd /srv/salt/

Next I needed to make a new directory and a file to store the state and another directory to store the scripts

	$ sudo mkdir helloworld
	$ sudo mkdir scripts

I already had the helloworld command done so I needed to copy it to copy it for salt to use

	$ sudo cp /usr/local/bin/helloworld.sh /srv/salt/scripts/
	$ ls scripts/
	helloworld.sh

After copying helloworld.sh I needed to make init.sls and define the state and run it

	$ cd helloworld/
	$ sudo micro init.sls
	$ cat init.sls
	/usr/local/bin/helloworld.sh:
	  file.managed:
	    - source: salt://scripts/helloworld.sh
	$ sudo salt-call --local state.apply helloworld
	local:
	----------
	          ID: /usr/local/bin/helloworld.sh
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/helloworld.sh is in the correct state
	     Started: 18:01:28.923689
	    Duration: 16.872 ms
	     Changes:   
	
	Summary for local
	------------
	Succeeded: 1
	Failed:    0
	------------
	Total states run:     1
	Total run time:  16.872 ms

I had the command already on my machine so there was no changes made. I just needed to test if the command works

	$ helloworld.sh 
	Hello World!

The command works but I still had to copy permissions of the command

	$ stat /usr/local/bin/helloworld.sh 
		  File: /usr/local/bin/helloworld.sh
		  Size: 36        	Blocks: 8          IO Block: 4096   regular file
		Device: 801h/2049d	Inode: 156132      Links: 1
		Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
		Access: 2022-11-23 13:55:49.497758803 +0000
		Modify: 2022-11-23 13:59:57.685751985 +0000
		Change: 2022-11-23 13:59:57.685751985 +0000
		 Birth: 2022-11-23 13:55:49.497758803 +0000

Only thing needed was the numeric value of the files permissions and I needed to add that to the sls file

	$ sudoedit init.sls 
	$ cat init.sls 
	/usr/local/bin/helloworld.sh:
	  file.managed:
	    - source: salt://scripts/helloworld.sh
	    - mode: '0755'	

Next I needed to delete helloworld.sh from /usr/local/bin/ and test if I can still run the command

	$ sudo rm /usr/local/bin/helloworld.sh
	$ helloworld.sh
	bash: helloworld.sh: command not found

The command didn't work anymore so I needed to apply the state again and test if the command works

	$ sudo salt-call --local state.apply helloworld
	local:
	----------
	          ID: /usr/local/bin/helloworld.sh
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/helloworld.sh updated
	     Started: 18:14:26.394896
	    Duration: 19.779 ms
	     Changes:   
	              ----------
	              diff:
	                  New file
	              mode:
	                  0755
	
	Summary for local
	------------
	Succeeded: 1 (changed=1)
	Failed:    0
	------------
	Total states run:     1
	Total run time:  19.779 ms
	$ helloworld.sh 
		Hello World!

My command works and I only needed to check the permissions

	$ ls -l /usr/local/bin/helloworld.sh 
	-rwxr-xr-x 1 root root 36 Nov 23 18:14 /usr/local/bin/helloworld.sh

## b) Make a new command which tells the date and time and install it with salt

I went to the directory where I want to store my scripts in my home directory

	$ cd scripts/
	$ pwd
	/home/tatu/scripts

I checked current date which I also wanted to do with my new command

	$ date
	Wed 23 Nov 18:22:23 GMT 2022

I made a new file and tried to run it with bash

	$ micro whatsup.sh
	$ cat whatsup.sh 
	date
	$ bash whatsup.sh 
	Wed 23 Nov 18:23:47 GMT 2022

I needed to add information that I want the command to be run in bash to the top of the file

	$ micro whatsup.sh 
	$ cat whatsup.sh 
	#!/usr/bin/bash
	date

And change the permissions to make the command executable

	$ chmod ugo+x whatsup.sh 
	$ ls -l whatsup.sh 
	-rwxr-xr-x 1 tatu tatu 21 Nov 23 18:24 whatsup.sh

Next I needed to copy the command where it can be found by the target machine

	$ sudo cp whatsup.sh /usr/local/bin/

And I tested if the command works by moving into another directory and running the command

	$ cd /etc/
	$ whatsup.sh
	Wed 23 Nov 18:27:56 GMT 2022

I needed to install it to a machine with salt so I needed to make a new directory and a new file where I can store the state

	$ cd /srv/salt/
	$ sudo mkdir whatsup
	$ sudo micro whatsup/init.sls

I needed to copy the command for my salt to use

	$ sudo cp /usr/local/bin/whatsup.sh /srv/salt/scripts/

And write the state where I install whatsup.sh to a target machine

	$ cd whatsup/
	$ sudoedit init.sls 
	$ cat init.sls 
	/usr/local/bin/whatsup.sh:
	  file.managed:
	    - source: salt://scripts/whatsup.sh
	    - mode: '0755'
	
I tested if the state works

	$ sudo salt-call --local state.apply whatsup
	local:
	----------
	          ID: /usr/local/bin/whatsup.sh
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/whatsup.sh is in the correct state
	     Started: 18:34:14.197225
	    Duration: 16.694 ms
	     Changes:   
	
	Summary for local
	------------
	Succeeded: 1
	Failed:    0
	------------
	Total states run:     1
	Total run time:  16.694 ms
	$ whatsup.sh 
	Wed 23 Nov 18:34:50 GMT 2022
	
Now I needed to delete the file from /usr/local/bin/ and test if I can still use the command

	$ sudo rm /usr/local/bin/whatsup.sh 
	$ whatsup.sh 
	bash: whatsup.sh: command not found

And apply the state and test the command again

	$ sudo salt-call --local state.apply whatsup
	local:
	----------
	          ID: /usr/local/bin/whatsup.sh
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/whatsup.sh updated
	     Started: 18:35:55.125205
	    Duration: 18.739 ms
	     Changes:   
	              ----------
	              diff:
	                  New file
	              mode:
	                  0755
	
	Summary for local
	------------
	Succeeded: 1 (changed=1)
	Failed:    0
	------------
	Total states run:     1
	Total run time:  18.739 ms
	$ whatsup.sh 
	Wed 23 Nov 18:35:58 GMT 2022

Last I wanted to check the permissions of the commaand

	$ ls -l /usr/local/bin/whatsup.sh 
	-rwxr-xr-x 1 root root 21 Nov 23 18:35 /usr/local/bin/whatsup.sh

## c) Make a new command with python and install it to a target machine with salt

I made a new script to my home directory and tested if it works

	$ micro hello.py
	$ cat hello.py 
	print("Hello Tatu")
	$ python3 hello.py 
	Hello Tatu

I changed the permissions and defined in the file that it has to be run in python3

	$ micro hello.py
	$ cat hello.py 
	#!/usr/bin/pyhton3
	print("Hello Tatu")
	$ chmod ugo+x hello.py
	$ ls -l hello.py
	-rwxr-xr-x 1 tatu tatu 39 Nov 23 18:40 hello.py

I copied the file to a directory where my machine can find it and tested if it works from any directory

	$ sudo cp hello.py /usr/local/bin/
	cd /etc/
	$ hello.py 
	Hello Tatu

I needed to make a new directory and file to store the state where salt can install this script to a target machine

	$ cd /srv/salt/
	$ sudo mkdir hello
	$ cd hello
	$ sudo micro init.sls

I copied the command where salt can access it and wrote the state I wanted the target machine to get to

	$ sudo cp /usr/local/bin/hello.py /srv/salt/scripts/
	$ sudoedit init.sls
	$ cat init.sls
	/usr/local/bin/hello.py:
	  file.managed:
	    - source: salt://scripts/hello.py
	    - mode: '0755'

I tried to run the state

	$ sudo salt-call --local state.apply hello
	local:
	----------
	          ID: /usr/local/bin/hello.py
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/hello.py is in the correct state
	     Started: 18:53:47.643906
	    Duration: 16.168 ms
	     Changes:   
	
	Summary for local
	------------
	Succeeded: 1
	Failed:    0
	------------
	Total states run:     1
	Total run time:  16.168 ms

I removed hello.py from /usr/local/bin/ and tried to run the command again

	$ sudo rm /usr/local/bin/hello.py 
	$ hello.py
	bash: hello.py: command not found

Next I needed to apply the state again and run the command again to see if it works

	$ sudo salt-call --local state.apply hello
	local:
	----------
	          ID: /usr/local/bin/hello.py
	    Function: file.managed
	      Result: True
	     Comment: File /usr/local/bin/hello.py updated
	     Started: 18:56:03.130156
	    Duration: 40.438 ms
	     Changes:   
	              ----------
	              diff:
	                  New file
	              mode:
	                  0755
	
	Summary for local
	------------
	Succeeded: 1 (changed=1)
	Failed:    0
	------------
	Total states run:     1
	Total run time:  40.438 ms
	$ hello.py 
	Hello Tatu

My new command worked but I wanted to check the permissions of that command

	$ ls -l /usr/local/bin/hello.py 
	-rwxr-xr-x 1 root root 39 Nov 23 18:56 /usr/local/bin/hello.py

## d) Make a directory and where you can copy all scripts to the target machine

In the previous exercises I had made a directory called "scripts" with 3 commands inside in my home directory so I used that.
Same scripts were also already saved in /srv/salt/scripts

I made a new directory to store salt state and added a file where the state could be applied from

	$ sudo mkdir copyscripts
	$ cd copyscripts/
	$ sudo touch init.sls
	
I needed to write the state I wanted the target machine to get to

	$ sudoedit init.sls 
	$ cat init.sls 
	/usr/local/bin/:
	  file.recurse:
	    - source: salt://scripts/
	    - file_mode: '0755'

And I tested to apply the state

	$ sudo salt-call --local state.apply copyscripts
	local:
	----------
	          ID: /usr/local/bin/
	    Function: file.recurse
	      Result: True
	     Comment: The directory /usr/local/bin/ is in the correct state
	     Started: 19:39:23.098075
	    Duration: 19.519 ms
	     Changes:   
	
	Summary for local
	------------
	Succeeded: 1
	Failed:    0
	------------
	Total states run:     1
	Total run time:  19.519 ms

The state was successfull but I needed to test if the scripts would still work

	$ hello.py
	Hello Tatu
	$ helloworld.sh 
	Hello World!
	$ whatsup.sh 
	Wed 23 Nov 19:40:55 GMT 2022

All of the scripts worked but I still needed to remove them and see if they can be installed with salt

	$ cd /usr/local/bin/
	$ ls
	hello.py  helloworld.sh  whatsup.sh
	$ sudo rm hello.py helloworld.sh whatsup.sh
	$ ls

After I had removed all of the scripts I applied the state again

	$ sudo salt-call --local state.apply copyscripts
	local:
	----------
	          ID: /usr/local/bin/
	    Function: file.recurse
	      Result: True
	     Comment: Recursively updated /usr/local/bin/
	     Started: 19:46:31.369815
	    Duration: 42.648 ms
	     Changes:   
	              ----------
	              /usr/local/bin/hello.py:
	                  ----------
	                  diff:
	                      New file
	                  mode:
	                      0755
	              /usr/local/bin/helloworld.sh:
	                  ----------
	                  diff:
	                      New file
	                  mode:
	                      0755
	              /usr/local/bin/whatsup.sh:
	                  ----------
	                  diff:
	                      New file
	                  mode:
	                      0755
	
	Summary for local
	------------
	Succeeded: 1 (changed=1)
	Failed:    0
	------------
	Total states run:     1
	Total run time:  42.648 ms

I still needed to check if the commands work

	$ hello.py 
	Hello Tatu
	$ helloworld.sh 
	Hello World!
	$ whatsup.sh 
	Wed 23 Nov 19:49:29 GMT 2022

All of the commands worked but I still wanted to see their permissions

	$ ls -l /usr/local/bin/
	total 12
	-rwxr-xr-x 1 root root 39 Nov 23 19:46 hello.py
	-rwxr-xr-x 1 root root 36 Nov 23 19:46 helloworld.sh
	-rwxr-xr-x 1 root root 21 Nov 23 19:46 whatsup.sh

## e)

## f)
