## Lab introduction : SSL/TLS Termination
- We have a Flask App, which has API to show students marks based on backend mysql students database.
- This flask App is insecure and handles http requests.
- In front we will start a secured web server, which will handle all incoming requests -- hence the name ingress proxy
- We are using nginx as a ingress proxy -- name=nginx-ingress-proxy
- This ingress-proxy will take care of ssl communication (securing the communication) -- between client and insecure server -- hence this setup is called SSL/TLS Termination

### Flow
```
user ---secured ---> nginx (ingress-proxy https://0.0.0.0:5443) --->unsecured --> app (flask webserver http://127.0.0.1:5000)
user <---secured --- nginx (ingress-proxy https://0.0.0.0:5443) <---unsecured --- app (flask webserver http://127.0.0.1:5000)
```

## High level steps
- Start insecure http web server `on port 5000 and listening on 127.0.0.1`
- Create keypairs. `(file-1) private key=server.key, (file-2) certificate=server.crt`
- Configure nginx ingress proxy `using the keypairs, port 5443 and listening on 0.0.0.0`

### Theory in my own words:
- The client initiates the communication by sending a "Hello" request to the server. 
- The server presents its certificate and the client verifies it. 
- It checks for a signature by a well known CA (certifying authority), its validity and if it's for the correct domain.
- After the verification, the client generates a random dynamic symmetric key which it encryptes with the public key of the server. 
- The server decodes it with its private key, and an agreement is made between the two. 
- All further messages are coded and encoded with the mutually agreed symmetric key.

### AI Theory 1: Establishing a Secure Connection
- The connection begins when the client initiates a handshake by sending a greeting to the server.
- In response, the server presents its digital SSL/TLS certificate to prove its identity.
- The client thoroughly verifies this certificate by ensuring it is cryptographically signed by a trusted Certificate Authority (CA), is currently valid, and matches the requested domain name.
- Once authentication is complete, the client generates a random Pre-Master Secret, encrypts it using the server's public key, and transmits it.
- The server decrypts this message using its corresponding private key.
- With the parameters successfully negotiated, both parties independently derive a shared symmetric session key, which is used to encrypt and decrypt all subsequent data transmissions.

### AI Theory 2: Ingress Proxy
- The TLS handshake is rarely handled by the application code itself.
- Instead, it is offloaded to an Ingress Proxy which acts as the gateway to the actual Application Server.
- The Ingress Proxy sits at the edge of the server infrastructure (handles incominng requests)
- The Ingress Proxy holds the SSL/TLS Certificate and the Private Key.
- It performs the verification steps, decrypts the Pre-Master Secret, and establishes the symmetric session keys with the client.
- This ingress-proxy will take care of ssl communication (securing the communication) -- between client and insecure server -- hence this setup is called SSL/TLS Termincation


### Create keypairs (1) private key=server.key, (2) certificate=server.crt
```
mkdir certs
cd certs

openssl req -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt

manasvi@mango certs % ls -lrt
total 16
-rw-------  1 manasvi  staff  1704 Jun 19 20:15 server.key
-rw-r--r--  1 manasvi  staff  1306 Jun 19 20:17 server.crt
manasvi@mango certs % 
```
#### Output
```
manasvi@mango certs % openssl req -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt
.+........+....+..+.+.........+...+...........+.+......+++++++++++++++++++++++++++++++++++++++*......+++++++++++++++++++++++++++++++++++++++*.+..+...+......+..........+.....+...............+..........+.....+......+............+.+............+..+.......+........+.+......+......++++++
.....+...+........+++++++++++++++++++++++++++++++++++++++*...........+....+..+.+..............+.+..+...+...+......+.+++++++++++++++++++++++++++++++++++++++*................+........+.+...+...........+.+......+...+.....+.+..................+........+...+....+..+.+.....+....+.....+.........+.+........+.+...............+.....+......+...+.+.....+......+.+..+.......+...+......+...........+...+...+.........+...+..........+...+..+.............+........+......................+.....+....+...+...+...+.....+....+..+.+............+..++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:IN
State or Province Name (full name) [Some-State]:MH
Locality Name (eg, city) []:mumbai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:mango
Organizational Unit Name (eg, section) []:it
Common Name (e.g. server FQDN or YOUR name) []:mango.in
Email Address []:
```

### Check the certificate
```
openssl x509 -in server.crt -text -noout
```

### Check the private key
```
openssl rsa -in server.key -text -noout
```

### Note
- server.crt is an X.509 certificate (not a raw public key, but contains public key)
- server.key is your private key.

### Extract the public key from certificate
```
openssl x509 -in server.crt -pubkey -noout > server_pub.pem
```

### Verify the cert and key actually match
```
openssl x509 -in server.crt -modulus -noout | openssl md5
openssl rsa -in server.key -modulus -noout | openssl md5

manasvi@mango certs % openssl x509 -in server.crt -modulus -noout | openssl md5
MD5(stdin)= 9dc55474a52ed65e93787db940005f0c
manasvi@mango certs % openssl rsa -in server.key -modulus -noout | openssl md5
MD5(stdin)= 9dc55474a52ed65e93787db940005f0c
manasvi@mango certs % 

```

### Small Demo of Encryption and Decryption 
```
echo -n "Hello World" | openssl pkeyutl -encrypt -pubin -inkey server_pub.pem -out /tmp/secret.enc
openssl pkeyutl -decrypt -inkey server.key -in /tmp/secret.enc -out /tmp/secret.dec

manasvi@mango certs % cat /tmp/secret.dec 
Hello World%                                                                                                                                                  
manasvi@mango certs % 
```


### Configure ingress proxy
- Add below lines in `/usr/local/etc/nginx/nginx.conf`
```
server {
    listen 5443 ssl;
    server_name mango.in;
    
    ssl_certificate    /Users/manasvi/git/lab/3_webapp/certs/server.crt;
    ssl_certificate_key /Users/manasvi/git/lab/3_webapp/certs/server.key;
    
    location / {
        proxy_pass http://127.0.0.1:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

### Test
|                                 | secure? | Identity checked ? | via ingress proxy? | request reached app ?                                        | Result  |
|---------------------------------|---------|--------------------|--------------------|--------------------------------------------------------------|---------|
| curl http://127.0.0.1:5000/     | No      | No                 | No (bypasses)      | Yes                                                          | Success |
| curl https://127.0.0.1:5000/    | Yes     | No                 | No (bypasses)      | Yes - but fails, since C using secure method with unsecure S | Fails   |
| curl https://127.0.0.1:5443/    | Yes     | No                 | Yes                | No  - C fails to verify identity cert                        | Fails   |
| curl http://127.0.0.1:5443/     | No      | No                 | Yes                | No  - error: The plain HTTP request was sent to HTTPS port   | Fails   |
| curl -k https://127.0.0.1:5443/ | Yes     | Yes(forced)        | Yes                | Yes                                                          | Success |
| curl -k http://127.0.0.1:5443/  | No      | No                 | Yes                | No  - client is using insecure method                        | Fails   |


### History
```
  653  curl http://127.0.0.1:5000/math-toppers
  654  curl https://127.0.0.1:5000/math-toppers
  655  curl https://127.0.0.1:5443/math-toppers
  659  curl http://127.0.0.1:5443/math-toppers
  660  curl -k https://127.0.0.1:5443/math-toppers
  661  curl -k http://127.0.0.1:5443/math-toppers
```
