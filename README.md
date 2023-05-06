
# 🛑 Information Gathering Write-up

### 🌀 Topic : 
- Fingerprint Web Server
- Fingerprint Web Application Framework
-  Review Webpage Content
- Review Webserver Metafiles

***


## Fingerprint Web Server

### 🗯️ Description :

>  It helps us to find the specific vulnerabilities and exploits of that web server during testing
     Determine the version and type of a running web server to enable further discovery of any known vulnerabilities.


### 🛑 How to test :

> Techniques used for web server fingerprinting include banner grabbing

##### what is banner grabbing :
  ###### A banner grab is performed by sending an HTTP request to the web server and examining its response header

**🔸You can use different methods to get information, but in this method Netcat tool is used🔸**

🔑 For example, here is the response to a request from an Apache server :

```
$ nc 202.41.76.251 80 
HEAD / HTTP/1.0 HTTP/1.1 200 OK
Date: Mon, 16 Jun 2003 02:53:29 GMT 
Server: Apache/1.3.3 (Unix) (Red Hat/Linux) 
Last-Modified: Wed, 07 Oct 1998 11:18:14 GMT 
ETag: "1813-49b-361b4df6" 
Accept-Ranges: bytes 
Content-Length: 1179 
Connection: close 
Content-Type: text/html
```

The information in the Server section indicates that the desired server type and version is Apache 3.3.1.

🔑 Here is another response, this time from nginx:

```
HTTP/1.1 200 OK
Server: nginx/1.17.3
Date: Thu, 05 Sep 2019 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
```


In these examples, the server type and version is clearly exposed. But it is possible to limit or hide server information or even show other information

```
403 HTTP/1.1 Forbidden 
Date: Mon, 16 Jun 2003 02:41: 27 GMT 
Server: Unknown-Webserver/1.0 
Connection: close 
Content-Type: text/HTML; charset=iso-8859-1
```

In this case, the server information is hidden. Therefore, the security assessor cannot rely on this information.Identify the type and version of the server.

#### 1️ -  In cases where the server information is obscured, testers may guess the type of server based on the ordering of the header fields 

Note that in the Apache example above, the fields follow this order:
-   Date
-   Server
-   Last-Modified
-   ETag
-   Accept-Ranges
-   Content-Length
-   Connection
-   Content-Type

Note that in the Nginx example above, the fields follow this order
-   Server
-   Date
-   Content-Type

🔴 Now we can guess that the above server is of Injinx type, but it is possible that many servers share the same field order, so this method is not certain.

#### 2- Using Automated Scanning Tools

> 90% of this process is done using automatic scanning tools
> Automated tools can compare responses from web servers much faster than manual testing and use large databases of known responses to identify the server

-   httprint – [http://net-square.com/httprint.html](http://net-square.com/httprint.html)
-   httprecon – [http://www.computec.ch/projekte/httprecon/](http://www.computec.ch/projekte/httprecon/)
-   Netcraft – [http://www.netcraft.com](http://www.netcraft.com/)
-   Nmap – [https://nmap.org/](https://nmap.org/)
-   Netcat – [https://sectools.org/tool/netcat/](https://sectools.org/tool/netcat/)

*********

## Fingerprint Web Application Framework


### 🗯️ Description :

>  One of the most important steps in gathering information is identifying the web framework.
>  This importance is not only due to the vulnerability of previous versions, but also to incorrect configuration.

### 🛑 How to test :

> There are several common locations to consider in order to identify frameworks or components:

-   HTTP headers
-   Cookies
-   HTML source code
-   Specific files and folders
-   File extensions
-   Error messages

#### HTTP Headers

> The most basic form of identifying a web framework is to look at the `X-Powered-By` field in the HTTP response header, tool used `netcat`

```
$ nc 127.0.0.1 80
HEAD / HTTP/1.0

HTTP/1.1 200 OK
Server: nginx/1.0.14
[...]
X-Powered-By: Mono
```

**💢 This method does not always work and the answer can be easily hidden**

***

#### Cookies

> Another similar and somewhat more reliable way to determine the current web framework are framework-specific cookies.

![[Cakephp_cookie.png]]

_Figure 4.1.8-7: Cakephp HTTP Request_

The cookie `CAKEPHP` has automatically been set, which gives information about the framework being used. A list of common cookie names is presented in [Cookies](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/01-Information_Gathering/08-Fingerprint_Web_Application_Framework#Cookies) section. Limitations still exist in relying on this identification mechanism - it is possible to change the name of cookies. For example, for the selected `CakePHP` framework this could be done via the following configuration (excerpt from `core.php`):

```php
/**
* The name of CakePHP's session cookie.
*
* Note the guidelines for Session names states: "The session name references
* the session id in cookies and URLs. It should contain only alphanumeric
* characters."
* @link http://php.net/session_name
*/
Configure::write('Session.cookie', 'CAKEPHP');
```


***

#### HTML Source Code

> In this technique, we want to find certain patterns in the html source. For example, links to framework-specific CSS or JS folders. Finally, specific script variables may also point to a specific framework.

***


## Tools

A list of general and well-known tools is presented below. There are also a lot of other utilities, as well as framework-based fingerprinting tools.

### WhatWeb

Website: [https://github.com/urbanadventurer/WhatWeb](https://github.com/urbanadventurer/WhatWeb)

### Wappalyzer

Website: [https://www.wappalyzer.com/](https://www.wappalyzer.com/)

***

## Review Webpage Content


### 🗯️ Description :

>  Adding comments and metadata in the original source of a program is normal and even suggested by programmers. 
>  With However, comments and metadata located in the HTML source code can provide confidential information to an intruder. 
>  Comments and metadata should be carefully checked to ensure that no information is leaked.

**💢 The purpose of this test and evaluation is to check the comments and metadata of web pages**

HTML source code in order to identify comments that can give an intruder sensitive information about the application.

These comments can contain information such as SQL code, username, password, IP address
Internal and troubleshooting information.

```
<!-- Use the DB administrator password for testing: f@keP@a$$w0rD -->
```


***

## Review Webserver Metafiles


### 🗯️ Description :

> Metafiles They give us information about the configuration of the site

### 🛑 How to test :

> Any of the actions performed below with `wget` could also be done with `curl`. Many Dynamic Application Security Testing (DAST) tools such as ZAP and Burp Suite include checks or parsing for these resources as part of their spider/crawler functionality. They can also be identified using various [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) or leveraging advanced search features such as `inurl:`.


### Robot.txt

As an example, the beginning of the `robots.txt` file from [Google](https://www.google.com/robots.txt) sampled on 2020 May 5 is quoted below

```
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
```

The [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) directive refers to the specific web spider/robot/crawler. For example, the `User-Agent: Googlebot` refers to the spider from Google while `User-Agent: bingbot` refers to a crawler from Microsoft. `User-Agent: *` in the example above applies to all [web spiders/robots/crawlers](https://support.google.com/webmasters/answer/6062608?visit_id=637173940975499736-3548411022&rd=1).

The `Disallow` directive specifies which resources are prohibited by spiders/robots/crawlers.


### Sitemap 

> A site map is a file that provides information about the content presented on the website and the relationship between them

The following excerpt is from Google’s primary sitemap retrieved 2020 May 05.

```
$ wget --no-verbose https://www.google.com/sitemap.xml && head -n8 sitemap.xml
2020-05-05 12:23:30 URL:https://www.google.com/sitemap.xml [2049] -> "sitemap.xml" [1]

<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.google.com/schemas/sitemap/0.84">
  <sitemap>
    <loc>https://www.google.com/gmail/sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.google.com/forms/sitemaps.xml</loc>
  </sitemap>
```

Exploring from there a tester may wish to retrieve the gmail sitemap

```
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
```


### Security TXT

> It is a file that defines the security policy of that website
-   Identifying further paths or resources to include in discovery/analysis.
-   Open Source intelligence gathering.
-   Finding information on Bug Bounties, etc.
-   Social Engineering

```
$ wget --no-verbose https://www.linkedin.com/.well-known/security.txt && cat security.txt
2020-05-07 12:56:51 URL:https://www.linkedin.com/.well-known/security.txt [333/333] -> "security.txt" [1]
# Conforms to IETF `draft-foudil-securitytxt-07`
Contact: mailto:security@linkedin.com
Contact: https://www.linkedin.com/help/linkedin/answer/62924
Encryption: https://www.linkedin.com/help/linkedin/answer/79676
Canonical: https://www.linkedin.com/.well-known/security.txt
Policy: https://www.linkedin.com/help/linkedin/answer/62924
```



### Humans TXT

> Information about people who interact with the website. Build, manage, and...

```
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```


### Tools
-   Browser (View Source or Dev Tools functionality)
-   curl
-   wget
-   Burp Suite
-   ZAP
