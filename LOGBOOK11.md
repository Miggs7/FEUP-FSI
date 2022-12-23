# SEED labs - PKI 

## Setup

First build the composer image with 'docker-compose build' or its alias 'dcbuild'

![dcbuild](images/logbook11/dcbuild.png)

Then you need to run 'docker-compose up' or its alias 'dcup' to have the container running in the background.

![dcup](images/logbook11/1.png)

List of container ID's:

![dockps](images/logbook11/2.png)

Using 'docksh' and the first few characters of the container ID will give you access to the shell of the container.

![docksh](images/logbook11/3.png)

We need to set up a HTTPS web server with a name, so we add the following entries to 'etc/hosts'

```shell
10.9.0.80 www.bank32.com
10.9.0.80 www.fsi2022.com
```
## Task 1 - Becoming a Certificate Authority (CA)

We need to change openssl configuration, so we will copy the configuration file to our current directory and instruct OpenSSL to use this copy

```
cp  /usr/lib/ssl/openssl.cnf .
```
Uncommented 'unique_subject' parameter, just like requested.

![task1_2](images/logbook11/4.png)


We generate a CA with the following command

```
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
-keyout ca.key -out ca.crt
```

After we fill the asked information the output of the command will be stored in two files: ca.key and ca.crt.

The ca.key contains the private key, while ca.crt contains the public-key certificate.

![ca.crt](images/logbook11/5.png)

### Questions

1. What part of the certificate indicates this is a CA’s certificate?

    Basic Constrainst, flag identifying certificate is CA.

![basic_constraints](images/logbook11/6.png)

1. What part of the certificate indicates this is a self-signed certificate?

    Issuer and subject of the certificate are equal.

![issuer_subject](images/logbook11/7.png)

1. In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret numbers p and q, such that n = pq. Please identify the values for these elements in your certificate and key files.

* Modulus

![modulus](images/logbook11/8.png)

* Exponents

![exponenets](images/logbook11/9.png)

* Primes

![primes](images/logbook11/10.png)


Observation: Answers for question 1 and 2 come from the ouput of the following command:
```
openssl x509 -in ca.crt -text -noout
```

And question 3 comes from this command:

```
openssl rsa -in ca.key -text -noout
```

## Task 2: Generating a Certificate Request for Your Web Server

We want to generate a CSR for www.fsi2022.com, to give it a public-key certificate from our CA. <br>
The CSR will be sent to the CA, who will
verify the identity information in the request, and then generate a certificate.

The following command will generate a CSR for www.fsi2022.com

```
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.fsi2022.com/O=fsi2022 Inc./C=PT" \
-passout pass:dees -addext "subjectAltName = DNS:www.fsi2022.com, \
                          DNS:www.fsi2022A.com, \
                          DNS:www.fsi2022B.com"
```

![Terminal print task2](images/logbook11/task2.png)

The command will generate a pair of public/private key then create a signing request from the public key.

The following command will look at the decoded content of the CSR and private key files:

```
openssl req -in server.csr -text -noout
openssl rsa -in server.key -text -noout
```
### Add alternative names
Adding the following option to the CSR generator command we will add alternative names to my certifcate signing request: 
````
-addext "subjectAltName = DNS:www.fsi2022.com, \
                          DNS:www.fsi2022A.com, \
                          DNS:www.fsi2022B.com"
````

## Task 3

Now, we can turn the CSR created previously into a certficate 'server.crt', using the CA's 'ca.crt' and 'ca.key' by running the following command:

```
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

The above command :

* 'myCA_openssl.cnf' is the configuration file we copied from /usr/lib/
ssl/openssl.cnf and made some changes in task 1
* The default setting in 'myCA_openssl.cnf' does not allow 'openssl ca' so we need to enable that by uncommenting the following line:

```
# Extension copying option: use with caution.
copy_extensions = copy
```
After editing 'myCA_openssl.cnf' we transform the CSR into a certificate

![task3](images/logbook11/task3.png)

## Task 4 - Deploying Certificate in an Apache-Based HTTPS Website

We will deployed our certicate into an apache-based HTTPS based website, hosted in a docker container:
    
* Configure 'bank32_apache_ssl.conf' so that it corresponds to our server's data and mover server.crt and server.key to the right path.
```
<VirtualHost *:443> 
    DocumentRoot /var/www/bank32
    ServerName www.fsi2022.com
    ServerAlias www.fsi2022A.com
    ServerAlias www.fsi2022B.com
    DirectoryIndex index.html
    SSLEngine On 
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>

<VirtualHost *:80> 
    DocumentRoot /var/www/bank32
    ServerName www.fsi2022.com
    DirectoryIndex index_red.html
</VirtualHost>

# Set the following gloal entry to suppress an annoying warning message
ServerName localhost

# Set the following gloal entry to suppress an annoying warning message
ServerName localhost
```
* Build and Launch the docker container<br>
![Terminal print 1](images/logbook11/1.png)

After acessing container root (like in the setup section of this logbook) you copy certificate and key from the volumes directory
```
root@cf2ba8dc58e9:/certs# cp /volumes/server.crt .
root@cf2ba8dc58e9:/certs# cp /volumes/server.key .
```

* Enable the SSL module and the sites described in this file with following commands when the container is built:

```
# a2enmod ssl // Enable the SSL module
# a2ensite bank32_apache_ssl // Enable the sites described in this file
```

* Start the apache server with the following commands in the container:

```
// Start the server
# service apache2 start
```
![apache_start](images/logbook11/apachestart.png)

If we go to the browser and go the URL 'https://www.fsi2022.com/' we will see a warning instead of the 'Hello, world!' page, this is due to the fact that our browser in this case Firefox doesn't know the Issuer.

![issuer](images/logbook11/task4_1.png)

After we load the certicate into Firefox (file 'ca.crt', which is public-key certificate) we will be able to visit the desired webpage and Firefox will recognize it as safe.

![import_CA](images/logbook11/task4_2.png)

![website](images/logbook11/task4_3.png)

The certificate will be the one we created before as you can see here in this screenshot.

![certificate](images/logbook11/task4_4.png)


# Task 5 - Launching a Man-In-The-Middle Attack

We want to prove how strong is PKI against MITM attacks, so we will setup a malicious website 'www.example.com' to impersonate 'www.fsi2022.com'.

First we change the servername to 'www.example.com' in the bank32_apache_ssl file, like this:
```
<VirtualHost *:443> 
    DocumentRoot /var/www/bank32
    ServerName www.example.com
    ServerAlias www.fsi2022A.com
    ServerAlias www.fsi2022B.com
    DirectoryIndex index.html
    SSLEngine On 
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>

<VirtualHost *:80> 
    DocumentRoot /var/www/bank32
    ServerName www.fsi2022.com
    DirectoryIndex index_red.html
</VirtualHost>

# Set the following gloal entry to suppress an annoying warning message
ServerName localhost
```
Then we modify the victims machine's '/etc/hosts' file to emulate the result of a DNS cache positing attack my mapping the hostname 'www.example.com' to our 'malicious web server'.

![Terminal print task5_1](/images/logbook11/task5_1.png)

When we visit the page with URL 'https://example.com' we will see the following:

![Man_in_the_Middle](images/logbook11/task5_2.png)

This is due to the fact that the 'www.example.com' does not match any of the names in the CA, proving the efficiency of the use of PKI infrastructure against Man-In-The-Middle Attacks. 

# Task 6 - Launching a Man-In-The-Middle Attack with a Compromised CA

In this task we will demonstrate how is performed a sucessful MITHM attack.

First we change our apache configuration so that we can acess "example.com":

```
<VirtualHost *:443> 
    DocumentRoot /var/www/bank32
    ServerName www.example.com
    ServerAlias www.fsi2022A.com
    ServerAlias www.fsi2022B.com
    DirectoryIndex index.html
    SSLEngine On 
    SSLCertificateFile /certs/server2.crt
    SSLCertificateKeyFile /certs/server2.key
</VirtualHost>

<VirtualHost *:80> 
    DocumentRoot /var/www/bank32
    ServerName www.example.com
    DirectoryIndex index_red.html
</VirtualHost>

# Set the following gloal entry to suppress an annoying warning message
ServerName localhost

```
We proved in the previous task that if the CA isn't made for the 'https://example.com', the browser will give an alert making the attack harder to be sucessful since the user is already aware of the danger.

In our experience, we will assume that the attacker stole the CA's private key.
So we will be able to generate the certificate for our scam website 'www.example.com' to do this we have to use the command in Task 2 but specifying for 'www.example.com' :

```
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.example.com/O=Example Inc./C=PT" -passout pass:dees
```
![Terminal print task6_1](/images/logbook11/task6_1.png)

And then sign our CA with the following command:

```
openssl ca -config myCA_openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

![Terminal print task6_2](/images/logbook11/task6_2.png)

After that we will send the certs to the Docker and reload the apache server.

![Terminal print task6_3](/images/logbook11/task6_sucess.png)


This time we will be able to visit the web page because the CA was compromised and an attacker can sign the certicate and impersonate a website.


## CTF Challenges

## Challenge 1

For this challenge, we want to decypher some messages. To start we need to find the signal. In this challenge we can find it by using the command "nc ctf-fsi.fe.up.pt 6000" that gets us: "000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000022f9284cc3e7fd3e1024b88ca52b463943c1acbe4a69b6be3c9009c47fbfd62c3c3ccb45b2bcbcba6571a47cf1abc9552b9792bf4b46a6736196dc3140863ed4311e5250031dcf1192f5c35b9b403f384de31a79c3da7c2fbcfb31eca13b446ce6529ee964879615066affce3d6568d65fce999ee6e9d8ee4b23dcf1210ba258
"

Then, we write this encrypted signal on the variable enc_flag with b before the encrypted signal.
The e variable from the template is in binary and switching to decimal equals 65537.
The variable n is equal to the multiplication between p and q, which are two random large prime numbers while d is the private key as the inverse of e given private total.
```python
from binascii import hexlify, unhexlify
from sympy import *

p = nextprime(2512)
q = nextprime(2513)
n = p*q
total = (p-1)*(q-1)
e = 65537
d = pow(e, -1, total)


enc_flag = b"000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000022f9284cc3e7fd3e1024b88ca52b463943c1acbe4a69b6be3c9009c47fbfd62c3c3ccb45b2bcbcba6571a47cf1abc9552b9792bf4b46a6736196dc3140863ed4311e5250031dcf1192f5c35b9b403f384de31a79c3da7c2fbcfb31eca13b446ce6529ee964879615066affce3d6568d65fce999ee6e9d8ee4b23dcf1210ba258"

def enc(x):
    int_x = int.from_bytes(x, "big")
    y = pow(int_x,e,n)
    return hexlify(y.to_bytes(256, 'big'))

def dec(y):
    int_y = int.from_bytes(unhexlify(y), "big")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'big')

y = dec(enc_flag)
print(y.decode())
```
After we run the script with the changes done, we get the flag.

![Terminal print flag](/images/CTFs12/flag.png)

## Challenge 2

For this CTF we first get from the nc command 

![CTF](/images/logbook11/ctf2.png)

We will use the following script to obtain the flag :

```python
e1 = 65537
e2 = 65539
n = 29802384007335836114060790946940172263849688074203847205679161119246740969024691447256543750864846273960708438254311566283952628484424015493681621963467820718118574987867439608763479491101507201581057223558989005313208698460317488564288291082719699829753178633499407801126495589784600255076069467634364857018709745459288982060955372620312140134052685549203294828798700414465539743217609556452039466944664983781342587551185679334642222972770876019643835909446132146455764152958176465019970075319062952372134839912555603753959250424342115581031979523075376134933803222122260987279941806379954414425556495125737356327411

m1 = 0x490c77d8212053ed58bdb013b14e5a06f6b7a421c1f32cbe296f8800409277448e7a3defa6bb0106d6c3f59b6b684c92d392f49e7a3cb97fa6477b09de3e95321fcbc776e63d46cc7e2e3a64e7c375f7a6a4e0057eac29af2f8844dcb52798cc031e226e91102884eca63a8b30b41fec7f2692e2716341b739c54ae083866cd6070d55e9a6f42dc14cb8c593d51278b83aec0fad9a6456590c7f81238bec348227eda1ae06bb865e67fcf0b1782e321b683d9eac350430cb876099a9073ea165217d165661d20eb0014d5c328bcd56d7d67ad01dc8bd1e6e0e6d220069aab92bb46720502f5814ccf467b61d15146397fdb2f44e93e637113e4371427b5e7cd9

m2 = 0xa917ef1a3eda8ba23e3821293a0c20767352d341e566b05c4122dbcd511e1d7a05ef3cbb41e56a67badd9fd2a825fc66fa618a38bb848786fbb60bd3dea58885913a545776167838f4d939ee72b3cf1827cd970ed4c41c5cdad102297d62c86a657795bd0da66926f4d0d3f1a885a3ec6bc34e443cda28f39e16a11973abefd4aca66d0d25c4bdd3d24f23ebc5c25d1853b5eced7ec1c6e626a9b8a233eda5b46610fefc892b0596950c36218e3db386cea374398e07383f8fa9dcca9748a93bc4046cff19391f6934e94d6ef39127401a5eaa79e67e22edcb44a9a3a66c77342e27210c4ac5ab2fbb4754f8f322eafdc46b3b19f724f829c47206391fce465e


def gcdExtended(a, b):
    if a == 0:
        return b, 0, 1
    gcd, x1, y1 = gcdExtended(b % a, a)
    a = y1 - (b//a) * x1
    b= x1

    return gcd, a, b

g, a ,b = gcdExtended(e1, e2)

i = pow(m2, -1, n)
tmp1 = pow(m1, a)
tmp2 = pow(i, -b)
M = tmp1 * tmp2 % n

print(M.to_bytes(256,'big').decode())

```

![flag2](/images/logbook11/flag2.png)

In RSA to encrypt a message we do ```C = (M^e) mod n```, where ```C``` is the encrypted message, ```M``` the original message, ```e``` the public exponent and ```n``` the modules. We have ```C1=(M^e1) mod n``` and ```C2=(M^e2) mod n```.
If ```e1``` and ```e2``` are prime numbers, then ```mdc(e1,e2) = 1```, meaning the will be at least one ```a``` and one ```b``` where ```e1*a + e2*b+1 ``` . We will be able to find ```a``` and ```b``` using the extended version of the Euclides Algorithm.<br>
We can say that ```C1^a * C2^a == M^(e1*a) * M^(e^b) == M (mod n)```, then we can say that ```M = (C1^a)*(C2^b) mod n```.
If the value of ```a``` or ```b``` is negative then we will have problems with the equation above. In our case, ```b``` was negative and we considered ```i=(C2^-1) mod n ``` the modular inverse of ```C2```.<br>
Then it will be ```M=(C1^a)*(i^-b) mod n```.


