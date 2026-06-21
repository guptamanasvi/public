### Lab introduction
- We have a Flask App, which has API to show students marks based on backend mysql students database.
- This flask App is insecure and handles http requests.
- In this demo, we will snoop the packets flowing between `client <-> "flask app web server"`
- And we will see the data being communicated between client and server.

### High level steps
- Start insecure http web server on port 5000 
- Start tcpdump, which will capture the packets being transferred between client and web server in a pcap file.
- Do client operation.
- Open the pcap file using wireshark.

### Start tcpdump, to capture packets flowing in/out from port 5000
```
sudo tcpdump -i lo0 -w flask.pcap 'tcp port 5000'
```

### Do client operation
```
curl http://127.0.0.1:5000/math-toppers
```

### Check the packets captured (you will have to control^C the tcpdump command)
- Using wireshark app, open the pcap file

![demo-wireshark-read-packets-during-insecure-communication.png](screenshots%2Fdemo-wireshark-read-packets-during-insecure-communication.png)

