# Host File List

## Host File Entries

Add the following entries to your hosts file located at: `C:\Windows\System32\drivers\etc\hosts`

```
################################
127.0.0.1 methodlocaldb
127.0.0.1 methodlocal
127.0.0.1 methodlocal.com
127.0.0.1 www.methodlocal.com
127.0.0.1 gateway.methodlocal.com
127.0.0.1 gateway.methodlocal.int
127.0.0.1 subscriber.methodlocal.com
127.0.0.1 runtime.methodlocal.com
127.0.0.1 microservices.methodlocal.int
127.0.0.1 auth.methodlocal.com
127.0.0.1 emailgadget.methodlocal.com
127.0.0.1 oauth.methodlocal.com
127.0.0.1 identityserver.methodlocal.com
127.0.0.1 eda.methodlocal.int
127.0.0.1 templatedevv2.methodlocal.com
127.0.0.1 alocetsystem.methodlocal.com
127.0.0.1 miurl.methodlocal.com
127.0.0.1 api.methodlocal.com
127.0.0.1 api.methodlocal.int
127.0.0.1 client.methodlocal.com
127.0.0.1 proxy.methodlocal.com
127.0.0.1 signup.methodlocal.com
127.0.0.1 signin.methodlocal.com
127.0.0.1 services.methodlocal.com
127.0.0.1 migration.methodlocal.com
127.0.0.1 sync.methodlocal.com
127.0.0.1 notify.methodlocal.com
127.0.0.1 intercom.methodlocal.com
127.0.0.1 billing.methodlocal.com
127.0.0.1 outlookgadget.methodlocal.com
127.0.0.1 appdirect.methodlocal.com
127.0.0.1 syncservices.methodlocal.com
127.0.0.1 designer.methodlocal.com
127.0.0.1 notifications.methodlocal.com
127.0.0.1 px.methodlocal.com
127.0.0.1 signin.methodportallocal.com
127.0.0.1 dataminer.methodlocal.com
################################
```

## How to Edit Hosts File

1. **Open Notepad as Administrator**
   - Press Windows key, type "notepad"
   - Right-click on Notepad and select "Run as administrator"

2. **Open Hosts File**
   - In Notepad: File → Open
   - Navigate to: `C:\Windows\System32\drivers\etc\`
   - Change file type filter to "All Files (*.*)"
   - Select "hosts" file

3. **Add Entries**
   - Scroll to the bottom of the file
   - Copy and paste the entries above
   - Save the file (Ctrl+S)

## Purpose

These host entries redirect Method's local development URLs to your local machine, allowing you to:

- Test Method applications locally
- Access local development services
- Bypass external DNS resolution for development URLs

⚠️ **Important:** These entries are required for local development to function properly.

**Related:** [DNS Setup (Optional)](./dns-setup.md)
