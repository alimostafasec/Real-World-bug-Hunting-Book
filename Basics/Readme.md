# Essential Terminology
• Scope: The list of assets (websites, applications, IP addresses) you are legally allowed to test.

• Proof of Concept (PoC): A clear demonstration showing how to reproduce a vulnerability.

• Triage: The process of reviewing a submitted bug report to validate it.

• Severity: The level of impact a bug has (usually rated from Low to Critical).

• VDP (Vulnerability Disclosure Program): A program that offers recognition but typically not money.

• BBP (Bug Bounty Program): A program that offers monetary rewards (bounties).

# Basic Definitions

1. Network: A collection of interconnected devices (e.g., computers, servers) for communication and resource sharing.
2. Protocol: Rules and standards governing communication in a network.
3. IP Address: Unique numerical identifier for devices on a network.
4. Firewall: Monitors and controls network traffic, enforcing access policies.
5. DNS (Domain Name System): Translates domain names to IP addresses.
6. Clients: Devices requesting services/resources from servers.
7. Server: Stores and manages resources, responding to client requests.
8. Request-Response: Client-server interaction: request, process, respond.

# Most Common Protocols
| Protocol | Port Number | Description |
|----------|------------|-------------|
| HTTP | 80 | Hypertext Transfer Protocol |
| HTTPS | 443 | HTTP Secure |
| FTP | 21 | File Transfer Protocol |
| SSH | 22 | Secure Shell |
| Telnet | 23 | Telnet Protocol |
| SMTP | 25 | Simple Mail Transfer Protocol |
| DNS | 53 | Domain Name System |
| DHCP | 67/68 | Dynamic Host Configuration Protocol |
| POP3 | 110 | Post Office Protocol 3 |
| IMAP | 143 | Internet Message Access Protocol |
| RDP | 3389 | Remote Desktop Protocol |
| NTP | 123 | Network Time Protocol |
| SNMP | 161 | Simple Network Management Protocol |
| LDAP | 389 | Lightweight Directory Access Protocol |
| SMTPS | 465 | SMTP Secure |
| SIP | 5060/5061 | Session Initiation Protocol |
| FTPS | 990 | FTP Secure |

# Network Types
1. Local Area Network (LAN): Connects devices in a limited geographical area.
2. Wide Area Network (WAN): Spans large distances, connecting multiple LANs.
3. Metropolitan Area Network (MAN): Covers a larger geographic area than a LAN.
4. Campus Area Network (CAN): Interconnects LANs within a university campus or large organization.
5. Personal Area Network (PAN): Connects devices in close proximity to an individual.
6. Wireless Local Area Network (WLAN): LAN using wireless communication technologies.
7. Virtual Private Network (VPN): Provides a secure, encrypted connection over a public network.
8. Storage Area Network (SAN): Dedicated network providing high-speed access to shared storage devices.
9. Cloud Computing Network: Infrastructure used in cloud computing environments.
10. Internet: Global network of interconnected networks.

# The Lifecycle of a Web Request
When you use that URL, a rapid sequence of events happens.

1. First, a DNS Lookup translates the domain name into an IP address.
2. Next, your browser sends an HTTP Request to the server at that IP address, asking for the page's content.
3. The server then processes the request and sends back an HTTP Response, which contains the raw data for the site.
4. Finally, your browser begins Rendering and Execution. It parses the HTML to build the structure, applies the CSS to style it, and then executes the JavaScript code to add interactivity and build the final, dynamic webpage that you see.

# HTTP Status Codes 
A complete list of HTTP response status codes with simple explanations for bug bounty hunters.

## Informational

| Code | Name                 |
| ---- | -------------------  |
| 100  | Continue             |
| 101  | Switching Protocols  |
| 102  | Processing           |

## Success

| Code | Name             |
| ---- | --------------- |
| 200  | OK               |
| 201  | Created         |
| 202  | Accepted        |
| 204  | No Content      |
| 206  | Partial Content |


## Redirection

| Code | Name               |
| ---- | ------------------ |
| 300  | Multiple Choices   |
| 301  | Moved Permanently  |
| 302  | Found              |
| 303  | See Other          | 
| 304  | Not Modified       |
| 307  | Temporary Redirect |
| 308  | Permanent Redirect |

## Client Errors

| Code | Name                          |
| ---- | ----------------------------- | 
| 400  | Bad Request                   |
| 401  | Unauthorized                  |
| 402  | Payment Required              |
| 403  | Forbidden                     |
| 404  | Not Found                     |
| 405  | Method Not Allowed            |
| 406  | Not Acceptable                |
| 407  | Proxy Authentication Required |
| 408  | Request Timeout               |
| 409  | Conflict                      | 
| 410  | Gone                          |
| 413  | Payload Too Large             |
| 414  | URI Too Long                  |
| 415  | Unsupported Media Type        |
| 429  | Too Many Requests             |

## Server Errors

| Code | Name                       |
| ---- | -------------------------- |
| 500  | Internal Server Error      |
| 501  | Not Implemented            |
| 502  | Bad Gateway                |
| 503  | Service Unavailable        |
| 504  | Gateway Timeout            |
| 505  | HTTP Version Not Supported |

---

# HTTP: The Language of the Web
HTTP is the language browsers and servers use to communicate. Every HTTP Request is made up of a Method, like GET or POST; Headers, which contain metadata like cookies; and sometimes a Body with the data you're sending.

Common HTTP Methods:

- GET – Retrieve data (loading a page, fetching info)

- POST – Send data to the server (login forms, creating something)

- PUT/PATCH – Update existing data

- DELETE – Remove data

- OPTIONS – Check what methods are allowed (useful for recon)

In return, every HTTP Response includes a Status Code to indicate the result, along with its own headers and body.
