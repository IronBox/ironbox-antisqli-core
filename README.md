# Anti-SQL Injection (AntiSQLi) Library
The AntiSQLi libray provides developers with convenient methods and class extensions to reduce the risk from SQL injection (SQLi) attacks in their .NET applications.

The library provides support for:

- SQL Server
- Azure SQL Database
- Azure Cosmos DB/DocumentDB

## Installation

You don't need this source code unless you want to modify the library. If you just want to use the package, you can install it through the NuGet package gallery:

```powershell
Install-Package IronBox.AntiSQLi.Core
```

### Requirements

- .NET Standard 2.1+



## Usage


#### `System.Data.SqlClient.SqlCommand`


#### `Microsoft.Data.SqlClient.SqlCommand`



#### `Microsoft.Azure.Documents.SqlQuerySpec`


#### `Microsoft.Azure.Cosmos.QueryDefinition`

## How this library works
Whenever a dynamic SQL query is constructed using data from an untrusted source and then processed by a SQL interpreter, the potential for an attacker to execute unauthorized commands through the interpreter is created. Application code that contains this pattern is known to be vulnerable to [SQL injection (SQLi)](https://owasp.org/www-community/attacks/SQL_Injection) attacks.

### Automatic Query Parameterization
A common pattern for creating a dynamic SQL query is to express the query as a string with placeholders for variables. For example, the class method `String.Format` is a convenient way to implement this pattern.

```csharp
// Vulnerable SQLi application code example, username is untrusted data
SqlCommand cmd = new SqlCommand();
cmd.Connection = new SqlConnection("connection_string");
cmd.CommandText = String.Format("SELECT * FROM UserTable WHERE uname = '{0}'", username);
var dataReader = await cmd.ExecuteReaderAsync();
...
```
An approach to remediate the above vulnerable code is to use [SQL parameters](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.parameters). When using parameters, a placeholder is left in the literal query that references a parameter that the untrusted data is assigned to. The SQL interpreter handles the parameter data as non-executable code and any risk from SQLi attacks is mitigated.

```csharp
SqlCommand cmd = new SqlCommand();
cmd.Connection = new SqlConnection("connection_string");
cmd.CommandText = "SELECT * FROM UserTable WHERE uname = '@username'");

SqlParameter parameter = new SqlParameter("@username", SqlDbType.VarChar);
parameter.Direction = ParameterDirection.Input;
parameter.Size = username.Length;
parameter.Value = username;
cmd.Parameters.Add(parameter);

var dataReader = await cmd.ExecuteReaderAsync();
...
```
Using parameters is an effective way to mitigate risk from SQLi attacks; however, it can become tedious and error-prone especially in scenarios with large numbers of query variables. Certainly, the convenience of using a single line of code to create queries is lost.

The **AntiSQLi library provides class extensions that automatically parameterizes and loads dynamic queries**. Using the original code example above, here is how the AntiSQLi library could be used.

```csharp
using IronBox.AntiSQLi.Core.Sql

SqlCommand cmd = new SqlCommand();
cmd.Connection = new SqlConnection("connection_string");
cmd.LoadQuerySecure("SELECT * FROM UserTable WHERE uname = '{0}'", username);
var dataReader = await cmd.ExecuteReaderAsync();
...
```
At runtime, the class extension `.LoadQuerySecure(queryText, params Object[] args)` performs two important tasks. The first is it analyzes the `args` parameters provide and performs the equivalent to:
```csharp
SqlParameter parameter = new SqlParameter("@AntiSQLi1", SqlDbType.VarChar);
parameter.Value = username;
cmd.Parameters.Add(parameter);

// Any additional parameters would also be generated: @AntiSQLi2, @AntiSQLi3 ...
...
```
The second task performed by the AntiSQLi library is it replaces the original format items IDs in the query to match the IDs of the dynamically generated parameters. The original query:

````sql
SELECT * FROM UserTable WHERE uname = '{0}'
````
would be replaced at runtime with:
````sql
SELECT * FROM UserTable WHERE uname = '@AntiSQLiParam1'
````

## About
In 2012, Kevin Lam ([IronBox](https://www.ironbox.io)) and Joe Basirico ([Security Innovation](https://www.securityinnovation.com)) were thinking of ways to help .NET developers more easily defend their applications against SQL injection (SQLi) attacks, the #1 web application attack then. The [initial version of the AntiSQLi Library](https://github.com/IronBox/AntiSQLi) was developed and released in 2013.

In 2020, SQLi continued to be the #1 web application attack; however, the surface area for this attack had greatly expanded. In response, the AntiSQLi library was completely re-written with the goals of improving usability, integration and protection coverage.
