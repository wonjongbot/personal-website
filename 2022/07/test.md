# Zerg-Rush

Zerg-Rush is a pentesting tool for Denial of Service (DoS) attacks in TCP networks, primarily to test vulnerabilties of industrial control systems (ICS). 

## Installation / dependencies

This project was programmed on Python 3.10. Any version lower than 3.10 would not work. Either create a new environment on conda or update python

This project is also heavily based on scapy python library. Please install scapy with following command:

``` console
pip install --pre scapy[basic]
```

## Usage

The program requires root privelidge to send packets.

Run the program by executing main.py in sudo.

Use -h argument to view help message

``` console
python3 main.py -h
python3 main.py --help
```

main.py takes three arguments, where respectively be used as attacker ip address, target ip address, and target port:

``` console
python3 main.py [attacker IP] [target IP] [target port]
```

example:

``` console
python3 main.py 192.168.0.1 192.168.0.2 5555
```

However, if no arguments are entered, all three arguments are automatically assigned as NULL

no commandline arguments sets all three values as NULL, which must be changed in the menu.

## Menu Navigation

Zerg Rush offers CLI menu navigation to select types of attack and modify attack configurations. A menu screen would look something like below:

``` console
    +---------------------------------------------------------------------------+
    |   __________                           __________             .__         |
    |   \____    /___________  ____          \______   \__ __  _____|  |__      |
    |     /     // __ \_  __ \/ ___\   ______ |       _/  |  \/  ___/  |  \     |
    |    /     /\  ___/|  | \/ /_/  > /_____/ |    |   \  |  /\___ \|   Y  \    |
    |   /_______ \___  >__|  \___  /          |____|_  /____//____  >___|  /    |
    |           \/   \/     /_____/                  \/           \/     \/     |
    +---------------------------------------------------------------------------+
    |           Welcome to Zerg rush, a simple network attacking tool.          |
    +---------------------------------------------------------------------------+
    |                                                       Peter Lee, UIUC 2022|
    |                                                      wonjong3@illinois.edu|
    |                                                 peterwlee.web.illinois.edu|
    |                                                      Github.com/wonjongbot|
    +---------------------------------------------------------------------------+
    
[*] Attack information:
    Attacker IP: NULL
    Target IP: NULL
    Target port number: NULL

[!] Attacker IP address is NULL. Please use opiton 3 to enter argument.
[!] Target IP address is NULL. Please use opiton 3 to enter argument.
[!] Target port is NULL. Please use opiton 3 to enter argument.

[*] Please select options below:
    1. Modify attacker IP address
    2. Modify target IP address
    3. Modify target port number
    4. SYN flood attack
    5. ACK flood attack
    6. Long packet attack
    7. Malformed HTTP request
    8. Fragmented UDP packet with malformed offset
    9. Fragmented TCP packet with malformed offset
    10. ARP spoofing for MIM
    11. Telnet long string attack

zRush > 
```

## Attacks

When TCP/IP connection is initilaized, 3-way handshake is performed between the server and the client. 

### _SYN flood attack_

* In SYN flood mode, the attacker sends spoofed, raw SYN packets to the target indefinitely. The target will respond by sending SYN/ACK packet, which the attacker will disregard. Therefore, the target computer will initiate half-open connections until its resource is depleted. 

### _ACK flood attack_

* In ACK flood mode, the attacker establishes a full connection to the target by sending ACK packet after receiving SYN/ACK from the target. The attacker continues this indefinitely, which prevents the target from serving legitimate users. To perform ACK flood attack, the attacker's kernal must disregard SYN/ACK packet received as kernal will respond with RST to the connections it did not initiate (main.py did!). Therefore attacker's machine must filter out / drop any outgoing RST packets. Such setting can be configured using iptables.

  ```console
  sudo iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP
  ```

* However, Zerg-Rush already runs this command whenever ACK flood attack is initiated. Also, the filtering above is reverted once the user exits the program.

### _Long string flooding_

* Sends very long string to designated IP and PORT

### _HTTP malformed request_

* Sends malformed req through HTTP

### _Teardrop(UDP)_

* Sends fragmented UDP with malformed offset

### _Teardrop(TCP)_

### _ARP Spoofing_

### _Telnet long string flooding_

## Achievements

Here are some notes from experiements performed with Zerg-rush

### ABB REF615 Feeder Protection and Control Relay

#### HTTP server

_SYN flood attack_:

* Target port: 80
* Affect: HTTP server crashed

_ACK flood attack_:

* Target port: 80
* Affect: HTTP server crashed

_long string attack_:

* Target port: 80
* Affect: when attack runs, program crashes after returning broken pipe error, indicating that the server cut the connection off. No noticeable affect on WHMI page of the relay. Wireshark shows that the relay sends RST to the attacker.

_malformed HTTP request_:

* Target port: 80
* experiment: very long request form
  * Normal HTTP request structure:

    ``` console
    GET [PATH] HTTP/1.1\r\nHost: [HOST]\r\n\r\n
    ```

  * Test 1 - long string:
    * Sending very long, no carriage return character
      * request sent:

        ``` console
        GET [PATH]XXXXXXXXX[...]XXXXXXXXX HTTP/1.1\r\nHost: [HOST]\r\n\r\n
        ```

      * effect: 400 error
      * server response:

        ``` console
        b'E\x00\x00j\x01\xb6\x00\x00@\x06\xf5g\xc0\xa8\x01\x0f\xc0\xa8\x01\x11\x00P\xf2\xf5\xe8\xd3YG\x00\x00\x04>P\x18\x12{5\xe3\x00\x00HTTP/1.1 400 Bad Request\r\nContent-Length: 0\r\nConnection: close\r\n\r\n'
        ```

    * Test 2 - relative path:
      * Sending a malformed request that has relative path to traverse into parent folders
      * request sent:

        ``` console
        GET [PATH]/../../ HTTP/1.1\r\nHost: [HOST]\r\n\r\n
        ```

      * effect: 404 error
      * server response:

        ``` console
        b'E\x00\x00\x8a\x01\xc5\x00\x00@\x06\xf58\xc0\xa8\x01\x0f\xc0\xa8\x01\x11\x00P@\xc3\x934\xb0\xf1\x00\x00\x00EP\x18\x16t\xc4\x1a\x00\x00HTTP/1.1 404 Not Found\r\nContent-Type: text/html\r\nTransfer-Encoding: chunked\r\nConnection: close\r\n\r\n'
        ```

### SEL-751 Feeder Protection Relay

#### FTP server

_SYN flood attack_:

* Target port: 21
* Affect: FTP server seems to be slowed down. Sometimes returns this message before correctly returning command

``` command
229 Entering Extended Passive Mode (|||PORT NUM|).
```

_ACK flood attack_:

* Target port: 21
* Affect: FTP server crashed with message like below

> 421 Service not available, remote server has closed connection.
226 Closing data connection.

_Long String Attack_:

* Target port: 21
* running two processes of this attacks blocks other users from connecting through FTP because SEL only allows 2 ftp connections
* I can hear relay clicks whenever one fails to log into FTP.

## Future Goals