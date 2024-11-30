# Implementing Register Functionality in ASP.NET Core

## Prerequisite

- **Stored Procedure: `PR_User_Insert`**

### Stored Procedure Code:

```sql
CREATE PROCEDURE [dbo].[PR_User_Insert]
    @Username VARCHAR(50),
    @Password VARCHAR(500),
    @DisplayName VARCHAR(100),
    @Email VARCHAR(100)
AS
BEGIN
    INSERT INTO [dbo].[User] 
		(
			[Username], 
			[Password], 
			[DisplayName], 
			[Email]
		)
    VALUES 
		(
			@Username,
			@Password, 
			@DisplayName,
			@Email
		);
END
```

## Step 1: Layout Setup in `_Layout_Register.cshtml`
Ensure that your layout file includes the necessary CSS and JS libraries to support the login page.

### Code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta content="width=device-width, initial-scale=1.0" name="viewport">
    <title>Pages / Login - NiceAdmin Bootstrap Template</title>
    <meta content="" name="description">
    <meta content="" name="keywords">

    <!-- Favicons -->
    <link href="~/img/favicon.png" rel="icon">
    <link href="~/img/apple-touch-icon.png" rel="apple-touch-icon">

    <!-- Google Fonts -->
    <link href="https://fonts.gstatic.com" rel="preconnect">
    <link href="https://fonts.googleapis.com/css?family=Open+Sans:300,300i,400,400i,600,600i,700,700i|Nunito:300,300i,400,400i,600,600i,700,700i|Poppins:300,300i,400,400i,500,500i,600,600i,700,700i" rel="stylesheet">
    <!-- Vendor CSS Files -->
    <link href="~/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="~/vendor/bootstrap-icons/bootstrap-icons.css" rel="stylesheet">
    <link href="~/vendor/boxicons/css/boxicons.min.css" rel="stylesheet">
    <link href="~/vendor/quill/quill.snow.css" rel="stylesheet">
    <link href="~/vendor/quill/quill.bubble.css" rel="stylesheet">
    <link href="~/vendor/remixicon/remixicon.css" rel="stylesheet">
    <link href="~/vendor/simple-datatables/style.css" rel="stylesheet">
    <!-- Template Main CSS File -->
    <link href="~/css/style.css" rel="stylesheet">
    <!-- =======================================================
    * Template Name: NiceAdmin
    * Updated: May 30 2023 with Bootstrap v5.3.0
    * Template URL: https://bootstrapmade.com/nice-admin-bootstrap-admin-html-template/
    * Author: BootstrapMade.com
    * License: https://bootstrapmade.com/license/
    ======================================================== -->
</head>
<body>
    @RenderBody()
    <a href="#" class="back-to-top d-flex align-items-center justify-content-center"><i class="bi bi-arrow-up-short"></i></a>

    <!-- Vendor JS Files -->
    <script src="~/vendor/apexcharts/apexcharts.min.js"></script>
    <script src="~/vendor/bootstrap/js/bootstrap.bundle.min.js"></script>
    <script src="~/vendor/chart.js/chart.umd.js"></script>
    <script src="~/vendor/echarts/echarts.min.js"></script>
    <script src="~/vendor/quill/quill.min.js"></script>
    <script src="~/vendor/simple-datatables/simple-datatables.js"></script>
    <script src="~/vendor/tinymce/tinymce.min.js"></script>
    <script src="~/vendor/php-email-form/validate.js"></script>
    <!-- Template Main JS File -->
    <script src="~/js/main.js"></script>
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

## Step 2: Create the `UserRegisterModel` in the UserModel

### Code:

```csharp
public class UserModel
{
    public int UserID { get; set; }

    [Required]
    [Display(Name = "User Name")]
    public string Username { get; set; }

    [Required(ErrorMessage = "Password is required.")]  
    public string Password { get; set; }

    [Required]
    [Display(Name = "Display name")]
    public string DisplayName { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
    public DateTime CreationDate { get; set; }
    public DateTime ModificationDate { get; set; }
}
```

## Step 3: Implement the Register Logic in `UserController.cs`

### Code:

```csharp
public IActionResult UserRegister(UserModel userRegisterModel)
{
    try
    {
        if (ModelState.IsValid)
        {
            string connectionString = this._configuration.GetConnectionString("ConnectionString");
            SqlConnection sqlConnection = new SqlConnection(connectionString);
            sqlConnection.Open();
            SqlCommand sqlCommand = sqlConnection.CreateCommand();
            sqlCommand.CommandType = System.Data.CommandType.StoredProcedure;
            sqlCommand.CommandText = "PR_User_Insert";
            sqlCommand.Parameters.Add("@Username", SqlDbType.VarChar).Value = userRegisterModel.Username;
            sqlCommand.Parameters.Add("@Password", SqlDbType.VarChar).Value = userRegisterModel.Password;
            sqlCommand.Parameters.Add("@DisplayName", SqlDbType.VarChar).Value = userRegisterModel.DisplayName;
            sqlCommand.Parameters.Add("@Email", SqlDbType.VarChar).Value = userRegisterModel.Email;
            sqlCommand.ExecuteNonQuery();
            return RedirectToAction("Login", "User");
        }
    }
    catch (Exception e)
    {
        TempData["ErrorMessage"] = e.Message;
        return RedirectToAction("Register");
    }
    return RedirectToAction("Register");
}
public IActionResult Register()
{
    return View();
}
```

## Step 3: Create the Register Page (`Register.cshtml`)
Set up your register page. Use `asp-for` to bind the model properties and `asp-validation-for` to display validation errors.

### Code:

```html
@{
    Layout = "~/Views/Shared/_Layout_Register.cshtml";
}

@model UserModel

<main>
    <div class="container">
        @if (TempData["ErrorMessage"] != null)
        {
            <div class="alert alert-danger" role="alert">
                @TempData["ErrorMessage"]
            </div>
        }
        <section class="section register min-vh-100 d-flex flex-column align-items-center justify-content-center py-4">
            <div class="container">
                <div class="row justify-content-center">
                    <div class="col-lg-4 col-md-6 d-flex flex-column align-items-center justify-content-center">

                        <div class="d-flex justify-content-center py-4">
                            <a class="logo d-flex align-items-center w-auto">
                                <img src="~/image/Logo.png" alt="">
                                <span class="d-none d-lg-block">Address Book</span>
                            </a>
                        </div><!-- End Logo -->

                        <div class="card mb-3">

                            <div class="card-body">

                                <div class="pt-4 pb-2">
                                    <h5 class="card-title text-center pb-0 fs-4">Register Your Account</h5>
                                    <p class="text-center small">Enter your details</p>
                                </div>

                                <form class="row g-3 needs-validation" asp-action="UserRegister" asp-controller="User">

                                    <div class="col-12">
                                        <label asp-for="Username" class="form-label">Username</label>
                                        <div class="input-group has-validation">
                                            <span class="input-group-text" id="inputGroupPrepend"></span>
                                            <input type="text" asp-for="Username" class="form-control" id="yourUsername">
                                            <span asp-validation-for="Username" class="text-danger"></span>
                                        </div>
                                    </div>

                                    <div class="col-12">
                                        <label asp-for="Password" class="form-label">Password</label>
                                        <input type="password" asp-for="Password" class="form-control" id="yourPassword">
                                        <span asp-validation-for="Password" class="text-danger"></span>
                                    </div>

                                    <div class="col-12">
                                        <label asp-for="DisplayName" class="form-label">Display name</label>
                                        <div class="input-group has-validation">
                                            <span class="input-group-text" id="inputGroupPrepend"></span>
                                            <input type="text" asp-for="DisplayName" class="form-control" id="yourUsername">
                                            <span asp-validation-for="DisplayName" class="text-danger"></span>
                                        </div>
                                    </div>

                                    <div class="col-12">
                                        <label asp-for="Email" class="form-label">Email</label>
                                        <input type="password" asp-for="Email" class="form-control" id="yourUsername">
                                        <span asp-validation-for="Email" class="text-danger"></span>
                                    </div>
                                    <div class="col-12">
                                        <button class="btn btn-primary w-100" type="submit">Register</button>
                                    </div>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </div>
</main><!-- End #main -->

@section Scripts {
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.3/jquery.validate.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validation-unobtrusive/3.2.12/jquery.validate.unobtrusive.min.js"></script>
}

<!-- OR -->

@section Scripts{
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
    }
}
```
