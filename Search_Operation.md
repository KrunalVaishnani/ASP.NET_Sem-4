# Search Operation in ASP.NET Core MVC

**Prerequisite**: Stored Procedure for Fetching Cities Based on Filters

## Step 1: Implement Search Logic in `CityFilter` Action Method

```csharp
public IActionResult CityFilter(IFormCollection fc)
{
    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command = connection.CreateCommand();
    command.CommandType = System.Data.CommandType.StoredProcedure;
    command.CommandText = "PR_City_SelectAll";
    command.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
    command.Parameters.Add("@CityName", SqlDbType.VarChar).Value = fc["CityName"].ToString();
    command.Parameters.Add("@CityCode", SqlDbType.VarChar).Value = fc["CityCode"].ToString();

    SqlDataReader reader = command.ExecuteReader();
    DataTable dataTable = new DataTable();
    dataTable.Load(reader);
    return View("CityList", dataTable);
}
```

## Step 2: Create Search Fields in View

```html
<div class="form-group col-md-2">
    <input type="text" id="searchBox" name="CityName" class="form-control" placeholder="City Name...">
</div>
<div class="form-group col-md-2">
    <input type="text" id="searchBox" name="CityCode" class="form-control" placeholder="City Code...">
</div>
<div class="form-group col-md-2">
    <input type="submit" asp-action="CityFilter" asp-controller="City" id="searchBox" class="btn btn-success form-control" value="Submit">
</div>
```

## Step 3: Create Stored Procedure for Searching Cities

```sql
CREATE PROCEDURE [dbo].[PR_City_SelectAll]
    @userID INT,
    @CityName VARCHAR(100) = NULL,
    @CityCode VARCHAR(100) = NULL
AS
BEGIN
    SELECT 
        [dbo].[City].[CityID],
        [dbo].[City].[CityName],
        [dbo].[City].[CityCode],
        [dbo].[City].[CreationDate],
        [dbo].[City].[StateID],
        [dbo].[State].[StateName],
        [dbo].[User].[UserID],
        [dbo].[User].[DisplayName],
        [dbo].[Country].[CountryName],
        [dbo].[State].[CountryID]
    FROM [dbo].[City]
    INNER JOIN [dbo].[State]
        ON [dbo].[City].[StateID] = [dbo].[State].[StateID]
    INNER JOIN [dbo].[User]
        ON [dbo].[City].[UserID] = [dbo].[User].[UserID]
    INNER JOIN [dbo].[Country]
        ON [dbo].[State].CountryID = [dbo].[Country].[CountryID]
    WHERE [dbo].[City].[UserID] = @userID 
        AND ([dbo].[City].[CityName] LIKE '%' + @CityName + '%' OR @CityName IS NULL)
        AND ([dbo].[City].[CityCode] LIKE '%' + @CityCode + '%' OR @CityCode IS NULL)
END
```

## Step 4: Verify Search Operation

After implementing the above logic, test the search functionality in your ASP.NET Core MVC application by entering different values in the search fields and checking if the correct records are displayed.

