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
