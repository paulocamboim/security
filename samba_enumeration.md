# Smb version
* SMB1 - Windows 2000, XP and Windows 2003. (Null Sessions)
* SMB2 - Windows Vista SP1 and Windows 2008
* SMB2.1 - Windows 7 and Windows 2008 R2
* SMB3 - Windows 8 and Windows 2012.

# Null Session Enumeration

## smbclient

Listing Shares
```
$ smbclient -L \\xx.xx.xx.xx
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Reports         Disk      
SMB1 disabled -- no workgroup available
```

Download all files from Share 
```
$ smbclient //10.10.10.xxx/Reports
smb> mask ""
smb> recurse ON
smb> prompt OFF
smb> cd 'path\to\remote\dir'
smb> lcd '~/path/to/download/to/'
smb> mget *
```

## smbMap
It shows shares permission right away.

```
$ smbmap -H 10.10.10.xxx -u 'WORKGROUP\test' -p ''
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.xxx...
[+] IP: 10.10.10.xxx:445        Name: 10.10.10.xxx                                      
        Disk                                                    Permissions
        ----                                                    -----------
        ADMIN$                                                  NO ACCESS
        C$                                                      NO ACCESS
        IPC$                                                    READ ONLY
        Reports                                                 READ ONLY
```

Recursively list all files in a Share
```
$ smbmap -H 10.10.10.xxx -u 'WORKGROUP\test' -p '' -R Reports

[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.xxx...
[+] IP: 10.10.10.xxx:445        Name: 10.10.10.xxx                                      
        Disk                                                    Permissions
        ----                                                    -----------
        Reports                                                 READ ONLY
        .\
        dr--r--r--                0 Mon Jan 28 23:26:31 2019    .
        dr--r--r--                0 Mon Jan 28 23:26:31 2019    ..
        -r--r--r--            12229 Mon Jan 28 23:26:31 2019    File.xlsm
```

Download files that match a Regex
```
$ smbmap -H 10.10.10.xxx -u 'WORKGROUP\test' -p '' -A "File.xlsm" -q  
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.xxx...
[+] IP: 10.10.10.xxx:445        Name: 10.10.10.xxx                                                             
        Disk                                                    Permissions                                    
        ----                                                    -----------                                    
        IPC$                                                    READ ONLY                                      
        [+] Match found! Downloading:                           Reports\.\Filexlsm                                    
        Reports                                                 READ ONLY  
```
The file will be downloaded to **/usr/share/smbmap/**




## enum4linux
This tool gives a lot of errros, prefer using smbclient to manually enumerate
```
$ enum4linux -a xx.xx.xx.xx
```
