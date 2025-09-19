# Certificates

This section covers SSL certificate setup for HTTPS development.

## Sections

1. [Download Store-Bought Certificate](./download-certificate.md)
2. [Import Certificates](./import-certificates.md)
3. [Add Certificates to IIS](./iis-certificates.md)
4. [OPTIONAL:BrowserStack Setup](./browserstack.md)
5. [OPTIONAL:Generate Custom Certificates](./custom-certificates.md)

SSL certificates are required for local HTTPS development and testing.

## Overview

Method applications require HTTPS for proper functionality, even in local development. This section covers:

- **Commercial Certificates** - Store-bought certificates for production-like testing
- **Certificate Import** - Proper installation into Windows certificate store
- **IIS Integration** - Configuring web server bindings with SSL
- **Cross-Browser Testing** - BrowserStack setup for testing across different browsers
- **Custom Certificate Generation** - Creating self-signed certificates when needed

⏱️ **Estimated Time:** 30 minutes

📋 **Prerequisites:**
- Administrator access for certificate management
- IIS installed and configured
- MMC (Microsoft Management Console) access

🔐 **Security Note:** Certificates are valid for one year and must be renewed periodically

⚠️ **Important:** All Method sites require HTTPS bindings with proper SSL certificates
