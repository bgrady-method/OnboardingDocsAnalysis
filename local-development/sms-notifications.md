# SMS Notifications Setup (Optional)

This guide covers configuring SMS notifications for Method's local development environment.

## Overview

SMS notifications in Method's local development environment provide real-time alerts for critical system events, build failures, and application errors. This is an optional feature that enhances the development experience by providing immediate feedback on system status.

## Prerequisites

- Method account created and configured
- Critical projects built and healthy
- Development environment fully operational
- Valid phone number for SMS reception

## SMS Service Overview

### Method SMS Architecture

Method uses multiple SMS providers for reliability:

```
SMS Notification Stack
├── Primary Provider (Twilio)
├── Backup Provider (AWS SNS)
├── Internal Notification Service
└── Developer Alert Management
```

### Notification Types

Common SMS notifications include:
- **Build Status** - Success/failure notifications for critical builds
- **Service Alerts** - When Method services go down or recover
- **Database Issues** - Connection problems or query failures
- **Performance Alerts** - High CPU, memory, or response time alerts
- **Security Events** - Authentication failures or suspicious activity

## Configuration Steps

### Step 1: SMS Service Configuration

```powershell
# Configure Method SMS notification service
function Set-MethodSMSConfiguration {
    param(
        [Parameter(Mandatory)]
        [string]$PhoneNumber,
        
        [string]$Provider = "Twilio",
        [string]$AlertLevel = "Critical"
    )
    
    Write-Host "Configuring Method SMS notifications..." -ForegroundColor Green
    
    # Validate phone number format
    if ($PhoneNumber -notmatch '^\+?1?[2-9]\d{2}[2-9]\d{2}\d{4}$') {
        Write-Host "⚠ Phone number format may be invalid. Use format: +1234567890" -ForegroundColor Yellow
    }
    
    # SMS configuration
    $smsConfig = @{
        phoneNumber = $PhoneNumber
        provider = $Provider
        alertLevel = $AlertLevel
        enabled = $true
        environment = "Development"
        userId = $env:USERNAME
        machine = $env:COMPUTERNAME
        configuredDate = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    }
    
    try {
        # Save configuration to Method config service
        $configData = $smsConfig | ConvertTo-Json
        $response = Invoke-RestMethod -Uri "https://config.method.local/api/notifications/sms" `
                                    -Method Post `
                                    -Body $configData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            Write-Host "✓ SMS configuration saved successfully" -ForegroundColor Green
            Write-Host "Phone: $PhoneNumber" -ForegroundColor Gray
            Write-Host "Provider: $Provider" -ForegroundColor Gray
            Write-Host "Alert Level: $AlertLevel" -ForegroundColor Gray
            return $response
        } else {
            Write-Host "✗ SMS configuration failed: $($response.message)" -ForegroundColor Red
            return $null
        }
        
    } catch {
        Write-Host "✗ SMS configuration error: $($_.Exception.Message)" -ForegroundColor Red
        
        # Fallback: Save to local configuration
        Write-Host "Saving SMS configuration locally..." -ForegroundColor Yellow
        return Save-LocalSMSConfig -Config $smsConfig
    }
}

# Save SMS configuration locally (fallback)
function Save-LocalSMSConfig {
    param([hashtable]$Config)
    
    try {
        $configPath = "C:\MethodDev\sms-config.json"
        $Config | ConvertTo-Json | Out-File -FilePath $configPath -Encoding UTF8
        
        Write-Host "✓ SMS configuration saved locally: $configPath" -ForegroundColor Green
        return @{ success = $true; configPath = $configPath }
    } catch {
        Write-Host "✗ Failed to save local SMS configuration" -ForegroundColor Red
        return $null
    }
}

# Prompt for SMS configuration
Write-Host "=== Method SMS Notification Setup ===" -ForegroundColor Green
$phoneNumber = Read-Host "Enter your phone number (format: +1234567890)"
$alertLevel = Read-Host "Alert level (Critical/Warning/Info) [Critical]"

if ([string]::IsNullOrWhiteSpace($alertLevel)) {
    $alertLevel = "Critical"
}

# Configure SMS
$smsResult = Set-MethodSMSConfiguration -PhoneNumber $phoneNumber -AlertLevel $alertLevel
```

### Step 2: Test SMS Functionality

```powershell
# Test SMS notification functionality
function Test-MethodSMS {
    param(
        [string]$PhoneNumber,
        [string]$TestMessage = "Method development environment SMS test - $(Get-Date)"
    )
    
    Write-Host "Testing Method SMS functionality..." -ForegroundColor Green
    
    try {
        # Send test SMS via Method notification service
        $testData = @{
            phoneNumber = $PhoneNumber
            message = $TestMessage
            priority = "Normal"
            source = "DevelopmentSetup"
            timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
        } | ConvertTo-Json
        
        $response = Invoke-RestMethod -Uri "https://notifications.method.local/api/sms/send" `
                                    -Method Post `
                                    -Body $testData `
                                    -ContentType "application/json" `
                                    -TimeoutSec 30
        
        if ($response.success) {
            Write-Host "✓ Test SMS sent successfully" -ForegroundColor Green
            Write-Host "Message ID: $($response.messageId)" -ForegroundColor Gray
            Write-Host "Check your phone for the test message" -ForegroundColor Yellow
            return $true
        } else {
            Write-Host "✗ Test SMS failed: $($response.message)" -ForegroundColor Red
            return $false
        }
        
    } catch {
        Write-Host "✗ SMS test error: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# Test SMS with configured phone number
if ($smsResult -and $smsResult.success) {
    $testConfirm = Read-Host "Send test SMS to verify configuration? (y/n)"
    if ($testConfirm -eq 'y' -or $testConfirm -eq 'Y') {
        Test-MethodSMS -PhoneNumber $phoneNumber
    }
}
```

### Step 3: Configure Alert Rules

```powershell
# Configure SMS alert rules for development environment
function Set-MethodSMSAlerts {
    Write-Host "Configuring Method SMS alert rules..." -ForegroundColor Green
    
    # Standard development alert rules
    $alertRules = @(
        @{
            name = "Build Failures"
            condition = "BuildStatus = Failed"
            severity = "Critical"
            enabled = $true
            throttle = "5 minutes"
            description = "Alert when critical builds fail"
        },
        @{
            name = "Service Outages" 
            condition = "ServiceStatus = Down"
            severity = "Critical"
            enabled = $true
            throttle = "1 minute"
            description = "Alert when Method services become unavailable"
        },
        @{
            name = "Database Connection Failures"
            condition = "DatabaseStatus = ConnectionFailed"
            severity = "Critical"
            enabled = $true
            throttle = "2 minutes"
            description = "Alert when database connections fail"
        },
        @{
            name = "High CPU Usage"
            condition = "CPU > 90% for 5 minutes"
            severity = "Warning"
            enabled = $false  # Disabled by default for dev
            throttle = "10 minutes"
            description = "Alert when CPU usage is consistently high"
        },
        @{
            name = "Memory Exhaustion"
            condition = "MemoryUsage > 95%"
            severity = "Critical"
            enabled = $true
            throttle = "5 minutes"
            description = "Alert when memory usage is critically high"
        }
    )
    
    $configuredRules = @()
    
    foreach ($rule in $alertRules) {
        try {
            $ruleData = $rule | ConvertTo-Json
            
            # Configure alert rule via Method API
            $response = Invoke-RestMethod -Uri "https://notifications.method.local/api/alerts/rules" `
                                        -Method Post `
                                        -Body $ruleData `
                                        -ContentType "application/json"
            
            if ($response.success) {
                Write-Host "✓ Configured alert: $($rule.name)" -ForegroundColor Green
                $configuredRules += $rule
            } else {
                Write-Host "⚠ Failed to configure: $($rule.name)" -ForegroundColor Yellow
            }
            
        } catch {
            Write-Host "✗ Error configuring $($rule.name): $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    Write-Host "`nConfigured Alert Rules:" -ForegroundColor Green
    Write-Host "=======================" -ForegroundColor Green
    $configuredRules | ForEach-Object {
        $status = if ($_.enabled) { "Enabled" } else { "Disabled" }
        Write-Host "- $($_.name): $status ($($_.severity))" -ForegroundColor Gray
    }
    
    return $configuredRules
}

# Configure alert rules
$alertRules = Set-MethodSMSAlerts
```

## Advanced Configuration

### Custom Alert Scripts

```powershell
# Create custom alert scripts for specific scenarios
function New-MethodCustomAlert {
    param(
        [string]$AlertName,
        [string]$TriggerCondition, 
        [string]$SMSMessage
    )
    
    $scriptPath = "C:\MethodDev\Alerts\$AlertName.ps1"
    
    # Ensure alerts directory exists
    $alertsDir = "C:\MethodDev\Alerts"
    if (!(Test-Path $alertsDir)) {
        New-Item -ItemType Directory -Path $alertsDir -Force
    }
    
    $alertScript = @"
# Method Custom Alert: $AlertName
# Generated: $(Get-Date)

# Alert trigger condition: $TriggerCondition

function Send-$AlertName {
    param([string]`$Message = "$SMSMessage")
    
    try {
        # Send SMS notification
        `$smsData = @{
            phoneNumber = "$phoneNumber"
            message = "`$Message - $(Get-Date -Format 'HH:mm:ss')"
            priority = "High"
            source = "$AlertName"
        } | ConvertTo-Json
        
        Invoke-RestMethod -Uri "https://notifications.method.local/api/sms/send" \`
                        -Method Post \`
                        -Body `$smsData \`
                        -ContentType "application/json"
        
        Write-Host "Alert sent: $AlertName" -ForegroundColor Yellow
    } catch {
        Write-Host "Failed to send alert: $AlertName" -ForegroundColor Red
    }
}

# Example usage:
# Send-$AlertName -Message "Custom alert message"
"@

    $alertScript | Out-File -FilePath $scriptPath -Encoding UTF8
    Write-Host "✓ Custom alert script created: $scriptPath" -ForegroundColor Green
    
    return $scriptPath
}

# Create some useful custom alerts
Write-Host "`nCreating custom alert scripts..." -ForegroundColor Green

$customAlerts = @(
    @{
        Name = "DeploymentFailure"
        Condition = "Deployment process exits with error code"
        Message = "Method deployment failed on $env:COMPUTERNAME"
    },
    @{
        Name = "TestSuiteFailure"
        Condition = "NUnit test suite has failing tests"
        Message = "Method test suite has failing tests - check build output"
    },
    @{
        Name = "ServiceRecovery"
        Condition = "Previously failed service comes back online"
        Message = "Method service has recovered and is operational"
    }
)

foreach ($alert in $customAlerts) {
    New-MethodCustomAlert -AlertName $alert.Name -TriggerCondition $alert.Condition -SMSMessage $alert.Message
}
```

### Integration with Build Tools

```powershell
# Integrate SMS alerts with MSBuild/dotnet build
function Add-BuildSMSIntegration {
    Write-Host "Adding SMS integration to build processes..." -ForegroundColor Green
    
    # Create MSBuild target for SMS notifications
    $msbuildTarget = @"
<Project>
  <Target Name="SendBuildNotification" AfterTargets="Build">
    <PropertyGroup>
      <BuildStatus Condition="'`$(MSBuildLastTaskResult)' == 'true'">Success</BuildStatus>
      <BuildStatus Condition="'`$(MSBuildLastTaskResult)' != 'true'">Failed</BuildStatus>
    </PropertyGroup>
    
    <Exec Command="powershell -Command &quot;& { 
      try { 
        `$smsData = @{ 
          phoneNumber = '$phoneNumber'; 
          message = 'Build `$(BuildStatus): `$(MSBuildProjectName) on $env:COMPUTERNAME'; 
          priority = 'Normal' 
        } | ConvertTo-Json; 
        Invoke-RestMethod -Uri 'https://notifications.method.local/api/sms/send' -Method Post -Body `$smsData -ContentType 'application/json' 
      } catch { 
        Write-Host 'SMS notification failed' 
      } 
    }&quot;" 
          ContinueOnError="true" 
          Condition="'`$(BuildStatus)' == 'Failed'" />
  </Target>
</Project>
"@

    $targetPath = "C:\MethodDev\SMS.Build.targets"
    $msbuildTarget | Out-File -FilePath $targetPath -Encoding UTF8
    
    Write-Host "✓ MSBuild SMS integration created: $targetPath" -ForegroundColor Green
    Write-Host "Add this line to your .csproj files:" -ForegroundColor Yellow
    Write-Host "  <Import Project=`"C:\MethodDev\SMS.Build.targets`" />" -ForegroundColor Gray
}

# Add build integration
Add-BuildSMSIntegration
```

## SMS Alert Management

### Daily Alert Summary

```powershell
# Create daily SMS alert summary
function Send-DailySMSSummary {
    Write-Host "Generating daily SMS summary..." -ForegroundColor Green
    
    try {
        # Get today's alert statistics
        $today = Get-Date -Format "yyyy-MM-dd"
        $summaryData = @{
            date = $today
            requestType = "DailySummary"
        } | ConvertTo-Json
        
        $response = Invoke-RestMethod -Uri "https://notifications.method.local/api/alerts/summary" `
                                    -Method Post `
                                    -Body $summaryData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            $summary = "Method Daily Summary ($today): " +
                      "$($response.criticalAlerts) critical, " +
                      "$($response.warningAlerts) warnings, " +
                      "$($response.builds) builds, " +
                      "$($response.uptime)% uptime"
            
            # Send summary SMS
            $summaryRequest = @{
                phoneNumber = $phoneNumber
                message = $summary
                priority = "Low"
                source = "DailySummary"
            } | ConvertTo-Json
            
            Invoke-RestMethod -Uri "https://notifications.method.local/api/sms/send" `
                            -Method Post `
                            -Body $summaryRequest `
                            -ContentType "application/json"
            
            Write-Host "✓ Daily summary sent via SMS" -ForegroundColor Green
        }
        
    } catch {
        Write-Host "⚠ Daily summary failed: $($_.Exception.Message)" -ForegroundColor Yellow
    }
}

# Schedule daily summary (example using Windows Task Scheduler)
function Schedule-DailySMSSummary {
    Write-Host "Scheduling daily SMS summary..." -ForegroundColor Green
    
    $taskName = "Method Daily SMS Summary"
    $scriptPath = "C:\MethodDev\Alerts\DailySummary.ps1"
    
    # Create the daily summary script
    $summaryScript = @"
# Method Daily SMS Summary
# Auto-generated script

Import-Module "$PSScriptRoot\SMS-Functions.psm1" -Force
Send-DailySMSSummary
"@

    $summaryScript | Out-File -FilePath $scriptPath -Encoding UTF8
    
    Write-Host "✓ Daily summary script created: $scriptPath" -ForegroundColor Green
    Write-Host "To schedule this script, run:" -ForegroundColor Yellow
    Write-Host "schtasks /create /tn `"$taskName`" /tr `"powershell -File `"$scriptPath`"`" /sc daily /st 18:00" -ForegroundColor Gray
}

# Create daily summary scheduling
Schedule-DailySMSSummary
```

## SMS Configuration Validation

### Test All SMS Features

```powershell
# Comprehensive SMS configuration test
function Test-MethodSMSSetup {
    Write-Host "Method SMS Configuration Validation" -ForegroundColor Green
    Write-Host "====================================" -ForegroundColor Green
    
    $testResults = @()
    
    # Test 1: SMS Service Connectivity
    try {
        $response = Invoke-WebRequest -Uri "https://notifications.method.local/health" -UseBasicParsing -TimeoutSec 10
        $serviceTest = $response.StatusCode -eq 200
    } catch {
        $serviceTest = $false
    }
    
    $testResults += [PSCustomObject]@{
        Test = "SMS Service Connectivity"
        Status = if ($serviceTest) { "✓ Pass" } else { "✗ Fail" }
        Details = if ($serviceTest) { "Notification service accessible" } else { "Service not reachable" }
    }
    
    # Test 2: Configuration Storage
    $configExists = Test-Path "C:\MethodDev\sms-config.json"
    $testResults += [PSCustomObject]@{
        Test = "Configuration Storage"
        Status = if ($configExists) { "✓ Pass" } else { "⚠ Partial" }
        Details = if ($configExists) { "SMS configuration saved" } else { "No local configuration found" }
    }
    
    # Test 3: Phone Number Validation
    $phoneValid = $phoneNumber -match '^\+?1?[2-9]\d{2}[2-9]\d{2}\d{4}$'
    $testResults += [PSCustomObject]@{
        Test = "Phone Number Format"
        Status = if ($phoneValid) { "✓ Pass" } else { "⚠ Warning" }
        Details = if ($phoneValid) { "Phone number format is valid" } else { "Phone number format may be incorrect" }
    }
    
    # Test 4: Alert Rules
    $rulesConfigured = $alertRules -and $alertRules.Count -gt 0
    $testResults += [PSCustomObject]@{
        Test = "Alert Rules Configuration"
        Status = if ($rulesConfigured) { "✓ Pass" } else { "⚠ Partial" }
        Details = if ($rulesConfigured) { "$($alertRules.Count) alert rules configured" } else { "No alert rules configured" }
    }
    
    # Test 5: Custom Scripts
    $scriptsExist = Test-Path "C:\MethodDev\Alerts"
    $testResults += [PSCustomObject]@{
        Test = "Custom Alert Scripts"
        Status = if ($scriptsExist) { "✓ Pass" } else { "⚠ Partial" }
        Details = if ($scriptsExist) { "Custom alert scripts available" } else { "No custom scripts created" }
    }
    
    # Display results
    $testResults | Format-Table -AutoSize
    
    # Summary
    $passed = ($testResults | Where-Object { $_.Status -like "*Pass*" }).Count
    $total = $testResults.Count
    $percentage = [math]::Round(($passed / $total) * 100, 1)
    
    Write-Host "`nSMS Setup Summary: $passed/$total tests passed ($percentage%)" -ForegroundColor $(if ($passed -eq $total) { "Green" } elseif ($percentage -ge 70) { "Yellow" } else { "Red" })
    
    if ($passed -ge ($total * 0.8)) {
        Write-Host "✓ SMS notifications are properly configured!" -ForegroundColor Green
    } else {
        Write-Host "⚠ Some SMS features may need additional configuration" -ForegroundColor Yellow
    }
    
    return $passed -eq $total
}

# Run comprehensive SMS validation
$smsSetupComplete = Test-MethodSMSSetup
```

## SMS Management Commands

### Useful SMS Management Functions

```powershell
# Collection of useful SMS management functions
function Disable-MethodSMS {
    Write-Host "Disabling Method SMS notifications..." -ForegroundColor Yellow
    
    try {
        $disableData = @{ enabled = $false } | ConvertTo-Json
        Invoke-RestMethod -Uri "https://config.method.local/api/notifications/sms/disable" `
                        -Method Post `
                        -Body $disableData `
                        -ContentType "application/json"
        Write-Host "✓ SMS notifications disabled" -ForegroundColor Green
    } catch {
        Write-Host "✗ Failed to disable SMS notifications" -ForegroundColor Red
    }
}

function Enable-MethodSMS {
    Write-Host "Enabling Method SMS notifications..." -ForegroundColor Green
    
    try {
        $enableData = @{ enabled = $true } | ConvertTo-Json
        Invoke-RestMethod -Uri "https://config.method.local/api/notifications/sms/enable" `
                        -Method Post `
                        -Body $enableData `
                        -ContentType "application/json"
        Write-Host "✓ SMS notifications enabled" -ForegroundColor Green
    } catch {
        Write-Host "✗ Failed to enable SMS notifications" -ForegroundColor Red
    }
}

function Get-SMSHistory {
    param([int]$Days = 7)
    
    Write-Host "Retrieving SMS history for last $Days days..." -ForegroundColor Green
    
    try {
        $historyData = @{ days = $Days } | ConvertTo-Json
        $response = Invoke-RestMethod -Uri "https://notifications.method.local/api/sms/history" `
                                    -Method Post `
                                    -Body $historyData `
                                    -ContentType "application/json"
        
        if ($response.success) {
            Write-Host "SMS History (Last $Days days):" -ForegroundColor Green
            $response.messages | Format-Table -Property Date, Priority, Message, Status -AutoSize
        }
    } catch {
        Write-Host "✗ Failed to retrieve SMS history" -ForegroundColor Red
    }
}

Write-Host "`nSMS Management Functions Available:" -ForegroundColor Green
Write-Host "- Disable-MethodSMS: Temporarily disable SMS alerts" -ForegroundColor Gray
Write-Host "- Enable-MethodSMS: Re-enable SMS alerts" -ForegroundColor Gray  
Write-Host "- Get-SMSHistory: View recent SMS message history" -ForegroundColor Gray
Write-Host "- Test-MethodSMS: Send test SMS message" -ForegroundColor Gray
```

## Troubleshooting

### Common SMS Issues

#### Messages Not Received
- Verify phone number format is correct
- Check that SMS service is running and accessible
- Confirm alert rules are properly configured
- Verify no carrier-level SMS blocking

#### Duplicate Messages
- Check alert throttling settings
- Review alert rule conditions for overlaps
- Verify SMS service isn't configured multiple times

#### Service Unavailable
- Confirm Method notification service is running
- Check IIS application pools for notification service
- Verify network connectivity to SMS providers

#### High SMS Volume
- Review and disable unnecessary alert rules
- Increase throttling intervals for non-critical alerts
- Consider using email for less urgent notifications

## Best Practices

### SMS Usage Guidelines

- **Use SMS for critical alerts only** to avoid alert fatigue
- **Set appropriate throttling** to prevent message spam
- **Test regularly** to ensure notifications are working
- **Review and adjust** alert rules based on actual development needs
- **Consider cost** if using paid SMS services
- **Have fallback options** (email, Slack, etc.) for when SMS fails

### Security Considerations

- **Never include sensitive data** in SMS messages
- **Use secure channels** for SMS configuration
- **Regularly review** who has access to SMS alerts
- **Monitor SMS usage** to detect potential abuse

## Next Steps

After configuring SMS notifications:

1. **Test thoroughly** by triggering various alert conditions
2. **Fine-tune alert rules** based on your development workflow
3. **Integrate with CI/CD** pipelines for build notifications
4. **Set up monitoring** to ensure SMS service remains healthy
5. **Document custom alerts** for team members

## Maintenance

```powershell
# Weekly SMS maintenance script
function Start-SMSMaintenance {
    Write-Host "SMS Notification Maintenance" -ForegroundColor Green
    
    # 1. Test SMS functionality
    Write-Host "Testing SMS functionality..." -ForegroundColor Yellow
    Test-MethodSMS -PhoneNumber $phoneNumber -TestMessage "Weekly SMS maintenance test"
    
    # 2. Review alert history
    Write-Host "Reviewing alert history..." -ForegroundColor Yellow
    Get-SMSHistory -Days 7
    
    # 3. Check service health
    Write-Host "Checking SMS service health..." -ForegroundColor Yellow
    Test-MethodSMSSetup
    
    # 4. Clean up old logs
    Write-Host "Cleaning up old alert logs..." -ForegroundColor Yellow
    # Implementation depends on logging system
    
    Write-Host "✓ SMS maintenance completed" -ForegroundColor Green
}

# Run maintenance
# Start-SMSMaintenance
```

**Back to:** [Local Development](./README.md)
