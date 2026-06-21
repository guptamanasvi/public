### Lab Introduction
- Here we use Python Flask App, and we start the app on different listening address.
- We test the client operation using different addresses -- `localhost, 127.0.0.1, and lan ip`
- We observe the result and save it in table below.

### Notes
- Well-known ports `0–1023` also known as system ports, and needs root(uid=0) privileges.
- Registered ports `1024-49151`
- Dynamic ports `49152-65535`

### High Level steps
- Start Python Flask App on different ports one after the other -- `localhost, 127.0.0.1, lan ip, and 0.0.0.0`
- We do three client operation during/when/with/for each of the above four server listening address.
- And we observe the result.
- We save the result in below table.

### Network Interfaces (Listening Addresses) 

| When listenning on ==> | localhost      | 127.0.0.1     | 192.168.1.3   | 0.0.0.0 |
|------------------------|----------------|---------------|---------------|---------|
| http://localhost       | Works          | Works         | Does not Work | Works   |
| http://127.0.0.1       | Works          | Works         | Does not Work | Works   |
| http://192.168.1.3     | Does not Work  | Does not Work | Works         | Works   | 


- **127.0.0.1 / localhost (The Loopback Loop)**: 
   - This is the computer's internal "mirror." (Never leaves the machine)
   - If the app listens here, only the user can access it from the same laptop.
- **192.168.1.3 (The Private LAN IP)**: 
   - This is the IP assigned to the machine by the connected Wi-Fi router. 
   - If the app listens here, anyone connected to the same Wi-Fi network can access it.
   - But the user can also access it locally using this IP.
- **0.0.0.0 (The "Listen to Everything" Wildcard)**: 
   - This tells the app: "Listen on every single network interface this computer has." 
   - It binds to 127.0.0.1 and the Wi-Fi IP simultaneously. 


Like a matchmaking game between the door the server unlocked (the Listening Address) and the address the client typed into the browser bar (the Request URL).

- **The Loopback Isolation (`localhost` / `127.0.0.1`):** 
  - When a server listens here, it is completely blind to the outside world and the local network.
  - Requesting `192.168.1.3` fails because the server refuses to look at the network card interface.
- **The LAN Bound Interface (`192.168.1.3`):** 
  - When a server listens here, it opens itself to the network but closes its internal loopback doors.
  - Requesting `localhost` or `127.0.0.1` fails because the server is actively ignoring loopback traffic.
- **The Wildcard Omnipresence (`0.0.0.0`):** 
  - This address never fails regardless of the request URL because it tells the operating system to duplicate the server's listening ear across **all** internal and external network paths.






