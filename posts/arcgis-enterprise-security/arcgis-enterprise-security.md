# Security best practices for ArcGIS Enterprise

The following post contains excerpts from _Mastering ArcGIS Enterprise Administration_ by Chad Cooper, published by Packt Publishing in October 2017 and available at [packtpub.com](https://www.packtpub.com/application-development/mastering-arcgis-enterprise-administration) and [Amazon](http://a.co/gabnhYi).

[Bon Qui Qui knows the importance of security](https://www.youtube.com/watch?v=jZkdcYlOn5M), and you do too. However, unlike Bon Qui Qui, most of us tend to, well, sort of forget about security. Let's be honest here; you know if your IT department didn't make you change your domain login every 6 months, you'd _never_ change it. I just had to change mine last week and I'm _still_ typing the wrong one at least once a day (nothing short of a miracle that I haven't locked myself out yet). All joking aside, security is a serious matter that can make you or break you - and break you it will if you don't respect it. 

As consultants here at GISinc, we see _a lot_ of ArcGIS Enterprise environments, are responsible for _a lot_ of those environments, and of course are looked to for guidance on all matters related to server environments, security included. That said, security is always on our minds, so let's discuss a few things you can do (most of which are fairly easy) to up your security game.

## Passwords

Considered by most to be a necessary evil, passwords are an essential first line of defense to your system and are the most widely used form of authentication throughout the world. We all know our passwords should be “strong”, but what does that really mean? Conventional wisdom has always said that passwords must have some form of complexity, usually in the form of one number, one upper case letter, one lower case letter, and one symbol (sound familiar?). Regarding length, the National Institute of Standards and Technology (https://www.nist.gov) recommends a minimum length of 8 characters. The key to getting the most from password length without the burden of complexity requirements is to create a memorable password, as is so brilliantly illustrated below in the classic XKCD comic.

![image](https://imgs.xkcd.com/comics/password_strength.png)

Using a password generator such as [XKPasswd](https://xkpasswd.net/s/) makes creating memorable passwords simple.

So, the next time you are creating an account, be sure to create a stong, memorable password.

## Changing up admin accounts

Historically, _siteadmin_ and _portaladmin_ have for years been the classic go-to administrator account user names for ArcGIS Server and Portal, respectively. Do your admin accounts _have_ to use those user names? Of course not. Feel free to mix things up and create more complex user names for your administrative accounts.

## Ports

Ports allow for inbound and/or outbound traffic between your servers and in from and out to the internet. That said, open ports offer entry points into your system. Is it easier to have all ports open inbound on all your servers? Sure it is, it makes setting up your system easier, but it is not secure. Make sure only the necessary ports are open, and if possible, only allow communication on those ports between the required servers. If your web server is out in a DMZ, ensure that 6080/6443 is only open on the DMZ firewall to allow communication between the web server and your ArcGIS Server machine. Likewise for Portal, 7080/7443 should only be opened on the DMZ firewall to allow traffic from your web server to your Portal server.

If you allow FTP or RDP access to any of your servers, set up the security rules to only allow inbound access from certain IPs or an appropriate CIDR range, if possible. This closes off access to the wide open internet locks access down. 

## HTTPS and SSL

As an IT professional, you've at least heard of HTTPS and SSL, but do you need _really_ need these for your ArcGIS Enterprise environment. The simple answer is yes, you do. Let's briefly examine why.

### Encryption

SSL encrpyts information such as usernames, passwords, and credit card information as it is sent across networks and the internet. Without SSL, these bits of information are sent as _plain text_, meaning that anyone that intercepts your traffic can see the information, which is no good. 

### Authentication

SSL also provides authentication, meaning that you can be sure that the server you _think_ you are communicating with is indeed that server. Acquiring a SSL certificate requires a request from the server to be submitted to a certificate registrar, and the requester must provide information to prove they are who they say they are.

### Trust

Seeing that little green lock to the left of the web address gives people (myself included) a nice warm fuzzy feeling and builds trust, making your users know they are where they intended to be and that the site is indeed secure. 

### Required for Portal for ArcGIS

Portal for ArcGIS requires a CA-signed or domain-level SSL certificate for production environments. 

## Scanning for best practices



## Conclusion

Keeping up with security is a necessary evil that, let's face it, no one really wants to deal with.



