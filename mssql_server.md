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

## DBeaver
Need to be installed
GUI 

```
$ apt-get install dbeaver
```
