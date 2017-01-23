- Copy/paste
- FTP
    - Old skool
    - Requires ports to be opened on server
    - Not secure
    - Scriptable though with Python
    - MB-scale tasks
    - Can restict to IP or range of IPs
- S3
    - Part of AWS ecosystem
    - Highly secure
    - Highly configurable
    - Easily scriptable with Python
        - mulitthreaded, so faster
    - Does incur AWS costs, but they are very low for modest transfers
    - MB- to GB-scale - log shipping, database backups, weekly GIS data updates
- Import/Export
    - Part of AWS ecosystem
    - Cost- and time-effective
    - You buy external HD, put your data on it, ship to AWS
        - AWS puts data in a snapshot or S3 bucket
    - Can be secure
    - TB-scale
    - Imagery libraries

Other services Amazon has available (haven't used these):

- Snowball
    - TB- to PB-scale
    - Secure AWS device they ship to you, you load your data, ship back to them
        - encrypt your data
    - Chain of custody?
- Snowmobile
    - PB- to EB-scale
    - An actual 18 wheeler, 100 PB 45-foot long container
    - Fastest way to move massive amounts of data across the country
    - Imports data to S3 or Glacier
    - Very new to AWS
    
    
So, you have yourself a new Windows EC2 instance on AWS. Chances are, you 
need to get some data up there, maybe a library of images or PDFs you 
plan to serve up or maybe a 50GB database backup that you need to 
restore. Maybe you need to get data up there routinely as some sort 
of an update process. Whatever the case, there are plenty of ways 
to get your data up to your EC2 instance. Here at GISinc, we deploy 
ArcGIS Server on AWS EC2 instances routinely and always have a need 
to get some sort of data up to the server. Sometimes these are 
one-shot uploads (a database backup for restore), but most of the 
time, as part of a automated update process to get production 
data from the client environment up the AWS server for publication. 
The following is a breakdown of some of the ways we have pushed data 
up to AWS, and following that are some other methods Amazon has 
available for extremely large data transfers.

Common methodologies
--------------------

Copy/paste
==========

When you connect to your EC2 instance through RDP, there's always 
copy/paste for simple one-off data tranfers.

FTP
===

FTP can be setup relatively easily on a Windows Server instance. 
Although a little old school, FTP is still a solid method for 
transferring MB-scale data across networks. FTP can also easily scripted 
with Python and the `ftplib` module from the standard library. 
In it's simpliest form, that would look like the following:

```
link = ftplib.FTP(ftp_server,
                  ftp_user,
                  ftp_pass)
link.set_pasv(True)
with open(input_file, "rb") as z:
    link.storbinary('STOR {0}'.format(input_file), z)
link.close()
```

Although not secure, with AWS, you can restrict port access to 
only a certain IP address or a range of IPs, meaning that you 
can restrict port 25 to only be available over certain ports, 
thus providing some level of security.

S3
==


Import/Export
=============


Other methods
-------------


Snowball
========


Snowmobile
==========


