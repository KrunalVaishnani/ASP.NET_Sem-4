
# Validation in ASP.NET Core MVC

Validation in ASP.NET Core MVC can be performed using the following methods:

- **Using ModelState Object (Server Side)**
- **Using Data Annotations (Server Side)**
- **Using jQuery (Client Side)**

**Prerequisite:** Ensure that the Add-Edit Page, Model, `lib` folder add necessary scripts and properly set up.

## Using jQuery (Client Side)

### Step 1: Include Required Scripts

To enable jQuery and Bootstrap functionalities, add the following scripts to your page:

```html
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
```

### Step 2: Import the Model into the Country Add-Edit Page

Ensure that the model is correctly imported into your Country Add-Edit page to allow for model binding and validation.

### Step 3: Apply Model Binding Using the `asp-for` Tag Helper

Use the `asp-for` tag helper in each text field or dropdown to seamlessly bind the model properties.

**Example:**

```html
<input type="text" class="form-control" asp-for="CountryName" placeholder="Enter Country Name"/>
```

### Step 4: Add `asp-validation-for` for Individual Field Validation

Attach the `asp-validation-for` tag to each text field or dropdown to display specific error messages for individual properties.

**Example:**

```html
<span asp-validation-for="CountryCode" class="text-danger"></span>
```

### Step 5: Render Partial Scripts for Validation

Include the following code in your Add-Edit page to render the partial scripts required for validation:

```csharp
@section Scripts {
    @{ await Html.RenderPartialAsync("_ValidationScriptsPartial"); }
}
```

### Step 6: Optional Script Rendering in `_Layout.cshtml`

To allow optional rendering of scripts in your views, add the following code to `_Layout.cshtml`:

```csharp
@await RenderSectionAsync("Scripts", required: false)
```

## Using Data Annotations (Server Side)

### Step 1: Add Data Annotations in the Model

Enhance your model by applying appropriate data annotations. Ensure that each annotation includes a relevant error message to guide users in case of invalid input.

**Example:**

```csharp
public class CountryModel
{
    public int CountryID { get; set; }

    [Required]
    [Display(Name = "Country Name")]
    public string CountryName { get; set; }

    [Required]
    [Display(Name = "Country code")]
    public string CountryCode { get; set; }

    [Required(ErrorMessage = "Country capital must not be null")]
    public string CountryCapital {  get; set; }

    [Required]
    [Display(Name = "User Name")]
    public int UserID { get; set; }

    [Required]
    [Display(Name = "Creation date")]
    public DateTime CreationDate { get; set; }
}
```

### Step 2: Create an Action Method in the Controller

Implement the following action method to handle the save operation in your controller:

```csharp
public IActionResult CountrySave(CountryModel countryModel)  
{  
    if (ModelState.IsValid)  
    {  
        return RedirectToAction("CountryList");  
    }  
    return View("CountryAddEdit", countryModel);  
}
```

## Using ModelState Object (Server Side)

### Step 1: Modify the `CountrySave` Method

To inspect ModelState errors before validation, modify the `CountrySave` method as follows:

```csharp
public IActionResult CountrySave(CountryModel countryModel)  
{
    foreach (var key in ModelState.Keys)  
    {  
        var state = ModelState[key];  
        foreach (var error in state.Errors)  
        {
            Console.WriteLine($"Key: {key}, Error: {error.ErrorMessage}");  
        }
    }  
    if (ModelState.IsValid)  
    {  
        return RedirectToAction("CountryList");  
    }  
    return View("CountryAddEdit", countryModel);  
}
```
### Step 2: Remove Default ModelState Errors

To replace the default error messages with custom ones, remove the existing ModelState entries:

```csharp
ModelState.Remove("CountryName");  
ModelState.Remove("CountryCode");  
ModelState.Remove("CountryCapital");  
ModelState.Remove("UserID");
```

### Step 3: Add Custom Error Messages

Add custom error messages to the ModelState object:

```csharp
public IActionResult ProductSave(ProductModel productModel)  
{  
    ModelState.Remove("CountryName");  
    ModelState.Remove("CountryCode");  
    ModelState.Remove("CountryCapital");  
    ModelState.Remove("UserID");  

    if (string.IsNullOrEmpty(countryModel.CountryName))  
    {
        ModelState.AddModelError("CountryName", "Country Name can't be null or empty.");  
    }  
    if (string.IsNullOrEmpty(countryModel.CountryCode))  
    {
        ModelState.AddModelError("CountryCode", "Country Code can't be null or empty.");  
    }  
    if (countryModel.UserID <= 0)  
    {
        ModelState.AddModelError("UserID", "UserID can't be zero or negative.");  
    }  

    if (ModelState.IsValid)  
    {
        return RedirectToAction("CountryList");  
    }   
    return View("CountryAddEdit", countryModel);  
}
```

With these steps, you will implement both client-side and server-side validation in ASP.NET Core MVC, ensuring that your application handles user input robustly and effectively.
