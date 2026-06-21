## Sql Injection
- It happens when a web application takes text typed by a user (like a username, a search term, or an ID) and **directly glues it into a sql query without checking it first**.
- A web based attack where we **manipulate the sql query** in order to always return "True", without even knowing the username or password
- Uses the concept of the **OR logic gate**
- Inject a logical statement that is always true, usually OR '1'='1'
- The query evaluates the condition for every single row in the table. Because 1=1 is always true, the entire query evaluates to true for every single record. As a result, the database unlocks itself and dumps the entire contents of the table right back to the hacker's screen.


### Demo (Tautology-based SQL Injection attack)
```python
#SQL Injection attack 

import pandas as pd
import mysql.connector

dbh = mysql.connector.connect(
    host="localhost",
    user="dbuser1001",
    password="strong_password",
    database="college"
)

user=input("Enter username:")
password=input("Enter password:")                                                              #abc' or '1'='1 
sql="SELECT * from injection_attack where user='{}' and password='{}' ".format(user,password)  #Table name is injection_attack
print(sql)

cursor = dbh.cursor()
cursor.execute(sql)

for row in cursor.fetchall():
    print(row)

```


### ScreenShot
![img_2.png](screenshots%2Fimg_2.png)

### The Code 
- #### Part 1: Importing the Required Tools
   - ```python
        import pandas as pd
        import mysql.connector
     ```
   - Imports the Pandas data manipulation library
   - Imports the Python driver that allows it to speak native SQL protocol directly to the running MySQL database server.

- #### Part 2: Establishing the Database Connection Pipeline
  - ```python
    dbh = mysql.connector.connect(
    host="localhost",
    user="dbuser1001",
    password="strong_password",
    database="college")
    ``` 
  - Establishes connection with the MySql database

- #### Part 3: Capturing User Input (The Trapdoor)
  - ```python
    user=input("Enter username:")
    password=input("Enter password:")   #'abc' or '1'='1'
    ``` 
    
  - **password=input(...)**: Asks for the password input. Here is where the attack payload enters the system. Typing abc' or '1'='1, Python treats this entirely as plain text data -- it doesn't realize it contains malicious syntax yet.

- #### Part 4: String Assembly (Where the Vulnerability Happens)
  - ```python
    sql="SELECT * from injection_attack where user='{}' and password='{}' ".format(user, password)
    ``` 
  - This uses Python's .format() string function to mechanically drop raw inputs from the user into the {} placeholder brackets.
  - This is a pure string-manipulation trick. It doesn't check what the user typed. It blindly drops the malicious password text right into the middle of the SQL command construction. ch