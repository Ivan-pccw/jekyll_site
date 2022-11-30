# FTP/SFTP

Notes for FTP/SFTP with spring camel.

## 1. Dependencies

```xml
<dependency>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-ftp</artifactId>
	<version>3.14.5</version>
</dependency>
```

## 2. FTP/SFTP servers

Free FTP/SFTP server for testing

FTP server: [https://dlptest.com/ftp-test/](https://dlptest.com/ftp-test/)

SFTP server: [https://www.sftp.net/public-online-sftp-servers](https://www.sftp.net/public-online-sftp-servers)

## 3. FTP component

Using FTP/SFTP by `ftp://` or `sftp://`  (same without `//`)

```java
//set up route like this
from("file:...")
	.to("ftp://[username]@[host]:[port]/[directory]");

//e.g.
	.to("ftp://dlpuser@ftp.dlptest.com/camel_ftp_test?password=rNrKYTX9g7z3RgJRmxWuGHbeu");
```

## 4. SFTP

Using FTP/SFTP by `ftp://` or `sftp://` (same without `//`)

```java
//set up route like this
from("file:...")
	.to("sftp://[username]@[host]:[port]/[directory]");

//e.g.
	.to("sftp://user@123.234.12.22:2222/share?password=somePassword");
```