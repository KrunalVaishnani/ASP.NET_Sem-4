# Establishing SQL Connection & Displaying Data in ASP.NET Core MVC

**Prerequisite**: Ensure SQL Server is installed and the `SelectAll` stored procedure is created.

```sql
CREATE PROCEDURE [dbo].[PR_Country_SelectAll]
AS
BEGIN
    SELECT 
        [dbo].[Country].[CountryID],
        [dbo].[Country].[CountryName],
        [dbo].[Country].[CountryCode],
        [dbo].[Country].[CountryCapital],
        [dbo].[User].[DisplayName],
		[dbo].[User].[UserID],
		[dbo].[Country].[CreationDate]
    FROM
        [dbo].[Country]
    INNER JOIN
        [dbo].[User] ON [dbo].[Country].[UserID] = [dbo].[User].[UserID]
END
```

## Step 1: Add Static Data to SQL Server

Insert static data into your SQL Server database using the provided script [here](https://codeshare.io/KWnnDK).

## Step 2: Configure the Connection String in `appsettings.json`

### For Windows Users:

Add the connection string for your SQL Server database in the `appsettings.json` file as shown below:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;Integrated Security=true;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=LAPTOP-LBMAFD6U\\SQLEXPRESS;Initial Catalog=StudentMaster;Integrated Security=true;"
}
```

### For Mac Users:

For Mac users, the connection string should include the user ID and password:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;User id=userID; password=Password;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=localhost;Initial Catalog=Practice;User id=SA; password=MyStrongPass123;"
}
```

## Step 3: Set Up the Configuration Variable in the Controller

In your controller, declare a configuration variable and initialize it using the constructor. This will allow you to access the connection string from the configuration.

```csharp
private IConfiguration configuration;

public CountryController(IConfiguration _configuration)
{
    configuration = _configuration;
}
```

## Step 4: Install the `System.Data.SqlClient` Package

Install the `System.Data.SqlClient` package from the NuGet Package Manager to enable SQL Server connectivity.

- Navigate to **Tools** -> **NuGet Package Manager** -> **Manage NuGet Packages for Solution**.
- Ensure the project is not running during the installation.

## Step 5: Write Logic to Fetch Data in the List Page Action Method

In the action method for your list page, add the following logic to fetch data from the SQL Server database:

```csharp
public IActionResult CountryList()
{
    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command = connection.CreateCommand();
    command.CommandType = System.Data.CommandType.StoredProcedure;
    command.CommandText = "PR_Country_SelectAll";
    command.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
    SqlDataReader reader = command.ExecuteReader();
    DataTable dataTable = new DataTable();
    dataTable.Load(reader);
    return View(dataTable);
}
```

## Step 6: Display the Data in the View

In your view page, import the `DataTable` and use a `foreach` loop to iterate through the data and display it:

```csharp
@model DataTable
@using System.Data

<table class="table">
    <thead>
        <tr>
            <th>CountryName</th>
            <th>CountryCode</th>
            <th>CountryCapital</th>
        </tr>
    </thead>
    <tbody>
        @foreach (DataRow row in Model.Rows)
        {
            <tr>
                <td>@row["CountryName"]</td>
                <td>@row["CountryCode"]</td>
                <td>@row["CountryCapital"]</td>
            </tr>
        }
    </tbody>
</table>
```
