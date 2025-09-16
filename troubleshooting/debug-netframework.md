# Debugging .NET Framework Projects

## Visual Studio Debugging Setup

Debugging .NET Framework projects in Method's ecosystem, particularly legacy components and IIS-hosted applications.

## Prerequisites

- Visual Studio 2019/2022 with .NET Framework debugging tools
- IIS Express or full IIS configured
- Method development environment fully configured
- Legacy Method applications and components

## Debugging Configuration

### Web.config Debug Settings
```xml
<system.web>
  <compilation debug="true" targetFramework="4.8" />
  <httpRuntime targetFramework="4.8" />
  <customErrors mode="Off" />
</system.web>

<system.webServer>
  <modules runAllManagedModulesForAllRequests="true" />
</system.webServer>
```

### Application Pool Configuration
- **Framework Version**: .NET Framework v4.0
- **Managed Pipeline Mode**: Integrated
- **Identity**: ApplicationPoolIdentity or NetworkService
- **Enable 32-bit Applications**: False (unless specifically required)

## Debugging Method Legacy Components

### Action Editor (AngularJS + .NET Framework)
- **Project**: Method.ActionEditor
- **Technology**: AngularJS frontend with .NET Framework API
- **Debugging**: Hybrid debugging (JavaScript + C#)
- **Configuration**: Both IIS Express and full IIS

### Classic ASP.NET WebForms
- **Projects**: Various legacy Method applications
- **Technology**: WebForms, User Controls, Master Pages
- **Debugging**: Page lifecycle, ViewState, PostBack events
- **Configuration**: web.config transforms for different environments

### WCF Services
- **Projects**: Method WCF service endpoints
- **Technology**: SOAP services, WCF bindings
- **Debugging**: Service contracts, data contracts, bindings
- **Configuration**: Service model configuration in web.config

## Debugging Techniques

### Setting Breakpoints in WebForms
```csharp
// Page lifecycle debugging
protected void Page_Load(object sender, EventArgs e)
{
    System.Diagnostics.Debugger.Break(); // Forces debugger attachment
    
    if (!IsPostBack)
    {
        // Initial page load logic
        LoadInitialData();
    }
}

// Control event debugging
protected void Button1_Click(object sender, EventArgs e)
{
    // Set breakpoint here to debug button click events
    ProcessButtonClick();
}
```

### Debugging User Controls
```csharp
public partial class MyUserControl : System.Web.UI.UserControl
{
    protected void Page_Load(object sender, EventArgs e)
    {
        // Debug user control lifecycle
        System.Diagnostics.Debug.WriteLine($"UserControl loaded: {this.ID}");
    }
    
    // Property debugging
    public string MyProperty
    {
        get
        {
            System.Diagnostics.Debug.WriteLine("Property accessed");
            return ViewState["MyProperty"] as string;
        }
        set
        {
            System.Diagnostics.Debug.WriteLine($"Property set: {value}");
            ViewState["MyProperty"] = value;
        }
    }
}
```

### Debugging ViewState Issues
```csharp
// ViewState inspection
protected override void OnPreRender(EventArgs e)
{
    base.OnPreRender(e);
    
    // Output ViewState size
    var viewStateSize = ViewState.ToString().Length;
    System.Diagnostics.Debug.WriteLine($"ViewState size: {viewStateSize} bytes");
}

// ViewState debugging in Page_Load
protected void Page_Load(object sender, EventArgs e)
{
    if (IsPostBack)
    {
        // Inspect ViewState contents
        foreach (DictionaryEntry item in ViewState)
        {
            System.Diagnostics.Debug.WriteLine($"ViewState: {item.Key} = {item.Value}");
        }
    }
}
```

## IIS Debugging

### Attaching to IIS Worker Process
1. **Debug → Attach to Process**
2. **Show processes from all users** ✓
3. **Find w3wp.exe** (IIS worker process)
4. **Select appropriate worker process** (check App Pool name)
5. **Attach**

### IIS Express Debugging
1. **Set project as StartUp Project**
2. **Press F5** or **Debug → Start Debugging**
3. **IIS Express automatically launches with debugging**

### Full IIS Debugging
```xml
<!-- web.config for full IIS debugging -->
<system.web>
  <compilation debug="true" />
  <httpRuntime executionTimeout="300" maxRequestLength="51200" />
</system.web>

<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

## Common Debugging Scenarios

### Session State Issues
```csharp
// Session debugging
protected void Page_Load(object sender, EventArgs e)
{
    // Check session availability
    if (Session != null)
    {
        System.Diagnostics.Debug.WriteLine($"Session ID: {Session.SessionID}");
        System.Diagnostics.Debug.WriteLine($"Session count: {Session.Count}");
        
        // List all session variables
        foreach (string key in Session.Keys)
        {
            System.Diagnostics.Debug.WriteLine($"Session[{key}] = {Session[key]}");
        }
    }
}
```

### Authentication and Authorization
```csharp
// User authentication debugging
protected void Page_Load(object sender, EventArgs e)
{
    if (User.Identity.IsAuthenticated)
    {
        System.Diagnostics.Debug.WriteLine($"User: {User.Identity.Name}");
        System.Diagnostics.Debug.WriteLine($"Auth Type: {User.Identity.AuthenticationType}");
        
        // Check roles
        if (User.IsInRole("Admin"))
        {
            System.Diagnostics.Debug.WriteLine("User is admin");
        }
    }
    else
    {
        System.Diagnostics.Debug.WriteLine("User not authenticated");
    }
}
```

### Database Connection Issues
```csharp
// ADO.NET debugging
protected void TestDatabaseConnection()
{
    string connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
    
    try
    {
        using (var connection = new SqlConnection(connectionString))
        {
            connection.Open();
            System.Diagnostics.Debug.WriteLine("Database connection successful");
            
            // Test query
            using (var command = new SqlCommand("SELECT 1", connection))
            {
                var result = command.ExecuteScalar();
                System.Diagnostics.Debug.WriteLine($"Query result: {result}");
            }
        }
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"Database error: {ex.Message}");
        // Log to application log
        Server.MapPath("~/App_Data/error.log");
    }
}
```

## Performance Debugging

### Page Performance Analysis
```csharp
public partial class MyPage : System.Web.UI.Page
{
    private DateTime _pageStartTime;
    
    protected void Page_Init(object sender, EventArgs e)
    {
        _pageStartTime = DateTime.Now;
    }
    
    protected void Page_PreRender(object sender, EventArgs e)
    {
        var elapsed = DateTime.Now - _pageStartTime;
        System.Diagnostics.Debug.WriteLine($"Page render time: {elapsed.TotalMilliseconds}ms");
    }
}
```

### Memory Usage Debugging
```csharp
// Memory usage monitoring
protected void Page_Load(object sender, EventArgs e)
{
    long memoryBefore = GC.GetTotalMemory(false);
    
    // Your page logic here
    LoadPageData();
    
    long memoryAfter = GC.GetTotalMemory(false);
    long memoryUsed = memoryAfter - memoryBefore;
    
    System.Diagnostics.Debug.WriteLine($"Memory used: {memoryUsed} bytes");
}
```

## JavaScript Debugging in Legacy Apps

### Browser Developer Tools
1. **Open F12 Developer Tools**
2. **Sources tab** for JavaScript debugging
3. **Console tab** for JavaScript errors
4. **Network tab** for AJAX call debugging

### JavaScript Console Debugging
```javascript
// Debug JavaScript in legacy Method apps
console.log('Page loaded');
console.log('ViewState:', document.getElementById('__VIEWSTATE').value);

// Debug AJAX calls
function debugAjaxCall(url, data) {
    console.log('AJAX call to:', url);
    console.log('Data:', data);
    
    $.ajax({
        url: url,
        data: data,
        success: function(response) {
            console.log('AJAX success:', response);
        },
        error: function(xhr, status, error) {
            console.error('AJAX error:', error);
            console.error('Status:', status);
            console.error('Response:', xhr.responseText);
        }
    });
}
```

## Troubleshooting Common Issues

### Compilation Errors
- Check web.config compilation settings
- Verify .NET Framework version compatibility
- Clear temporary ASP.NET files
- Rebuild solution with clean

### Runtime Errors
- Enable custom errors mode="Off" for detailed error messages
- Check application pool identity and permissions
- Verify database connection strings
- Review IIS application pool settings

### ViewState Errors
- Check page enableViewState settings
- Verify control IDs are unique and consistent
- Review dynamic control creation timing
- Check ViewState MAC validation settings

**Next**: [App Pools & Services](./app-pools-services.md)
