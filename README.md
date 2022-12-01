# dotnet-odbc-connector-on-mac

How to get odbc up and running on MacOS with a dotnet project in visual studio mac.

This document describes the process of installing the ODBC Connector for MySQL / MariaDB on macbook pro m1 running MacOS 13 Ventura.

## The problem

I was trying to run our companys dotnet 6 project on a mac.

The required driver was specified in the Connection string in `appsettings.json`

My appsettings.json was trying to find the MariaDB ODBC driver like so: 

```
{
  "ConnectionStrings": {
    "ServerDatabaseConnectionString": "DRIVER={MariaDB ODBC 3.1}; (...)
  }
(...)
}
```

But when running any connections to the DB I got the following error:

```
System.DllNotFoundException: Dependency unixODBC with minimum version 2.3.1 is required.
Unable to load shared library 'libodbc.2.dylib' or one of its dependencies. In order to help diagnose loading problems, consider setting the DYLD_PRINT_LIBRARIES environment variable: dlopen(liblibodbc.2.dylib, 0x0001): tried: 'liblibodbc.2.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OSliblibodbc.2.dylib' (no such file), '/usr/lib/liblibodbc.2.dylib' (no such file, not in dyld cache), 'liblibodbc.2.dylib' (no such file), '/usr/local/lib/liblibodbc.2.dylib' (no such file), '/usr/lib/liblibodbc.2.dylib' (no such file, not in dyld cache
```

## How to fix it


### 1. Install unixudbc and maria db connector

```bash
 
  $ brew install unixudbc
  
  $ brew install mariadb-connector-odbc
 
```

### 2. make the lib folder visible to visual studio via a symlink

```bash
$ sudo ln -s /opt/homebrew/lib /usr/local/lib
```


> If you rebuild and run at this stage you'll get the following error:
> `System.Data.Odbc.OdbcException (0x80131937): ERROR [01000] [unixODBC][Driver Manager]Can't open lib 'MariaDB ODBC 3.1' : file not found`
> This is because it does not know about our the mapping between the name 'MariaDB ODBC 3.1' and the driver..



### 3. Map the driver name to the driver location in odbcinst.ini

Open the config file: 

`$ vim /opt/homebrew/etc/odbcinst.ini`

Paste following into the file 

```

[MariaDB ODBC 3.1]
Description=Maria DB ODBC 3.1 in etc odbcinst.ini
Driver=/opt/homebrew/Cellar/mariadb-connector-odbc/REPLACE_ME_WITH_VERSION_NUMBER/lib/mariadb/libmaodbc.dylib
UsageCount=1

```  

**Replace version number** 

You can find the version number by running this command in another termanil

```
$ ls /opt/homebrew/Cellar/mariadb-connector-odbc
3.1.17
```

Save and quit

`:wq`

### Rebuild entire solution and all should be good :) 
