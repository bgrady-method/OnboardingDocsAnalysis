# Portal & Public Pages

## Method Portal and Public Pages Setup

This guide covers setting up Method's customer portal and public-facing pages for local development.

## Overview

Method's portal ecosystem includes:

- **Customer Portal** - Account management, billing, app marketplace
- **Public Pages** - Marketing pages, documentation, support
- **Email Templates** - Transactional email rendering
- **Sign-up Flow** - New customer registration process

## Prerequisites

- Method core development environment configured
- IIS with appropriate application pools
- SQL Server with Method databases restored
- Node.js and npm for frontend build processes

## Portal Applications

### Customer Portal Setup

#### 1. Portal Application Structure
```
C:\MethodDev\Portal\
├── src\
│   ├── components\     # React components
│   ├── pages\         # Portal pages
│   ├── services\      # API service calls
│   ├── styles\        # CSS and styling
│   └── utils\         # Helper utilities
├── public\            # Static assets
├── build\             # Built application
└── config\            # Configuration files
```

#### 2. Environment Configuration
```json
// config/local.json
{
  "api": {
    "baseUrl": "https://localhost:5001",
    "authUrl": "https://localhost:5005",
    "gatewayUrl": "https://localhost:5003"
  },
  "portal": {
    "title": "Method Portal - Local",
    "theme": "default",
    "features": {
      "billing": true,
      "appStore": true,
      "support": true,
      "analytics": true
    }
  },
  "stripe": {
    "publishableKey": "pk_test_...",
    "webhookSecret": "whsec_test_..."
  },
  "auth": {
    "sessionTimeout": 3600,
    "rememberMeDuration": 2592000
  }
}
```

#### 3. IIS Configuration
```powershell
# Create Portal application pool
New-WebAppPool -Name "Method-Portal"
Set-ItemProperty -Path "IIS:\AppPools\Method-Portal" -Name managedRuntimeVersion -Value "v4.0"
Set-ItemProperty -Path "IIS:\AppPools\Method-Portal" -Name processModel.identityType -Value NetworkService

# Create Portal site
New-WebSite -Name "Method-Portal" -Port 8080 -PhysicalPath "C:\MethodDev\Portal\build"
Set-WebBinding -Name "Method-Portal" -BindingInformation "*:8080:" -PropertyName Port -Value 8080

# Add HTTPS binding
New-WebBinding -Name "Method-Portal" -Protocol https -Port 8443
```

### Public Pages Setup

#### 1. Marketing Site Structure
```
C:\MethodDev\PublicPages\
├── src\
│   ├── pages\         # Static and dynamic pages
│   ├── components\    # Reusable components
│   ├── assets\        # Images, fonts, etc.
│   └── data\          # Content and configuration
├── dist\              # Built static site
└── templates\         # Page templates
```

#### 2. Build Process
```powershell
# Navigate to public pages directory
Set-Location "C:\MethodDev\PublicPages"

# Install dependencies
npm install

# Development build with hot reload
npm run dev

# Production build
npm run build

# Start development server
npm run serve
```

#### 3. Content Management
```yaml
# data/pages.yaml - Page configuration
home:
  title: "Method - Business Process Platform"
  description: "Streamline your business processes"
  template: "landing"
  sections:
    - hero
    - features
    - testimonials
    - pricing

pricing:
  title: "Method Pricing"
  description: "Choose the plan that's right for you"
  template: "pricing"
  plans:
    - name: "Starter"
      price: 29
      features: ["Basic automation", "5 users", "Email support"]
    - name: "Professional"
      price: 99
      features: ["Advanced automation", "25 users", "Priority support"]
```

## Email Templates and Public Pages

### Email Template Development

#### 1. Template Structure
```html
<!-- templates/email/welcome.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to Method</title>
    <style>
        /* Inline CSS for email compatibility */
        .container { max-width: 600px; margin: 0 auto; }
        .header { background-color: #0066cc; color: white; padding: 20px; }
        .content { padding: 20px; }
        .footer { background-color: #f5f5f5; padding: 10px; text-align: center; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Welcome to Method!</h1>
        </div>
        <div class="content">
            <p>Hello {{customerName}},</p>
            <p>Thank you for joining Method. Your account has been created successfully.</p>
            <p>
                <a href="{{portalUrl}}/login" style="background-color: #0066cc; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;">
                    Access Your Account
                </a>
            </p>
        </div>
        <div class="footer">
            <p>&copy; 2023 Method. All rights reserved.</p>
            <p><a href="{{unsubscribeUrl}}">Unsubscribe</a></p>
        </div>
    </div>
</body>
</html>
```

#### 2. Template Testing
```javascript
// Test email template rendering
const templateEngine = require('./src/utils/templateEngine');

function testEmailTemplate() {
    const template = 'welcome';
    const data = {
        customerName: 'John Doe',
        portalUrl: 'https://portal.method.com',
        unsubscribeUrl: 'https://portal.method.com/unsubscribe?token=abc123'
    };
    
    const renderedHtml = templateEngine.render(template, data);
    console.log('Rendered email:', renderedHtml);
    
    // Save to file for browser testing
    require('fs').writeFileSync('./test-email.html', renderedHtml);
}

testEmailTemplate();
```

### Public Sign-up Flow

#### 1. Registration Process
```javascript
// src/pages/signup.js
import React, { useState } from 'react';
import { validateEmail, validatePassword } from '../utils/validation';

const SignupPage = () => {
    const [formData, setFormData] = useState({
        firstName: '',
        lastName: '',
        email: '',
        password: '',
        confirmPassword: '',
        company: '',
        agreeToTerms: false
    });
    
    const [errors, setErrors] = useState({});
    const [loading, setLoading] = useState(false);
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        setLoading(true);
        
        // Validate form
        const validationErrors = validateSignupForm(formData);
        if (Object.keys(validationErrors).length > 0) {
            setErrors(validationErrors);
            setLoading(false);
            return;
        }
        
        try {
            // Call signup API
            const response = await fetch('/api/auth/signup', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(formData)
            });
            
            if (response.ok) {
                window.location.href = '/signup/success';
            } else {
                const error = await response.json();
                setErrors({ general: error.message });
            }
        } catch (error) {
            setErrors({ general: 'An error occurred. Please try again.' });
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div className="signup-container">
            <form onSubmit={handleSubmit}>
                <h1>Create Your Method Account</h1>
                
                {errors.general && (
                    <div className="error-message">{errors.general}</div>
                )}
                
                <div className="form-group">
                    <label>First Name</label>
                    <input
                        type="text"
                        value={formData.firstName}
                        onChange={(e) => setFormData({...formData, firstName: e.target.value})}
                        required
                    />
                    {errors.firstName && <span className="error">{errors.firstName}</span>}
                </div>
                
                {/* Additional form fields */}
                
                <button type="submit" disabled={loading}>
                    {loading ? 'Creating Account...' : 'Create Account'}
                </button>
            </form>
        </div>
    );
};

export default SignupPage;
```

#### 2. Account Verification
```javascript
// src/pages/verify-email.js
import React, { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';

const VerifyEmailPage = () => {
    const { token } = useParams();
    const [status, setStatus] = useState('verifying');
    const [message, setMessage] = useState('');
    
    useEffect(() => {
        verifyEmail(token);
    }, [token]);
    
    const verifyEmail = async (verificationToken) => {
        try {
            const response = await fetch(`/api/auth/verify-email/${verificationToken}`, {
                method: 'POST'
            });
            
            if (response.ok) {
                setStatus('success');
                setMessage('Your email has been verified successfully!');
            } else {
                const error = await response.json();
                setStatus('error');
                setMessage(error.message || 'Verification failed');
            }
        } catch (error) {
            setStatus('error');
            setMessage('An error occurred during verification');
        }
    };
    
    return (
        <div className="verify-container">
            {status === 'verifying' && (
                <div>
                    <h1>Verifying your email...</h1>
                    <div className="spinner"></div>
                </div>
            )}
            
            {status === 'success' && (
                <div>
                    <h1>Email Verified!</h1>
                    <p>{message}</p>
                    <a href="/login" className="btn-primary">Continue to Login</a>
                </div>
            )}
            
            {status === 'error' && (
                <div>
                    <h1>Verification Failed</h1>
                    <p>{message}</p>
                    <a href="/signup" className="btn-secondary">Sign Up Again</a>
                </div>
            )}
        </div>
    );
};

export default VerifyEmailPage;
```

## Portal Features Development

### Dashboard Components

#### 1. Account Overview
```javascript
// src/components/dashboard/AccountOverview.js
import React, { useState, useEffect } from 'react';

const AccountOverview = () => {
    const [accountData, setAccountData] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        fetchAccountData();
    }, []);
    
    const fetchAccountData = async () => {
        try {
            const response = await fetch('/api/account/overview', {
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('authToken')}`
                }
            });
            
            if (response.ok) {
                const data = await response.json();
                setAccountData(data);
            }
        } catch (error) {
            console.error('Failed to fetch account data:', error);
        } finally {
            setLoading(false);
        }
    };
    
    if (loading) return <div>Loading...</div>;
    if (!accountData) return <div>Unable to load account data</div>;
    
    return (
        <div className="account-overview">
            <h2>Account Overview</h2>
            
            <div className="stats-grid">
                <div className="stat-card">
                    <h3>Active Users</h3>
                    <span className="stat-value">{accountData.activeUsers}</span>
                </div>
                
                <div className="stat-card">
                    <h3>Apps Deployed</h3>
                    <span className="stat-value">{accountData.appsDeployed}</span>
                </div>
                
                <div className="stat-card">
                    <h3>Storage Used</h3>
                    <span className="stat-value">{accountData.storageUsed} GB</span>
                </div>
                
                <div className="stat-card">
                    <h3>API Calls</h3>
                    <span className="stat-value">{accountData.apiCallsThisMonth}</span>
                    <span className="stat-period">This Month</span>
                </div>
            </div>
            
            <div className="recent-activity">
                <h3>Recent Activity</h3>
                <ul>
                    {accountData.recentActivity.map((activity, index) => (
                        <li key={index} className="activity-item">
                            <span className="activity-time">{activity.timestamp}</span>
                            <span className="activity-description">{activity.description}</span>
                        </li>
                    ))}
                </ul>
            </div>
        </div>
    );
};

export default AccountOverview;
```

#### 2. Billing Management
```javascript
// src/components/billing/BillingManagement.js
import React, { useState, useEffect } from 'react';

const BillingManagement = () => {
    const [billingData, setBillingData] = useState(null);
    const [paymentMethods, setPaymentMethods] = useState([]);
    const [invoices, setInvoices] = useState([]);
    
    useEffect(() => {
        fetchBillingData();
        fetchPaymentMethods();
        fetchInvoices();
    }, []);
    
    const fetchBillingData = async () => {
        // Fetch current subscription and billing info
    };
    
    const fetchPaymentMethods = async () => {
        // Fetch saved payment methods
    };
    
    const fetchInvoices = async () => {
        // Fetch invoice history
    };
    
    return (
        <div className="billing-management">
            <h2>Billing & Subscription</h2>
            
            <div className="current-plan">
                <h3>Current Plan</h3>
                <div className="plan-details">
                    <span className="plan-name">{billingData?.planName}</span>
                    <span className="plan-price">${billingData?.monthlyPrice}/month</span>
                </div>
                <button className="btn-primary">Upgrade Plan</button>
            </div>
            
            <div className="payment-methods">
                <h3>Payment Methods</h3>
                {paymentMethods.map(method => (
                    <div key={method.id} className="payment-method">
                        <span>**** **** **** {method.last4}</span>
                        <span>{method.brand}</span>
                        <button className="btn-danger">Remove</button>
                    </div>
                ))}
                <button className="btn-secondary">Add Payment Method</button>
            </div>
            
            <div className="invoice-history">
                <h3>Invoice History</h3>
                <table>
                    <thead>
                        <tr>
                            <th>Date</th>
                            <th>Amount</th>
                            <th>Status</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {invoices.map(invoice => (
                            <tr key={invoice.id}>
                                <td>{invoice.date}</td>
                                <td>${invoice.amount}</td>
                                <td>{invoice.status}</td>
                                <td>
                                    <a href={invoice.downloadUrl}>Download</a>
                                </td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            </div>
        </div>
    );
};

export default BillingManagement;
```

## Development Workflow

### Local Development Setup

#### 1. Development Scripts
```json
// package.json scripts
{
  "scripts": {
    "dev": "webpack serve --mode development --hot",
    "build": "webpack --mode production",
    "test": "jest",
    "lint": "eslint src/",
    "serve": "serve -s build -l 3000",
    "deploy:staging": "npm run build && deploy-to-staging.sh",
    "deploy:production": "npm run build && deploy-to-production.sh"
  }
}
```

#### 2. Environment Management
```javascript
// src/config/environment.js
const environments = {
  development: {
    API_BASE_URL: 'https://localhost:5001',
    PORTAL_URL: 'http://localhost:3000',
    STRIPE_PUBLISHABLE_KEY: 'pk_test_...',
    DEBUG: true
  },
  staging: {
    API_BASE_URL: 'https://api-staging.method.com',
    PORTAL_URL: 'https://portal-staging.method.com',
    STRIPE_PUBLISHABLE_KEY: 'pk_test_...',
    DEBUG: false
  },
  production: {
    API_BASE_URL: 'https://api.method.com',
    PORTAL_URL: 'https://portal.method.com',
    STRIPE_PUBLISHABLE_KEY: 'pk_live_...',
    DEBUG: false
  }
};

const currentEnv = process.env.NODE_ENV || 'development';
export const config = environments[currentEnv];
```

### Testing Strategy

#### 1. Component Testing
```javascript
// src/components/__tests__/AccountOverview.test.js
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import AccountOverview from '../dashboard/AccountOverview';

// Mock fetch
global.fetch = jest.fn();

describe('AccountOverview', () => {
    beforeEach(() => {
        fetch.mockClear();
    });
    
    test('displays loading state initially', () => {
        render(<AccountOverview />);
        expect(screen.getByText('Loading...')).toBeInTheDocument();
    });
    
    test('displays account data when loaded', async () => {
        const mockData = {
            activeUsers: 25,
            appsDeployed: 5,
            storageUsed: 2.5,
            apiCallsThisMonth: 1250,
            recentActivity: [
                { timestamp: '2023-12-01', description: 'User login' }
            ]
        };
        
        fetch.mockResolvedValueOnce({
            ok: true,
            json: async () => mockData
        });
        
        render(<AccountOverview />);
        
        await waitFor(() => {
            expect(screen.getByText('25')).toBeInTheDocument();
            expect(screen.getByText('5')).toBeInTheDocument();
            expect(screen.getByText('2.5 GB')).toBeInTheDocument();
        });
    });
});
```

#### 2. Integration Testing
```javascript
// src/tests/integration/signup-flow.test.js
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import SignupPage from '../pages/signup';

describe('Signup Flow Integration', () => {
    test('complete signup process', async () => {
        // Mock successful API responses
        fetch
            .mockResolvedValueOnce({ ok: true }) // Email availability check
            .mockResolvedValueOnce({ ok: true, json: async () => ({ userId: '123' }) }); // Signup
        
        render(
            <BrowserRouter>
                <SignupPage />
            </BrowserRouter>
        );
        
        // Fill out form
        fireEvent.change(screen.getByLabelText('First Name'), {
            target: { value: 'John' }
        });
        fireEvent.change(screen.getByLabelText('Email'), {
            target: { value: 'john@example.com' }
        });
        fireEvent.change(screen.getByLabelText('Password'), {
            target: { value: 'SecurePass123!' }
        });
        
        // Submit form
        fireEvent.click(screen.getByText('Create Account'));
        
        // Verify redirect to success page
        await waitFor(() => {
            expect(window.location.href).toContain('/signup/success');
        });
    });
});
```

## Deployment Configuration

### Build and Deploy Process

#### 1. Production Build
```powershell
# Build portal for production
function Build-MethodPortal {
    Set-Location "C:\MethodDev\Portal"
    
    # Install dependencies
    npm ci
    
    # Run tests
    npm test -- --watchAll=false
    
    # Build production bundle
    $env:NODE_ENV = "production"
    npm run build
    
    # Copy to IIS directory
    robocopy "build" "C:\inetpub\wwwroot\portal" /MIR
    
    Write-Host "Portal build completed successfully!" -ForegroundColor Green
}
```

#### 2. IIS Deployment
```powershell
# Deploy portal to IIS
function Deploy-MethodPortal {
    param(
        [string]$Environment = "local"
    )
    
    $siteName = "Method-Portal"
    $appPoolName = "Method-Portal"
    
    # Stop app pool
    Stop-WebAppPool -Name $appPoolName
    
    # Update application files
    $sourcePath = "C:\MethodDev\Portal\build"
    $destPath = "C:\inetpub\wwwroot\portal"
    
    robocopy $sourcePath $destPath /MIR /XD node_modules
    
    # Update configuration for environment
    $configPath = "$destPath\config\$Environment.json"
    if (Test-Path $configPath) {
        Copy-Item $configPath "$destPath\config\config.json" -Force
    }
    
    # Start app pool
    Start-WebAppPool -Name $appPoolName
    
    Write-Host "Portal deployed successfully!" -ForegroundColor Green
}
```

## Monitoring and Analytics

### Performance Monitoring
```javascript
// src/utils/analytics.js
class PortalAnalytics {
    constructor() {
        this.sessionId = this.generateSessionId();
        this.startTime = Date.now();
    }
    
    // Track page views
    trackPageView(pageName) {
        this.sendEvent('page_view', {
            page: pageName,
            timestamp: Date.now(),
            sessionId: this.sessionId
        });
    }
    
    // Track user actions
    trackAction(action, properties = {}) {
        this.sendEvent('user_action', {
            action,
            properties,
            timestamp: Date.now(),
            sessionId: this.sessionId
        });
    }
    
    // Track performance metrics
    trackPerformance(metric, value) {
        this.sendEvent('performance', {
            metric,
            value,
            timestamp: Date.now(),
            userAgent: navigator.userAgent
        });
    }
    
    sendEvent(eventType, data) {
        fetch('/api/analytics/events', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ eventType, data })
        }).catch(error => {
            console.warn('Analytics event failed:', error);
        });
    }
    
    generateSessionId() {
        return Math.random().toString(36).substr(2, 9);
    }
}

export const analytics = new PortalAnalytics();
```

**Next**: [Audit Trail Projects](./audit-trail.md)
