# Deleting a Data Entry in ASP.NET Core MVC

**Prerequisite**: Ensure the Delete Procedure is set up in your database.

### Delete Procedure

Below is the SQL stored procedure for deleting a Country from the database:

```sql
CREATE PROCEDURE [dbo].[PR_Country_DeleteByPk]
    @CountryID INT,
    @UserID INT
AS
BEGIN
    DELETE FROM [dbo].[Country]
    WHERE 
        CountryID = @CountryID AND UserID = @UserID;
END
```

## Step 1: Implement the `CountryDelete` Action Method in the Controller

Add the following code in your controller to handle the deletion of a Country. This method connects to the database, executes the delete procedure, and then redirects to the Country list page.

```csharp
public IActionResult CountryDelete(int countryID)
{
    try
    {
        string connectionString = this._configuration.GetConnectionString("ConnectionString");
        SqlConnection connection = new SqlConnection(connectionString);
        connection.Open();
        SqlCommand command = connection.CreateCommand();
        command.CommandType = System.Data.CommandType.StoredProcedure;
        command.CommandText = "PR_Country_DeleteByPK";
        command.Parameters.Add("@CountryID", SqlDbType.Int).Value = countryID;
        command.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
        command.ExecuteNonQuery();
    }
    catch (Exception ex)
    {
        TempData["ErrorMsg"] = ex.Message;
    }
    return RedirectToAction("CountryList");
}
```

## Step 2: Add a Delete Link on the Country List Page

In the list page, add a delete link/button that calls the `CountryDelete` action method. This link will pass the `CountryID` to the method to identify which Country to delete.

```html
<form method="post" asp-controller="Country" asp-action="CountryDelete">
  <input type="hidden" name="CountryID" value="@dataRow["CountryID"]" />
  <button type="submit" class="btn btn-outline-danger btn-xs">
    <i class="bi bi-x"></i>
  </button>
</form>
```

Another way to add a delete link is to use an anchor tag with a `href` attribute that points to the `CountryDelete` action method. This method will be called when the user clicks the link.

```html
<a href="/Country/CountryDelete?CountryID=@dataRow["CountryID"]" class="btn btn-outline-danger btn-xs">
  <i class="bi bi-x"></i>
</a>

```

using the `asp-route-` attribute:

```html
<a asp-controller="Country" asp-action="CountryDelete" asp-route-CountryID="@dataRow["CountryID"]" class="btn btn-outline-danger btn-xs">
  <i class="bi bi-x"></i>
</a>
```

## Step 3: Test the Delete Operation

Display Error Message in View Page
```csharp
<span class="text-danger">@TempData["ErrorMessage"]</span>
```
Ensure that the delete functionality works by testing it in the application. Check that the Country is removed from the database and that the list page updates accordingly.
