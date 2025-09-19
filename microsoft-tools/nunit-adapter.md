# NUnit Test Adapter

This guide covers installing and configuring the NUnit Test Adapter for Visual Studio to run Method's unit tests.

## Overview

Method uses NUnit as the primary unit testing framework for .NET applications. The NUnit Test Adapter integrates NUnit with Visual Studio's Test Explorer, enabling test discovery, execution, and debugging directly within the IDE.

## Prerequisites

- Visual Studio 2022 Enterprise installed
- NuGet feeds configured (for Method test packages)
- Administrator access for extension installation
- Method project with existing unit tests

## NUnit Test Adapter Installation

### Through Visual Studio Extension Manager

#### Install from Extension Manager

1. **Open Visual Studio**
2. **Go to Extensions Menu:**
   - **Extensions** → **Manage Extensions**
   - Or press `Ctrl+Shift+X`

3. **Search and Install:**
   - Search for "NUnit 3 Test Adapter"
   - Select "NUnit 3 Test Adapter" by Charlie Poole
   - Click **Download**
   - Restart Visual Studio when prompted

### Through NuGet Package Manager

#### Project-Level Installation

```xml
<!-- Add to test projects' .csproj files -->
<PackageReference Include="NUnit" Version="3.13.3" />
<PackageReference Include="NUnit3TestAdapter" Version="4.5.0" />
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.7.2" />
```

#### PowerShell Installation

```powershell
# Install NUnit packages in test projects
cd C:\MethodDev\YourTestProject
dotnet add package NUnit --version 3.13.3
dotnet add package NUnit3TestAdapter --version 4.5.0
dotnet add package Microsoft.NET.Test.Sdk --version 17.7.2

# Verify installation
dotnet list package
```

### Verify Installation

```powershell
# Check if adapter is installed in Visual Studio
$vsPath = "${env:ProgramFiles}\Microsoft Visual Studio\2022\Enterprise\Common7\IDE\Extensions"
if (Test-Path "$vsPath\*\NUnit*") {
    Write-Host "✓ NUnit Test Adapter found in Visual Studio" -ForegroundColor Green
} else {
    Write-Host "✗ NUnit Test Adapter not found" -ForegroundColor Red
}
```

## Method Test Project Configuration

### Standard Test Project Structure

```
Method.ProjectName.Tests/
├── Method.ProjectName.Tests.csproj
├── Properties/
│   └── AssemblyInfo.cs
├── TestBase.cs
├── UnitTests/
│   ├── ServiceTests/
│   ├── ControllerTests/
│   └── ModelTests/
├── IntegrationTests/
└── TestData/
```

### Test Project Template (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="NUnit" Version="3.13.3" />
    <PackageReference Include="NUnit3TestAdapter" Version="4.5.0" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.7.2" />
    <PackageReference Include="Moq" Version="4.20.69" />
    <PackageReference Include="FluentAssertions" Version="6.12.0" />
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="7.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Method.ProjectName\Method.ProjectName.csproj" />
  </ItemGroup>

  <ItemGroup>
    <!-- Method-specific test packages -->
    <PackageReference Include="Method.Testing.Utilities" Version="1.1.0" />
    <PackageReference Include="Method.Testing.Data" Version="1.0.5" />
  </ItemGroup>

</Project>
```

### Base Test Class

```csharp
// TestBase.cs - Common base class for Method unit tests
using NUnit.Framework;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Method.Testing.Utilities;

namespace Method.ProjectName.Tests
{
    [TestFixture]
    public abstract class TestBase
    {
        protected IServiceProvider ServiceProvider { get; private set; }
        protected ILogger Logger { get; private set; }
        
        [OneTimeSetUp]
        public virtual void OneTimeSetUp()
        {
            var services = new ServiceCollection();
            ConfigureServices(services);
            ServiceProvider = services.BuildServiceProvider();
            
            var loggerFactory = ServiceProvider.GetRequiredService<ILoggerFactory>();
            Logger = loggerFactory.CreateLogger(GetType());
        }
        
        [OneTimeTearDown]
        public virtual void OneTimeTearDown()
        {
            if (ServiceProvider is IDisposable disposable)
            {
                disposable.Dispose();
            }
        }
        
        [SetUp]
        public virtual void SetUp()
        {
            // Per-test setup
            Logger.LogInformation("Starting test: {TestName}", TestContext.CurrentContext.Test.Name);
        }
        
        [TearDown]
        public virtual void TearDown()
        {
            // Per-test cleanup
            Logger.LogInformation("Completed test: {TestName}", TestContext.CurrentContext.Test.Name);
        }
        
        protected virtual void ConfigureServices(IServiceCollection services)
        {
            // Configure dependency injection for tests
            services.AddLogging(builder => builder.AddConsole().AddDebug());
            
            // Add Method-specific test services
            services.AddMethodTestingUtilities();
        }
        
        protected T GetService<T>() where T : notnull
        {
            return ServiceProvider.GetRequiredService<T>();
        }
    }
}
```

## Test Explorer Configuration

### Visual Studio Test Settings

#### Configure Test Explorer

1. **Open Test Explorer:**
   - **Test** → **Test Explorer**
   - Or press `Ctrl+E, T`

2. **Configure Settings:**
   - **Test** → **Options** → **Test**
   - **General Settings:**
     - Enable "Show test hierarchy"
     - Enable "Group by project"
     - Set "Default test timeout" to 30 seconds

3. **NUnit-Specific Settings:**
   - **Test** → **Options** → **Test** → **NUnit**
   - Enable "Include NUnit output in Test Results"
   - Set "Number of test workers" to `auto`

### Test Discovery Configuration

#### NUnit Settings File

Create `NUnit.settings` in test project root:

```xml
<?xml version="1.0" encoding="utf-8"?>
<NUnitSettings>
  <TestRunner>
    <!-- Configure test execution -->
    <NumberOfTestWorkers>auto</NumberOfTestWorkers>
    <WorkDirectory>.</WorkDirectory>
    <InternalTraceLevel>Info</InternalTraceLevel>
  </TestRunner>
  
  <TestOutput>
    <!-- Configure test output -->
    <Output>TestResult</Output>
    <WorkDirectory>.</WorkDirectory>
  </TestOutput>
  
  <CodeCoverage>
    <!-- Enable code coverage -->
    <Enabled>true</Enabled>
    <Threshold>80</Threshold>
  </CodeCoverage>
</NUnitSettings>
```

## Method Testing Patterns

### Unit Test Example

```csharp
using NUnit.Framework;
using FluentAssertions;
using Moq;
using Microsoft.Extensions.Logging;
using Method.ProjectName.Services;
using Method.ProjectName.Models;

namespace Method.ProjectName.Tests.UnitTests.ServiceTests
{
    [TestFixture]
    [Category("Unit")]
    public class UserServiceTests : TestBase
    {
        private Mock<IUserRepository> _mockUserRepository;
        private Mock<IEmailService> _mockEmailService;
        private UserService _userService;
        
        [SetUp]
        public override void SetUp()
        {
            base.SetUp();
            
            _mockUserRepository = new Mock<IUserRepository>();
            _mockEmailService = new Mock<IEmailService>();
            
            _userService = new UserService(
                _mockUserRepository.Object,
                _mockEmailService.Object,
                GetService<ILogger<UserService>>()
            );
        }
        
        [Test]
        public async Task CreateUser_ValidUser_ShouldCreateSuccessfully()
        {
            // Arrange
            var newUser = new CreateUserRequest
            {
                Email = "test@method.com",
                FirstName = "John",
                LastName = "Doe"
            };
            
            var expectedUser = new User
            {
                Id = 1,
                Email = newUser.Email,
                FirstName = newUser.FirstName,
                LastName = newUser.LastName
            };
            
            _mockUserRepository
                .Setup(r => r.CreateAsync(It.IsAny<User>()))
                .ReturnsAsync(expectedUser);
                
            _mockEmailService
                .Setup(e => e.SendWelcomeEmailAsync(It.IsAny<string>(), It.IsAny<string>()))
                .Returns(Task.CompletedTask);
            
            // Act
            var result = await _userService.CreateUserAsync(newUser);
            
            // Assert
            result.Should().NotBeNull();
            result.Email.Should().Be(newUser.Email);
            result.FirstName.Should().Be(newUser.FirstName);
            result.LastName.Should().Be(newUser.LastName);
            
            _mockUserRepository.Verify(r => r.CreateAsync(It.IsAny<User>()), Times.Once);
            _mockEmailService.Verify(e => e.SendWelcomeEmailAsync(newUser.Email, newUser.FirstName), Times.Once);
        }
        
        [Test]
        [TestCase("")]
        [TestCase("invalid-email")]
        [TestCase("@method.com")]
        public void CreateUser_InvalidEmail_ShouldThrowArgumentException(string invalidEmail)
        {
            // Arrange
            var newUser = new CreateUserRequest
            {
                Email = invalidEmail,
                FirstName = "John",
                LastName = "Doe"
            };
            
            // Act & Assert
            _userService.Invoking(s => s.CreateUserAsync(newUser))
                .Should().ThrowAsync<ArgumentException>()
                .WithMessage("*email*");
        }
    }
}
```

### Integration Test Example

```csharp
using NUnit.Framework;
using FluentAssertions;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.EntityFrameworkCore;
using Method.ProjectName.Data;
using Method.ProjectName.Services;
using Method.Testing.Data;

namespace Method.ProjectName.Tests.IntegrationTests
{
    [TestFixture]
    [Category("Integration")]
    public class UserServiceIntegrationTests : TestBase
    {
        private ApplicationDbContext _dbContext;
        private UserService _userService;
        
        protected override void ConfigureServices(IServiceCollection services)
        {
            base.ConfigureServices(services);
            
            // Use in-memory database for testing
            services.AddDbContext<ApplicationDbContext>(options =>
                options.UseInMemoryDatabase($"TestDb_{Guid.NewGuid()}"));
                
            services.AddScoped<IUserRepository, UserRepository>();
            services.AddScoped<UserService>();
            
            // Mock external dependencies
            services.AddMethodTestingEmail(); // Mock email service
        }
        
        [SetUp]
        public override void SetUp()
        {
            base.SetUp();
            
            _dbContext = GetService<ApplicationDbContext>();
            _userService = GetService<UserService>();
            
            // Ensure clean database state
            _dbContext.Database.EnsureCreated();
        }
        
        [TearDown]
        public override void TearDown()
        {
            // Clean up database
            _dbContext.Database.EnsureDeleted();
            base.TearDown();
        }
        
        [Test]
        public async Task CreateUser_WithDatabase_ShouldPersistCorrectly()
        {
            // Arrange
            var createRequest = new CreateUserRequest
            {
                Email = "integration@method.com",
                FirstName = "Integration",
                LastName = "Test"
            };
            
            // Act
            var createdUser = await _userService.CreateUserAsync(createRequest);
            
            // Assert
            createdUser.Should().NotBeNull();
            createdUser.Id.Should().BeGreaterThan(0);
            
            // Verify database persistence
            var persistedUser = await _dbContext.Users.FindAsync(createdUser.Id);
            persistedUser.Should().NotBeNull();
            persistedUser.Email.Should().Be(createRequest.Email);
        }
    }
}
```

## Running Tests

### Through Test Explorer

1. **Build Solution** to discover tests
2. **Right-click in Test Explorer** → **Run All Tests**
3. **Filter tests** using the search box or categories
4. **Debug tests** by right-clicking → **Debug**

### Through Command Line

```powershell
# Run all tests in a project
cd C:\MethodDev\YourTestProject
dotnet test

# Run tests with specific filter
dotnet test --filter "Category=Unit"
dotnet test --filter "ClassName~UserService"

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run tests with detailed output
dotnet test --logger "console;verbosity=detailed"
```

### Through Visual Studio Code

```powershell
# Install .NET Test Explorer extension
# Then use Command Palette (Ctrl+Shift+P)
# Run: ".NET: Run All Tests"
```

## Test Categories and Organization

### Method Test Categories

```csharp
// Use consistent categories for test organization
[Category("Unit")]          // Fast, isolated unit tests
[Category("Integration")]   // Database/service integration tests
[Category("E2E")]          // End-to-end tests
[Category("Performance")]  // Performance/load tests
[Category("Security")]     // Security-focused tests
```

### Test Naming Conventions

```csharp
// Method test naming pattern: MethodName_Scenario_ExpectedBehavior
[Test]
public void CreateUser_ValidInput_ShouldReturnCreatedUser() { }

[Test] 
public void CreateUser_InvalidEmail_ShouldThrowArgumentException() { }

[Test]
public void GetUser_NonExistentId_ShouldReturnNull() { }
```

## Troubleshooting

### Tests Not Discovered

#### Check Test Adapter Installation

```powershell
# Verify NUnit adapter packages
dotnet list package | Select-String "NUnit"

# Expected output:
# NUnit                               3.13.3
# NUnit3TestAdapter                   4.5.0
# Microsoft.NET.Test.Sdk             17.7.2
```

#### Rebuild and Refresh

```powershell
# Clean and rebuild solution
dotnet clean
dotnet build

# Refresh Test Explorer
# In Visual Studio: Test → Windows → Test Explorer → Refresh
```

### Test Execution Issues

#### Check Test Project Configuration

```xml
<!-- Ensure test project has correct SDK -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
</Project>
```

#### Verify Test Runner

```powershell
# Test with command line to isolate issues
cd YourTestProject
dotnet test --verbosity normal

# Check for specific errors in output
```

### Method-Specific Issues

#### Missing Method Test Packages

```powershell
# Verify access to Method internal packages
nuget list Method.Testing -Source "Method-Internal" -PreRelease

# If missing, check NuGet feed configuration
```

#### Database Connection Issues

```csharp
// Use in-memory database for unit tests
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseInMemoryDatabase($"TestDb_{Guid.NewGuid()}"));

// Or configure test database connection
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=MethodTest;Trusted_Connection=true;"));
```

## Performance Optimization

### Parallel Test Execution

```xml
<!-- In .runsettings file -->
<RunSettings>
  <RunConfiguration>
    <MaxCpuCount>4</MaxCpuCount>
    <DisableAppDomain>true</DisableAppDomain>
  </RunConfiguration>
  <NUnit>
    <NumberOfTestWorkers>4</NumberOfTestWorkers>
  </NUnit>
</RunSettings>
```

### Test Data Management

```csharp
// Use object builders for test data
public class UserBuilder
{
    private User _user = new User();
    
    public UserBuilder WithEmail(string email)
    {
        _user.Email = email;
        return this;
    }
    
    public UserBuilder WithName(string firstName, string lastName)
    {
        _user.FirstName = firstName;
        _user.LastName = lastName;
        return this;
    }
    
    public User Build() => _user;
}

// Usage in tests
var user = new UserBuilder()
    .WithEmail("test@method.com")
    .WithName("John", "Doe")
    .Build();
```

## Best Practices

### Test Organization

- **Separate unit and integration tests** into different directories
- **Use consistent naming patterns** for test classes and methods
- **Group related tests** using TestFixture classes
- **Use categories** to enable selective test execution

### Test Writing

- **Follow AAA pattern** (Arrange, Act, Assert)
- **One assertion per test** when possible
- **Use descriptive test names** that explain the scenario
- **Mock external dependencies** in unit tests
- **Use FluentAssertions** for readable assertions

### Maintenance

- **Keep tests fast** (unit tests should run in milliseconds)
- **Make tests independent** (no shared state between tests)
- **Clean up resources** properly in TearDown methods
- **Update tests** when code changes

## Verification

### Test Installation Verification

```powershell
# Comprehensive test setup verification
function Test-NUnitSetup {
    Write-Host "NUnit Test Adapter Verification" -ForegroundColor Green
    
    # Check Visual Studio extension
    $vsExtensions = Get-ChildItem "${env:ProgramFiles}\Microsoft Visual Studio\2022\Enterprise\Common7\IDE\Extensions" -Recurse -Filter "*nunit*" 2>$null
    if ($vsExtensions) {
        Write-Host "✓ NUnit extension found in Visual Studio" -ForegroundColor Green
    } else {
        Write-Host "✗ NUnit extension not found in Visual Studio" -ForegroundColor Red
    }
    
    # Check for test project
    $testProjects = Get-ChildItem -Recurse -Filter "*.Tests.csproj"
    if ($testProjects) {
        Write-Host "✓ Found $($testProjects.Count) test project(s)" -ForegroundColor Green
        
        foreach ($project in $testProjects) {
            Write-Host "  - $($project.FullName)" -ForegroundColor Gray
        }
    } else {
        Write-Host "⚠ No test projects found" -ForegroundColor Yellow
    }
    
    # Test dotnet test command
    try {
        $testOutput = dotnet test --list-tests 2>&1
        if ($testOutput -match "The following Tests are available:") {
            Write-Host "✓ dotnet test command working" -ForegroundColor Green
        } else {
            Write-Host "⚠ No tests found via dotnet test" -ForegroundColor Yellow
        }
    } catch {
        Write-Host "✗ dotnet test command failed" -ForegroundColor Red
    }
}

# Run verification
Test-NUnitSetup
```

## Next Steps

After installing NUnit Test Adapter:

1. **Add IIS Components:** [URL Rewrite Installation](./url-rewrite.md)
2. **Create Test Projects:** Set up unit test projects for Method applications
3. **Write Sample Tests:** Create basic tests to verify setup
4. **Configure Environment:** [Environment Setup](../environment-setup/README.md)

## Additional Resources

- **NUnit Documentation:** https://docs.nunit.org/
- **NUnit Test Adapter GitHub:** https://github.com/nunit/nunit3-vs-adapter
- **Method Testing Guidelines:** Internal wiki
- **FluentAssertions Documentation:** https://fluentassertions.com/

**Back to:** [Microsoft Tools](./README.md)
