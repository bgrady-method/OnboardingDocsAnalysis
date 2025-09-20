# Local Development

## ⚠️ Enhanced Beyond Original - Under Construction

**Specific Enhancements Beyond Original Document:**

### ORIGINAL CONTENT (Sections 13.1-13.4):
- Important Prerequisites (Section 13.1) ✅
- Critical Projects Health Check & Rebuild (Section 13.2) ✅  
- Create Your Local Account (Section 13.3) ✅
- SMS Notifications Setup (Section 13.4) ✅

### MASSIVE ENHANCEMENTS (Beyond Original):
- **Enterprise Account Creation** (`create-account.md`) - Complete PowerShell automation with account validation, testing, daily workflow functions, and automated troubleshooting
- **Comprehensive SMS Alert System** (`sms-notifications.md`) - Build integration, custom scripts, load testing, daily summaries, and alert management framework
- **Advanced Prerequisites Verification** (`prerequisites.md`) - PowerShell verification commands, comprehensive troubleshooting, and host file content from Section 18
- **Enhanced Project Health** (`critical-projects.md`) - All 15 services with health check URLs, automated troubleshooting, and comprehensive validation

**Original Document Coverage:** Basic manual steps only. Enhanced version transforms into sophisticated automation framework with enterprise-level account management, comprehensive SMS alerting, and advanced validation systems.

This section covers running Method applications in your local development environment.

## Sections

1. [Important Prerequisites](./prerequisites.md)
2. [Critical Projects Health Check & Rebuild](./critical-projects.md)
3. [Create Your Local Account](./create-account.md)
4. [SMS Notifications Setup](./sms-notifications.md)

This is where you'll actually start running Method applications and create test accounts for development.

## Overview

Local development setup involves getting Method's microservices architecture running on your machine:

- **Prerequisites Verification** - Ensuring all dependencies are running
- **Core Services** - Building and validating critical microservices
- **Account Creation** - Setting up local test accounts for development
- **Additional Services** - Optional services like SMS notifications

⏱️ **Estimated Time:** 2-3 hours

📋 **Prerequisites:**
- All previous setup sections completed successfully
- VPN connection active
- All databases restored and accessible
- IIS sites and app pools running
- Docker services running (MongoDB, Redis, Elasticsearch, RabbitMQ)

⚠️ **Critical:** This section requires all previous setup to be completed successfully

🔧 **Health Check Required:**
- SQL Server and Agent running
- MongoDB, Redis, Elasticsearch, RabbitMQ accessible
- All IIS application pools started
- Method microservices building without errors

## Architecture Overview

Method uses a microservices architecture with these critical components:

- **Gateway API** - Main API gateway and routing
- **Runtime Core** - Business logic engine
- **Authentication Services** - Identity and access management  
- **Account Management** - User account lifecycle
- **Method UI** - Front-end application platform

Each service must be built, configured, and health-checked before creating local accounts.
