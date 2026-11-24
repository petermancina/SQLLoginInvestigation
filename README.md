<h1>Investigate Suspicious Login Attempts with SQL queries</h1>

<h2>Project Description</h2>
SQL is a query language that we can use to query relational databases like MariaDB PostgreSQL, SQLite, Microsoft SQL Server, and Oracle Database among others. We can use SQL to obtain database information about our users, employees, devices, and various processes on our network. We might for instance use SQL to view data about login attempts to investigate possible brute force password attacks or to view where in the world users are attempting to login from. We can access SQL through the Linux command line and issue direct queries to a database in a manner which returns tailored datasets organized in exactly the way that we want to view the data so we can drill down and get answers. In this project, I will demonstrate how to use SQL to investigate failed login attempts, login attempts on specific dates, login attempts in specific geographic areas, and how to identify employees in various departments and buildings of a company. The dataset for this project is fictitious and comes from Chinook.

<h2>Scenario</h2>
I work at a large company based in Mexico and I have recently discovered a potential security incident that occurred after business hours on May 9, 2022 (imagine that today is May 10, 2022, so the suspicious incident happened yesterday). To investigate further, I am going to use my Linux command line to access our company database (MariaDB) and look at both the <b>log_in_attempts</b> table and the <b>employees</b> table.

<h2>Retrieve after hours failed login attempts </h2>
The first thing I will do is query the database to get after hours login activity for any day recorded in the database. Close of business is 6pm, and attackers might be more likely to attack when they think fewer people would notice, so I will issue the query for any login attempts after COB. 
</p>
To do that, I will issue the command:
</p>
<pre><b>SELECT * FROM log_in_attempts WHERE login_time > '18:00:00' AND success = 0;</b></pre>
</p>
<p></p>You can also issue the command this way and it will return the same data to you:</p>
<pre><b>SELECT *  
FROM log_in_attempts 
WHERE login_time > '18:00:00' AND success = 0;</b></pre>

Let's explain each of the parts of that query:
<pre><b>SELECT *</b></pre>
SELECT allows you to ask for specific columns of the table that you want to see the values for and can name each column that you want the returned table to include. You can also use the asterisk, a wildcard character, to return all columns
<pre><b>FROM</b></pre>           
FROM allows you to name the table you want the database to pull from, in this case, we want to see data in log_in_attempts.
<pre><b>WHERE</b></pre>        
WHERE lets you add conditions that filter your data and return a specific subset. In this case, we want the database to return anything in the login_time column that is greater than 18:00:00 which is 6pm on a 24-hour clock. Greater than 6pm, would be 6pm-11:59pm. Be sure to put the time in single quotes.
<pre><b>AND</b></pre>	          
AND is used when we want to see only records that match both conditions (in this case, attempts made after 6pm that failed.
<pre><b>success = 0</b></pre>   
success refers to the login attempt, whether it was a successful attempt (1 or "TRUE") or if it was a failed attempt (0 or "FALSE") 
<pre><b>;</b></pre> 	         
You end your SQL query with the semi-colon which tells the database that this is the end of the query. If you forget this, the database will return nothing.
</p>
Below we see that my query yielded 19 failed login attempts after hours from May 9-12 in Mexico, Canada, and the US.
<p align="center">
<br/>
<img src="https://i.imgur.com/PiQmF8b.png" height="100%" width="100%" alt="19 Failed login attempts"/>
</p>

<h2>Retrieve login attempts on specific dates</h2>
Since the suspicious incident we have been alerted to was from yesterday, May 9th, lets hone in on yesterday's login attempts and the day before to see if there is anything in those logs that might give me more information. To return those two dates I will use the <b>OR</b> operator rather than AND because we would like to see records that are either from May 8th or 9th, that is, we want to see both types of records (individual records do not need to match both conditions May 8th and May 9th).
<pre><b>SELECT * 
FROM log_in_attempts 
WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';</b></pre>
<p align="center">
<br/>
<img src="https://i.imgur.com/9MMwxZL.png" height="100%" width="100%" alt="Login attempts on specific dates image"/>
</p>
<h2>Retrieve login attempts outside of Mexico</h2>
Our team has ruled out that any Mexico-based login attempts are true positives (real intrusion attempts), so we need to remove those records from our results and look at all non-Mexico login attempts. To do this, we will use the <b>NOT</b> operator, which tells the database to exclude a specified value:
<pre><b>SELECT *
FROM log_in_attempts
WHERE NOT country LIKE 'MEX%';</b></pre>
In this query, we are telling the database to not include records where the value in the country column begins with MEX. We do this because sometimes people entering data in a database might use different conventions to refer to the same thing like a country name. In this case, the column country in the <b>log_in_attempts</b> table contains values "MEX" and "MEXICO" that both refer to the country Mexico. To return all records for login attempts from Mexico, we use the % wildcard which tells the database to include any values starting with MEX and continuing with any character and that is any number of characters long. When we use the % wildcard, we cannot use the equals sign = as the operator, but rather the operator <b>LIKE</b>.
<p align="center">
<br/>
<img src="https://i.imgur.com/NZUOtGW.png" height="100%" width="100%" alt="Login attempts outside of Mexico image"/>
</p>
The query yielded records only for the United States and Canada, with all Mexico entries removed.

<h2>Retrieve employees in Marketing</h2>
Since a major motivation of many attackers is financial, we can use SQL to identify accounts in Marketing, Sales, and Finance that may be targeted. For this query, I will request data from the <b>employees</b> table. To get a quick look at what this table contains before I make a more narrow query, I will issue a query to return all columns and all records:
<pre><b>SELECT *
FROM employees;</b></pre>
<p align="center">
<br/>
<img src="https://i.imgur.com/srSr1Y0.png" height="100%" width="100%" alt="All Employees image"/>
</p>
Now that I know what is in the table - employee_id, device_id, username, department, and office - I will narrow my query so that the database only returns employee accounts in the Marketing department. I also can further narrow the query so that it returns only accounts in a particular building. In this case, my team suspects suspicious activity related to a Marketing account in the East building, so I will narrow my query to the following: 
<pre><b>SELECT *
FROM employees
WHERE department = "Marketing' AND office LIKE 'East%';</b></pre>
<p align="center">
<br/>
<img src="https://i.imgur.com/FpMc2ga.png" height="100%" width="100%" alt="Employees in Marketing image"/>
</p>

<h2>Retrieve employees in Finance or Sales</h2>
Now I will investigate employee user accounts in Finance and Sales. To return user accounts from either Finance or Sales in one table, I will use the <b>OR</b> operator again:
<pre><b>SELECT *
FROM employees
WHERE department = 'Finance' OR department = 'Sales';</b></pre>
<p align="center">
<br/>
<img src="https://i.imgur.com/YOyLl5R.png" height="100%" width="100%" alt="Employees in Finance or Sales image"/>
</p>

<h2>Retrieve all employees not in IT</h2>
And lastly, I want to look at a list of all company user accounts for employees that are not in the IT department. To do that, I will use the NOT operator. 
<pre><b>SELECT *
FROM employees
WHERE NOT department = 'Information Technology';</b></pre>
<p align="center">
<br/>
<img src="https://i.imgur.com/HWvlbkN.png" height="100%" width="100%" alt="All employees not in IT image"/>
</p>

<h2>Summary</h2>
In this project I demonstrated how you can issue SQL queries inside of a Linux terminal to return information that you can use to investigate alerts about suspicious login attempts and user accounts. SQL is an incredibly powerful tool in the cybersecurity toolbox and allows you to filter data and drill down to view exactly the information that you need and nothing more. This shortens the time it takes you to find the answers to your questions, enforce your security posture, and protect critical business processes from unauthorized intrusions. 


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>

