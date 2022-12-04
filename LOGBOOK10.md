
## CTF Challenges

### Challenge 1 

For this challenge, we found out that the website is vulnerable to XSS scripts, because after typing something in the text box, we are sent to a page where the admin can give the flag and mark the request as read. These buttons are only enabled to the admin, but we could enable them by inspecting element tool and changing disabled to enabled, so our purpose was to make the admin click the button Give Flag for us. For that, we 
typed the following code on the text box: 


![ctf](/images/logbook10/ctf1.png)

When the admin goes to evaluate the request, the script will click the button, and as it will be enabled, the flag will be given to us.
We used "giveflag" on the script because if we inspect the text box we get this id.

![ctf11](/images/ctf12.png)

Then, after waiting for a while, we got the flag 

![ctf1](/images/logbook10/ctf11.png)

### Challenge 2 

For this challenge, we have a black-boxed window.

We tried to ping google server by typing 8.8.8.8 and saw that this server was probably using a bash to run the unix command ping, so then we tried: 

```
; ls / -la 
```

The ; symbol was used at the beggining in order to add a command. And we got a list of all content including a folder called flags

![ctf2](/images/logbook10/ctf21.png)

Then, by using: 
```
; ls /flags
```
We were able to see the content inside, the flag.txt file.

![ctf22](/images/logbook10/ctf22.png)

All that was left was to display the content that was inside the flag.txt file by using the cat command
```
; cat flags/flag.txt
```

And voilá! We got the flags.

![ctf23](/images/logbook10/ctf25.png)
