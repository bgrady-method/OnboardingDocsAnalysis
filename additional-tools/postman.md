# Postman Setup

## API Testing with Postman for Method Development

Postman is essential for testing Method's REST APIs during development. This guide covers setup and usage.

## Installation

### Download and Install Postman
1. **Download** from [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
2. **Install** the desktop application (recommended over web version for local API testing)
3. **Create account** or sign in for collection syncing

### Alternative: Postman CLI
```powershell
# Install Postman CLI for automated testing
npm install -g postman
```

## Method API Collections

### Import Method Collections
Method provides pre-configured Postman collections for common API operations:

1. **Download Collections** from internal repository
2. **Import to Postman**:
   - File → Import
   - Select collection JSON files
   - Collections appear in left sidebar

### Core Method Collections
- **Authentication API** - Login, logout, token management
- **Account Management** - User account operations
- **Runtime Core** - Business logic and workflow APIs
- **Gateway API** - API routing and aggregation
- **App Store API** - Application marketplace operations

## Environment Configuration

### Create Method Development Environment
```json
{
  "name": "Method Local Development",
  "values": [
    {
      "key": "baseUrl",
      "value": "https://localhost:5001",
      "enabled": true
    },
    {
      "key": "authUrl", 
      "value": "https://localhost:5005",
      "enabled": true
    },
    {
      "key": "gatewayUrl",
      "value": "https://localhost:5003", 
      "enabled": true
    },
    {
      "key": "username",
      "value": "your-test-username",
      "enabled": true
    },
    {
      "key": "password",
      "value": "your-test-password",
      "enabled": true,
      "type": "secret"
    },
    {
      "key": "authToken",
      "value": "",
      "enabled": true,
      "type": "secret"
    },
    {
      "key": "refreshToken",
      "value": "",
      "enabled": true,
      "type": "secret"
    }
  ]
}
```

### Environment Variables Setup
```javascript
// Pre-request script for token management
if (!pm.environment.get("authToken") || pm.environment.get("tokenExpiry") < Date.now()) {
    // Token refresh logic
    const loginRequest = {
        url: pm.environment.get("authUrl") + "/api/auth/login",
        method: 'POST',
        header: {
            'Content-Type': 'application/json'
        },
        body: {
            mode: 'raw',
            raw: JSON.stringify({
                username: pm.environment.get("username"),
                password: pm.environment.get("password")
            })
        }
    };
    
    pm.sendRequest(loginRequest, function(err, response) {
        if (err) {
            console.log("Login failed:", err);
        } else {
            const jsonData = response.json();
            pm.environment.set("authToken", jsonData.accessToken);
            pm.environment.set("refreshToken", jsonData.refreshToken);
            pm.environment.set("tokenExpiry", Date.now() + (jsonData.expiresIn * 1000));
        }
    });
}
```

## Common API Testing Scenarios

### Authentication Flow
```javascript
// 1. Login Request
POST {{authUrl}}/api/auth/login
Content-Type: application/json

{
    "username": "{{username}}",
    "password": "{{password}}"
}

// Test script to capture token
pm.test("Login successful", function () {
    pm.response.to.have.status(200);
    const jsonData = pm.response.json();
    pm.environment.set("authToken", jsonData.accessToken);
    pm.environment.set("refreshToken", jsonData.refreshToken);
});
```

### Account Management APIs
```javascript
// Get user profile
GET {{baseUrl}}/api/account/profile
Authorization: Bearer {{authToken}}

// Update user settings
PUT {{baseUrl}}/api/account/settings
Authorization: Bearer {{authToken}}
Content-Type: application/json

{
    "theme": "dark",
    "notifications": true,
    "language": "en-US"
}
```

### Runtime Core APIs
```javascript
// Execute workflow
POST {{baseUrl}}/api/runtime/workflow/execute
Authorization: Bearer {{authToken}}
Content-Type: application/json

{
    "workflowId": "workflow-123",
    "parameters": {
        "param1": "value1",
        "param2": "value2"
    }
}

// Get workflow status
GET {{baseUrl}}/api/runtime/workflow/{{workflowId}}/status
Authorization: Bearer {{authToken}}
```

## Test Automation

### Collection Runner
1. **Select Collection** in Postman
2. **Click Runner** button
3. **Configure Environment** and data files
4. **Run Collection** to execute all requests

### Newman CLI Testing
```powershell
# Install Newman
npm install -g newman

# Run collection from command line
newman run "Method-APIs.postman_collection.json" `
  -e "Method-Local.postman_environment.json" `
  --reporters cli,json `
  --reporter-json-export results.json

# Run with data file
newman run collection.json -d test-data.csv -e environment.json
```

### Test Data Management
```csv
# test-data.csv for parameterized testing
username,password,expectedStatus
testuser1,password123,200
testuser2,wrongpass,401
admin,adminpass,200
```

## Advanced Testing Features

### Dynamic Variable Generation
```javascript
// Pre-request script for dynamic data
pm.environment.set("timestamp", Date.now());
pm.environment.set("randomId", Math.random().toString(36).substr(2, 9));
pm.environment.set("uuid", pm.variables.replaceIn("{{$guid}}"));

// Generate test data
const testData = {
    name: "Test User " + pm.variables.replaceIn("{{$randomFirstName}}"),
    email: pm.variables.replaceIn("{{$randomEmail}}"),
    phone: pm.variables.replaceIn("{{$randomPhoneNumber}}")
};

pm.environment.set("testUserData", JSON.stringify(testData));
```

### Response Validation
```javascript
// Comprehensive test script
pm.test("Response status is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has required fields", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('id');
    pm.expect(jsonData).to.have.property('name');
    pm.expect(jsonData).to.have.property('status');
});

pm.test("Response time is acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

pm.test("Content-Type is JSON", function () {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});
```

### Chain Requests
```javascript
// Test script to chain requests
pm.test("Create user and verify", function () {
    pm.response.to.have.status(201);
    const user = pm.response.json();
    
    // Store user ID for next request
    pm.environment.set("createdUserId", user.id);
    
    // Make follow-up request
    const verifyRequest = {
        url: pm.environment.get("baseUrl") + "/api/users/" + user.id,
        method: 'GET',
        header: {
            'Authorization': 'Bearer ' + pm.environment.get("authToken")
        }
    };
    
    pm.sendRequest(verifyRequest, function(err, response) {
        pm.test("User verification successful", function() {
            pm.expect(response.code).to.equal(200);
            const verifiedUser = response.json();
            pm.expect(verifiedUser.id).to.equal(user.id);
        });
    });
});
```

## Mock Servers

### Create Method API Mock
1. **Create Mock Server** in Postman
2. **Add Examples** to requests with sample responses
3. **Configure Mock URL** for frontend development
4. **Use Mock** when backend services are unavailable

### Mock Response Examples
```json
// Success response example
{
    "id": "user-123",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "status": "active",
    "createdAt": "2023-01-01T00:00:00Z",
    "lastLoginAt": "2023-12-01T10:30:00Z"
}

// Error response example  
{
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid email format",
        "details": [
            {
                "field": "email",
                "message": "Must be a valid email address"
            }
        ]
    }
}
```

## Performance Testing

### Load Testing with Newman
```powershell
# Simple load test
for ($i=1; $i -le 100; $i++) {
    newman run collection.json -e environment.json --suppress-exit-code
}

# Parallel execution
1..10 | ForEach-Object -Parallel {
    newman run collection.json -e environment.json --suppress-exit-code
} -ThrottleLimit 5
```

### Performance Monitoring
```javascript
// Monitor response times
pm.test("Performance check", function () {
    const responseTime = pm.response.responseTime;
    
    if (responseTime > 1000) {
        console.warn(`Slow response: ${responseTime}ms`);
    }
    
    pm.globals.set("lastResponseTime", responseTime);
    
    // Track average response time
    const times = pm.globals.get("responseTimes") || [];
    times.push(responseTime);
    pm.globals.set("responseTimes", times);
    
    if (times.length >= 10) {
        const avg = times.reduce((a, b) => a + b) / times.length;
        console.log(`Average response time: ${avg.toFixed(2)}ms`);
    }
});
```

## Debugging and Monitoring

### Console Logging
```javascript
// Debug request/response
console.log("Request URL:", pm.request.url.toString());
console.log("Request Headers:", JSON.stringify(pm.request.headers.toJSON()));
console.log("Request Body:", pm.request.body);

console.log("Response Status:", pm.response.status);
console.log("Response Headers:", JSON.stringify(pm.response.headers.toJSON()));
console.log("Response Body:", pm.response.text());
```

### Error Handling
```javascript
// Robust error handling
try {
    const jsonData = pm.response.json();
    
    if (pm.response.code >= 400) {
        console.error("API Error:", jsonData.error || jsonData.message);
        pm.test("Error response format", function() {
            pm.expect(jsonData).to.have.property('error');
        });
    } else {
        pm.test("Success response", function() {
            pm.response.to.have.status(200);
        });
    }
} catch (e) {
    console.error("Failed to parse JSON response:", e.message);
    pm.test("Valid JSON response", function() {
        pm.expect.fail("Response is not valid JSON");
    });
}
```

## Integration with CI/CD

### GitHub Actions Integration
```yaml
# .github/workflows/api-tests.yml
name: API Tests
on: [push, pull_request]

jobs:
  api-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          
      - name: Install Newman
        run: npm install -g newman
        
      - name: Run API Tests
        run: |
          newman run postman/collections/Method-APIs.json \
            -e postman/environments/CI.json \
            --reporters cli,junit \
            --reporter-junit-export results.xml
            
      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v1
        if: always()
        with:
          files: results.xml
```

### PowerShell Integration
```powershell
# Run API tests as part of Method setup validation
function Test-MethodAPIs {
    Write-Host "Running Method API validation tests..." -ForegroundColor Yellow
    
    # Ensure services are running
    $services = @("Method-UI", "Method-API", "Method-Auth")
    foreach ($service in $services) {
        $appPool = Get-WebAppPoolState -Name $service
        if ($appPool.Value -ne "Started") {
            Write-Host "Starting $service..." -ForegroundColor Yellow
            Start-WebAppPool -Name $service
            Start-Sleep -Seconds 5
        }
    }
    
    # Run Newman tests
    $result = newman run "C:\MethodDev\PostmanCollections\Method-HealthCheck.json" `
        -e "C:\MethodDev\PostmanCollections\Local.json" `
        --reporters cli,json `
        --reporter-json-export "C:\temp\api-test-results.json"
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ All API tests passed!" -ForegroundColor Green
    } else {
        Write-Host "✗ Some API tests failed" -ForegroundColor Red
        Get-Content "C:\temp\api-test-results.json" | ConvertFrom-Json | ForEach-Object {
            $_.run.failures | ForEach-Object {
                Write-Host "  Failed: $($_.source.name)" -ForegroundColor Red
            }
        }
    }
}
```

## Best Practices

### Organization
1. **Use Collections** to group related APIs
2. **Create Folders** within collections for logical grouping
3. **Name Requests** descriptively (e.g., "Create User - Valid Data")
4. **Use Environments** for different deployment targets

### Security
1. **Never commit** passwords or API keys to version control
2. **Use Environment Variables** for sensitive data
3. **Enable SSL verification** for production testing
4. **Rotate API keys** regularly

### Maintainability
1. **Document APIs** with descriptions and examples
2. **Version collections** for API changes
3. **Share collections** with team members
4. **Keep tests updated** with API changes

**Next**: [Portal & Public Pages](./portal-setup.md)
