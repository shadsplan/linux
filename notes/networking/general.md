# General Networking Notes

## [How a DNS Server (Domain Name System) works.](https://www.youtube.com/watch?v=mpQZVYPuDGU)

DNS resolves domain names to IP addresses.

1. User types a URL (e.g., www.yahoo.com) into their web browser. If the web browser / OS has the IP address cached, it will use that.
2. If not, it sends a query to the DNS resolver server. This is basically your ISP.
3. The Resolver server will check if it has the IP address cached. If so, it returns it to the user.
4. If not, it sends a query to a root DNS server.
    1. The root server is the top or the root, of the DNS hierarchy.
    2. There are 13 root DNS servers worldwide. They are operated by 12 different organizations. Each set has their own unique IP address.
    3. The root server won't know the IP address of www.yahoo.com, but it does know where to send the Resolver to help it find the IP address.
    4. The root server will direct the Resolver to the TLD (top-level domain) DNS server for .com domains.
5. The Resolver will now ask the TLD server for the IP address of www.yahoo.com.
    1. The TLD server stores the address information for top-level domains (ex. com, .net, .org, etc.). This particular TLD server manages the .com domain, which yahoo.com is a part of.
    2. When the TLD server receives the query for www.yahoo.com, it won't know the IP address of www.yahoo.com, so it will direct the Resolver to the next and final level, the Authoritative Name server.
6. The Resolver will now ask the Authoritative Name server for the IP address of www.yahoo.com.
    1. The Authoritative Name server is responsible for knowing everything about the domain, including its IP address. They are the final authority.
    2. When the Authoritative Name server receives the query for www.yahoo.com, it will respond with the IP address of www.yahoo.com.
7. The Resolver will now return the IP address of www.yahoo.com to the user's web browser.
    1. Once the resolver receives the IP address, it will store it in its cache memory, in case another user requests the same domain name.
8. Your computer can now retrieve the web page from the web server using the IP address.

## [What happens when you type a URL into your browser?](https://www.youtube.com/watch?v=AlkDbnbv7dk)

1. Bob enters a URL into the browser.
2. Browser looks up IP in cache. If not in browser cache, checks OS cache.
    1. If not in OS cache, goes to DNS resolver. See above.
3.  Browser establishes a TCP connection to the web server using a 3-way handshake. Modern browsers use a keep-alive connection, so they can reuse the same TCP connection for multiple requests.
    1. If the website uses HTTPS, the browser and web server will perform a TLS handshake to establish an encrypted connection.
4. Browser sends an HTTP request to the web server.
5. Server sends back an HTTP response.
6. Browser renders the HTTP content.