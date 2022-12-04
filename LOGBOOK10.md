
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
