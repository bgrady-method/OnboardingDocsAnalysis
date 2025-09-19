# Critical Projects Health Check & Rebuild

This guide covers building and verifying the health of Method's critical microservices for local development, based on the Method Developer Machine Setup documentation.

## Overview

Method's local development environment relies on several critical microservices that must be built, configured, and running properly. This section ensures all core components are healthy before creating local accounts.

## Prerequisites

Before proceeding, ensure all [prerequisites](./prerequisites.md) have been met:

- All services are running (SQL Server, IIS, Docker)
- VPN connected (if working remotely)
- All previous setup sections completed

## Important Notes

- Make sure you're **connected to the VPN** for builds/nuget restoration
- Health Checks usually should have **No "Fail" keyword** in them
- If you see Errors during build or "Fails" in health check, contact your team
- **Exception:** If "File Storage" dependency failing, you can ignore that
- If you get a Network Request Error such as 400, 500, 503, etc - see **Troubleshooting section**

## Critical Method Projects

### 1. ms-gateway-api

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://api.methodlocal.com/v2/health/check

**Alternative Health Check:** Press the "play" button with "IIS" selected

### 2. Runtime Project

**Configuration:**
1. In *Visual Studio* → Open "**C:\MethodDev\runtime-core\Runtime.stack.sln"**
2. Make sure "*Startup Project*" is set to **Runtime.Core.Api** and "*Play*" profile is "**IIS**"

**Rebuild:** Rebuild **Runtime.Core.Engine**

### 3. ms-authentication-api

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://api.methodlocal.com/v2/authentication/health/check

**Alternative:** Press the "play" button with "IIS" selected

### 4. ms-authentication-api-oauth2

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://auth.methodlocal.com/health/check

**Reference:** [auth.methodlocal.com (identityserver)](https://docs.google.com/document/d/1XwK8jg0V9jWYidg6j_xOb-RZj1h-feAe1a1riAL5VPQ/edit#heading=h.bf98lfn328fq)

**Troubleshoot:** If you are not able to access the service, then probably the IIS is pointing to incorrect path.

- The old path/project is: C:/MethodDev/ms-authentication-oauth2
- The new project is: oauth2
- Download this from github to C:/MethodDev/ and rebuild it: [methodcrm/oauth2 (github.com)](https://github.com/methodcrm/oauth2)
- Go to IIS, go to advanced settings for auth.methodlocal.com, and change the path to C:\MethodDev\oauth2\authentication\API

### 5. ms-identity-api

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://api.methodlocal.com/v2/identity/health/check

### 6. ms-preferences-api

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://api.methodlocal.com/v2/preferences/health/check

### 7. ms-email-api

**Rebuild:** Press the "play" button with "IIS" selected in Visual Studio

**Health Check:** https://api.methodlocal.com/v2/email/health/check

### 8. ms-account-api

**Health Check:** http://microservices.methodlocal.int/account/health/check

#### First-Run and Troubleshooting

An offline template database needs to be generated for new local test accounts to be based on. If you find you're missing important migrations on new test accounts, or you encounter an error "*missing C:\\\\MethodData\\\\MethodIntegrationSQL\\\\TemplateMethodNewOffline.mdf*", perform these steps:

1. **Open Postman**
2. **Use Account Management → v3 → Offlines → internal endpoints**
3. **Call these GET endpoints in order:**
   1. Get Clean Generation State - Internal: http://microservices.methodlocal.int/account/v3/offline/internal/GetCleanGenerationState
   2. Get Unified Template Generated - Internal: http://microservices.methodlocal.int/account/v3/offline/internal/GenerateUnifiedTemplate
   3. SetOfflineUnifiedTemplate - Internal: http://microservices.methodlocal.int/account/v3/offline/internal/SetOfflineUnifiedTemplate
4. **Verify no errors** appear in D:\logs\microservices\api\account and http://microservices.methodlocal.int/account/health/check returns success
5. **TemplateMethodNewOffline_Log.ldf file was missing:** manually copied ldf provided by others into C:\MethodData\MethodIntegrationSQL

### 9. ms-apps-api

**Health Check:** http://microservices.methodlocal.int/apps/health/check

### 10. ms-tables-fields-api

**Health Check:** http://microservices.methodlocal.int/tables-fields/health/check

**Troubleshoot:** Make sure you have run registry: [Export the Registry Key - Salt](../iis-configuration/README.md)

### 11. legacy-authentication-api

**Health Check:** http://services.methodlocal.com/authservice/health/check

### 12. method-signin-ui

**Health Check:** https://signin.methodlocal.com/health/check

### 13. method-signup-ui

**Health Check:** https://signup.methodlocal.com/health/check

### 14. internal-migration-api

**Health Check:** https://migration.methodlocal.com/health/check

**Endpoints:** https://migration.methodlocal.com/help

#### Run migrations (migrateall):

1. **Open Postman**
2. **Use Migrations/All Companies**
3. **URL:** https://migration.methodlocal.com/migration/MigrateAllAccounts?stopOnError=false
4. **Run**
5. **Verify no errors** appear in D:\logs\MigrationService

### 15. Configure and Rebuild MethodUI project

1. **Open Command Prompt in Admin mode**

2. **Go to C:\MethodDev\method-platform-ui\m-one**

3. **Install frontend npm dependencies:** `npm install`

4. **Build the monorep:** `npm run build -ws`

5. **Go to C:\MethodDev\method-platform-ui\MethodUI**

6. **Install frontend npm dependencies:** `npm install`
   - If you face any errors, try: `npm i --legacy-peer-deps`
   - **Note:** Some errors were previously resolved by installing the Windows 10 SDK, however that should *not* be required, please ask for assistance if issues with this step.

7. **npm gyp errors solution:** If you face npm gyp errors with message "You need to install the latest version of Visual Studio" or "Missing any windows SDK":
   - **Solution:** install "Desktop development with C++" workload
   - Also go to individual components and install the latest version of Windows 10 SDK (Windows 11 SDK doesn't seem to work)

8. **Install grunt-cli:** `npm install -g grunt-cli`

9. **Build all legacy, react and sass-foundation files:** `npm run build:dev`

   - **Out of memory errors solution:** If you run into out of memory errors, open **C:\MethodDev\method-platform-ui\methodui\package.json** and update the "build:dev" line under "scripts" to:
   
   `"set NODE_OPTIONS=--max-old-space-size=4096 npm run prettify && npm run lint && npm run build:sass && npx mix",`

   *If the action editor designer or dashboard does not load properly or the layout is out of whack run the above command (npm run build:dev) again and restart your IIS.*

10. **In Visual Studio:** Open "**C:\MethodDev\method-platform-ui\MethodPhoenix.sln**"

11. **Go to MethodUI properties → Web**
    1. Select "Local IIS"
    2. Project URL: `http://methodlocal.com/apps`
    3. Save All files

12. **Rebuild MethodPhoenix**

**Health Check:** You can run this once there's an account to work with

## Troubleshooting Common Issues

### Build/Service Issues

**Missing ssl:** Check SSL certificate installation

**Not built service:** HTTP Error 500.0 - ANCM In-Process Handler Load Failure
- Verify service is built and running
- Check application pool is started

**Incorrect netcore config:** Verify environment variables are set

**Stopped IIS app pool:** HTTP Error 503. The service is unavailable.
- Start the application pool in IIS Manager

**HTTP Error 502.5:** Check service configuration and logs

**RabbitMQ is unhealthy:** If health checks show that rabbitmq is unhealthy (They are not able to access rabbitmq running in docker), try installing rabbitmq directly using: `choco install rabbitmq`

### Network/Connection Issues

**400, 500, 503 errors:** See troubleshooting section and check:
- VPN connection
- Services are running  
- Database connectivity
- Application pool status

## Verification Checklist

Before proceeding to create accounts:

- [ ] All critical services built successfully
- [ ] Health check URLs return success (no "Fail" keyword)
- [ ] VPN connected
- [ ] All application pools running
- [ ] Database services accessible
- [ ] Method UI built and configured

## Next Steps

After successful critical projects health check:

1. **Create Local Account:** [Create Your Local Account](./create-account.md)
2. **Optional Services:** [SMS Notifications Setup](./sms-notifications.md)

## Getting Help

If you encounter issues:

- **Check the logs** in D:\logs\ for specific error messages
- **Verify prerequisites** are all completed
- **Ask team members** for assistance with specific services
- **Reference health check URLs** to verify service status

**Back to:** [Local Development](./README.md)
