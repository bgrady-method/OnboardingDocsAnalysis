# BrowserStack Setup

This guide covers configuring BrowserStack for cross-browser testing with Method applications using SSL certificates.

## Overview

BrowserStack provides cloud-based browser testing capabilities, allowing developers to test Method applications across different browsers, operating systems, and devices with proper SSL certificate handling.

## Prerequisites

- Method SSL certificates configured in IIS ([Add Certificates to IIS](./iis-certificates.md))
- BrowserStack account credentials (provided by Method)
- VPN connection for accessing local Method applications
- Local Method development environment running with HTTPS

## BrowserStack Account Access

### Account Credentials

**Sign-in URL**: [https://www.browserstack.com/users/sign_in](https://www.browserstack.com/users/sign_in)

**Shared Account Credentials**:
- **Email**: qa@method.me
- **Password**: Admin00+

⚠️ **Important**: This is a shared account, so please be mindful of your usage and ensure you're not kicking anyone out of their session.

### Account Features

- Live testing across multiple browsers and devices
- Automated testing capabilities
- Local testing for development environments
- Screenshot and video recording

## BrowserStack Local Setup

### 1. Download and Install BrowserStack Local

BrowserStack Local enables testing of local development environments:

1. **Download BrowserStack Local**
   - Visit [BrowserStack Local download page](https://www.browserstack.com/local-testing)
   - Download appropriate version for Windows
   - No installation required - it's a standalone executable

2. **Configure Local Testing**
   - Extract BrowserStackLocal.exe to a convenient location
   - Note the location for command line usage

### 2. Start BrowserStack Local Tunnel

1. **Basic Local Connection**
   ```cmd
   # Navigate to BrowserStack Local directory
   cd C:\path\to\browserstack\local
   
   # Start basic local tunnel (get access key from BrowserStack dashboard)
   BrowserStackLocal.exe --key YOUR_ACCESS_KEY
   ```

2. **SSL-Optimized Connection**
   ```cmd
   # Start with SSL and local HTTPS proxy support
   BrowserStackLocal.exe --key YOUR_ACCESS_KEY --force-local --local-https-proxy
   ```

3. **Corporate Network Configuration**
   ```cmd
   # If behind corporate firewall/proxy
   BrowserStackLocal.exe --key YOUR_ACCESS_KEY --proxy-host proxy.company.com --proxy-port 8080
   ```

### 3. Verify Connection

1. **Check BrowserStack Dashboard**
   - Look for "Local" indicator in BrowserStack dashboard
   - Green indicator confirms local tunnel is active

2. **Test Local Connectivity**
   ```cmd
   # Test that local sites are accessible
   curl http://localhost
   curl https://methodlocal.com
   ```

## SSL Certificate Configuration for BrowserStack

### Method Certificate Handling

Since Method uses a commercial SSL certificate (`*.methodlocal.com`), BrowserStack should automatically trust it without additional configuration.

#### Trusted Certificate Testing (Recommended)

1. **Use Method's Commercial Certificate**
   - Certificates are automatically trusted in BrowserStack browsers
   - No additional configuration required
   - Provides production-like testing environment

2. **Configure Method URLs**
   - Primary URL: `https://methodlocal.com`
   - API endpoints: `https://api.methodlocal.com`
   - Authentication: `https://auth.methodlocal.com`
   - Other Method services as configured

#### Self-Signed Certificate Support (If Needed)

If using custom/self-signed certificates for testing:

```cmd
# Start with certificate checking disabled (testing only)
BrowserStackLocal.exe --key YOUR_ACCESS_KEY --disable-cert-checks
```

**Note**: This is not recommended for production-like testing.

## Testing Configuration

### Method Application Testing Setup

1. **Start Local Method Environment**
   - Ensure IIS is running with SSL bindings
   - Verify HTTPS sites are accessible locally
   - Confirm VPN connection if required for backend services

2. **Test Preparation Checklist**
   - [ ] All Method services running and healthy
   - [ ] HTTPS certificates properly configured
   - [ ] BrowserStack Local tunnel active
   - [ ] Host file entries configured for local domains
   - [ ] VPN connected for backend service access

### BrowserStack Test Configuration

1. **Browser Selection Strategy**
   - **Primary Browsers**: Chrome (latest), Firefox (latest), Edge (latest)
   - **Safari Testing**: macOS Safari versions
   - **Mobile Browsers**: Chrome Mobile, Safari Mobile
   - **Legacy Support**: IE 11, older browser versions as needed

2. **Operating System Coverage**
   - **Windows**: Windows 10, Windows 11
   - **macOS**: Latest macOS versions
   - **Mobile**: iOS, Android (various versions)

## Common Testing Scenarios

### SSL Certificate Validation Testing

1. **Certificate Trust Verification**
   - Navigate to `https://methodlocal.com`
   - Verify no certificate warnings appear
   - Check certificate details show correct Method certificate
   - Test certificate chain validation

2. **Cross-Browser Certificate Behavior**
   - Test SSL handshake across different browsers
   - Verify consistent certificate trust indicators
   - Document any browser-specific SSL behaviors

3. **Mixed Content Testing**
   - Verify all resources load over HTTPS
   - Check for insecure content warnings
   - Test JavaScript and CSS resource loading over SSL

### Method-Specific Application Testing

1. **Authentication Flow Testing**
   - Test login over HTTPS at `https://signin.methodlocal.com`
   - Verify secure cookie handling
   - Check session management with SSL

2. **API Endpoint Testing**
   - Test HTTPS API calls to `https://api.methodlocal.com`
   - Verify CORS configuration with SSL
   - Check WebSocket connections over WSS

3. **Cross-Domain Communication**
   - Test communication between Method subdomains
   - Verify SSL works across different Method services
   - Check for any mixed content issues

## BrowserStack Testing Workflow

### Manual Testing Process

1. **Start BrowserStack Session**
   - Login to BrowserStack
   - Select desired browser/OS combination
   - Start live testing session

2. **Configure Local Testing**
   - Enable "Local Testing" in BrowserStack session
   - Verify local tunnel connection is active

3. **Test Method Applications**
   - Navigate to `https://methodlocal.com`
   - Test login and core functionality
   - Verify SSL behavior and certificate trust

### Automated Testing Integration

```javascript
// Example Selenium WebDriver configuration for BrowserStack
const { Builder, By, until } = require('selenium-webdriver');

const capabilities = {
  'browserName': 'Chrome',
  'browserVersion': 'latest',
  'os': 'Windows',
  'osVersion': '10',
  'browserstack.local': 'true',
  'browserstack.localIdentifier': 'Method-Local-Test'
};

const driver = new Builder()
  .usingServer('https://hub-cloud.browserstack.com/wd/hub')
  .withCapabilities(capabilities)
  .build();

async function testMethodSSL() {
  try {
    await driver.get('https://methodlocal.com');
    // Verify no certificate errors
    const title = await driver.getTitle();
    console.log('Method page loaded successfully:', title);
  } catch (error) {
    console.error('SSL or loading error:', error);
  } finally {
    await driver.quit();
  }
}
```

## Troubleshooting

### Common BrowserStack Issues

#### Local Connection Problems
- **Symptom**: Cannot access local Method sites
- **Solutions**:
  - Verify BrowserStack Local is running
  - Check firewall settings allow BrowserStack traffic
  - Confirm access key is correct
  - Restart BrowserStack Local tunnel

#### Certificate Errors in BrowserStack
- **Symptom**: SSL certificate warnings in BrowserStack browsers
- **Solutions**:
  - Ensure certificates are properly installed locally
  - Verify IIS bindings are correct
  - Check DNS resolution for local domains
  - Confirm certificate hasn't expired

#### Performance Issues
- **Symptom**: Slow loading or timeouts
- **Solutions**:
  - Use `--only-automate` flag for automated testing only
  - Configure proxy settings for corporate networks
  - Optimize local environment for testing
  - Check VPN impact on performance

### Debug Commands

```cmd
# Test BrowserStack Local connection with verbose output
BrowserStackLocal.exe --key YOUR_KEY --verbose

# Check local HTTPS connectivity
curl -k https://methodlocal.com

# Verify certificate details
powershell "Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like '*methodlocal*'}"
```

## Best Practices

### Testing Strategy

1. **Systematic Approach**
   - Start with major browsers (Chrome, Firefox, Edge)
   - Expand to mobile browsers
   - Include legacy browsers as needed by Method requirements

2. **SSL-Specific Test Cases**
   - Test certificate installation on fresh environments
   - Verify HTTPS redirects work correctly
   - Check SSL performance across different browsers
   - Test certificate expiration handling

3. **Documentation**
   - Record browser-specific SSL behaviors
   - Document any compatibility issues
   - Maintain testing checklist for SSL scenarios

### Security and Access Management

⚠️ **Important Security Notes:**
- Never use production certificates in BrowserStack testing
- Use development-specific certificates when possible
- Regularly rotate BrowserStack access keys
- Monitor BrowserStack usage for security compliance
- Log out of shared account when finished testing

## Integration with Method Development

### Development Workflow Integration

1. **Local Development Phase**
   - Configure SSL certificates locally
   - Test basic functionality in local browsers

2. **BrowserStack Testing Phase**
   - Test across target browser matrix
   - Verify SSL behavior consistency
   - Document any browser-specific issues

3. **Deployment Preparation**
   - Validate certificate configuration
   - Test production-like SSL setup
   - Confirm cross-browser compatibility

### CI/CD Integration

Consider integrating BrowserStack testing into automated workflows:

```yaml
# Example CI/CD step for BrowserStack testing
- name: BrowserStack SSL Testing
  run: |
    # Start BrowserStack Local
    ./BrowserStackLocal --key ${{ secrets.BROWSERSTACK_KEY }} --daemon start
    
    # Run SSL-specific tests
    npm run test:browserstack:ssl
    
    # Stop BrowserStack Local
    ./BrowserStackLocal --daemon stop
```

## Next Steps

After setting up BrowserStack:
- [Generate Custom Certificates](./custom-certificates.md) for additional testing scenarios
- Integrate automated SSL testing into CI/CD pipeline
- Document browser-specific SSL behaviors for Method applications
- Set up regular cross-browser SSL compatibility testing

## Additional Resources

- **BrowserStack Local Documentation**: [Local Testing Guide](https://www.browserstack.com/docs/live/local-testing)
- **SSL Testing Best Practices**: Focus on certificate trust, mixed content, and performance
- **Method-Specific Testing**: Prioritize authentication flows and API connectivity over HTTPS
