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

| Code | Name                | Description                   |
| ---- | ------------------- | ----------------------------- |
| 100  | Continue            | السيرفر وافق تكمل إرسال الطلب |
| 101  | Switching Protocols | السيرفر بيغير البروتوكول      |
| 102  | Processing          | السيرفر لسه بيعالج الطلب      |

## Success

| Code | Name            | Description             |
| ---- | --------------- | ----------------------- |
| 200  | OK              | الطلب نجح               |
| 201  | Created         | تم إنشاء resource جديد  |
| 202  | Accepted        | الطلب اتقبل ولسه بيتنفذ |
| 204  | No Content      | مفيش محتوى في الرد      |
| 206  | Partial Content | جزء من البيانات         |


## Redirection

| Code | Name               | Description       |
| ---- | ------------------ | ----------------- |
| 300  | Multiple Choices   | أكتر من اختيار    |
| 301  | Moved Permanently  | نقل دائم          |
| 302  | Found              | إعادة توجيه مؤقتة |
| 303  | See Other          | تحويل لـ URL تاني |
| 304  | Not Modified       | الكاش لسه صالح    |
| 307  | Temporary Redirect | redirect مؤقت     |
| 308  | Permanent Redirect | redirect دائم     |

## Client Errors

| Code | Name                          | Description         |
| ---- | ----------------------------- | ------------------- |
| 400  | Bad Request                   | الطلب فيه خطأ       |
| 401  | Unauthorized                  | محتاج تسجيل دخول    |
| 402  | Payment Required              | نادر                |
| 403  | Forbidden                     | مفيش صلاحية         |
| 404  | Not Found                     | مش موجود            |
| 405  | Method Not Allowed            | method مش مسموح     |
| 406  | Not Acceptable                | مش مناسب            |
| 407  | Proxy Authentication Required | محتاج auth للبروكسي |
| 408  | Request Timeout               | الطلب اتأخر         |
| 409  | Conflict                      | تعارض               |
| 410  | Gone                          | اتحذف نهائي         |
| 413  | Payload Too Large             | الداتا كبيرة        |
| 414  | URI Too Long                  | الرابط طويل         |
| 415  | Unsupported Media Type        | نوع مش مدعوم        |
| 429  | Too Many Requests             | rate limit          |

## Server Errors

| Code | Name                       | Description           |
| ---- | -------------------------- | --------------------- |
| 500  | Internal Server Error      | خطأ في السيرفر        |
| 501  | Not Implemented            | مش مدعوم              |
| 502  | Bad Gateway                | مشكلة بين سيرفرات     |
| 503  | Service Unavailable        | السيرفر واقع          |
| 504  | Gateway Timeout            | مفيش رد من سيرفر تاني |
| 505  | HTTP Version Not Supported | version مش مدعوم      |

---

