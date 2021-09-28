---
title: DNS
date: 2019-05-01 12:23:43
tags: network
---
# DNS Principal

### Why DNS?

- The most communication on the Internet is based on TCP/IP, and TCP/IP is based on the IP address. However, it is impossible for people to memory the correct IP address for some websites. That's why DNS is introduced to provide a service to transfer the IP address to domain name as well as domain name to IP address.

### What is DNS?

- Domain Name System, which is in the application layer of TCP/IP. It is used to transfer between IP address to domain name.
  ![DNS][1]

### DNS Lookup Process

- DNS is a application layer protocol, and actually it is working for other application layer including HTTP and FTP.
- The process steps for DNS:
  - Each client has some cached DNS either in their browser or hosts or router.
  - When a url is entered in the browser, for example ``https://www.google.com.au/``, browser will retrieve the domain name from the Url and try to look for this domain name in the client cache.
  - If the domain name can't be found in the client side, then a request message will be sent to the DNS server on the Internet.
  - A response message will be received by client when a matching IP address is found from other DNS server.
  - Browser will use the received IP address to start a TCP connection with the matching server.
- Process order:
  - 1. **Browser cache**: when user try to access a domain name in browser, browser firstly will try to look for the browser cache to see if it already has the matching IP for that domain.
  - 2. **System cache**: If the browser can't find the matching IP, then the hosts file will be checked. If still not found, the local DNS will be checked as well, it is usually in the TCP/IP settings.
  - 3. **Router cache**: If there is no result matched in system and browser, router cache will be checked. All these three are called **Client DNS cache**.
  - 4. **ISP DNS cache**: There will be some cached IP on the ISP DNS server. It will the checked as well.
  - 5. **Root name server**: If no matched result above, then it will go into the root domain server to look for the result. There are only 13 root name servers worldwide. That is a tree structure for the root name servers and below top level name servers. If the root name server can't find the result, it will ask the top level name servers to look for the matching IP address.
  - 6. **Top Level name server**: A top level name server manages a lot of primary domain name servers. If it can't find the matching result, then it will ask the lower primary domain name servers to look for it.
  - 7. **Primary name server**: This will be a loop until a matching result is found in the primary name server or lower level name server and the results will be returned to the client.
  - 8. **Cache the result**: Result will be cached on the client side and client will use this IP to build the connection with the right server.

### Types of DNS

- **Root Name Server**: First stop for all domain name resolver requests,  based on the extension of the domain (.com/.net/.org/etc.), the root name server will redirect the requests to the matching TLD name server.
- **TLD (Top Level Domain) Server**: A TLD nameserver maintains information for all the domain names that share a common domain extension, such as .com, .net, or whatever comes after the last dot in a url. After receiving a response from a root nameserver, the recursive resolver would then send a query to a .com TLD nameserver, which would respond by pointing to the authoritative nameserver (primary name server) for that domain.
- **Authoritative nameserver (primary nameserver)**: When a recursive resolver receives a response from a TLD nameserver, that response will direct the resolver to an authoritative nameserver. The authoritative nameserver is usually the resolver’s last step in the journey for an IP address. It can provide a recursive resolver with the IP address of that server found in the DNS record.

![DNS Type][2]

- **Example for different domain name types**:

![Domain types][3]


### DNS Lookup Type

- **Recursive Query**: This normally happens between client and DNS resolver. It means DNS resolver will help client to find the matching IP address. If it can't be found in the next server, a resolver will keep requesting itself (recurisive) to the other available options until the matching IP address is found.
    - Recursive：仅返回应答，DNS Server可以不用支持Recursive查询模式。
- **Iterative Query**: This normally happens between the DNS resolver and the root server/TLD server and primary sever. It means if root server can't find a result, it will return the possible TLD server address to the DNS resolver, and then resolver will send a new request to that TLD server and ask for the matching IP. TLD server will do the same thing until the result is found in the authoritative name server.
    - Iterative：返回完整应答或者将请求引到其它DNS，所有DNS server都支持Iterative查询模式。

![DNS Loopup][4]


[1]: https://pic1.zhimg.com/80/e5143bc08d4ec9d7f210522c7e540f4d_hd.jpg
[2]: https://raw.githubusercontent.com/eziceice/blog/master/dns/DNS_Category.png
[3]: https://pic3.zhimg.com/80/v2-f5f5595b99be23d0b8d74e38b1e47b31_hd.jpg
[4]: https://pic4.zhimg.com/80/v2-1f77f41f7d43696f77a07874e0d86132_hd.jpg
