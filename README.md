# WebSec Header & Cookie Auditor

A Chrome extension for quick, repeatable web application security checks during manual testing. It inspects the active tab and reports what is safe and what is not, with a plain explanation of why each finding matters and what to fix.

## What it checks

- Security response headers: HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, COOP, COEP, CORS configuration, Server and X-Powered-By disclosure
- Cookies: Secure, HttpOnly, and SameSite flags per cookie, domain scope, session vs persistent lifetime, and the raw value, framed as a hijacking risk assessment (network interception, XSS theft, cross site riding)
- Page level DOM checks: mixed content, forms posting to plain HTTP, password fields in GET forms, missing Subresource Integrity on cross origin scripts and styles, sensitive looking keys in localStorage and sessionStorage, basic secret pattern scanning in page source, third party hosts loaded, open redirect style URL parameters
- Reconnaissance: robots.txt and security.txt presence
- TLS and certificate info for the current connection: negotiated protocol version, cipher suite, certificate subject, issuer, validity window, self signed detection, Subject Alternative Names, and Certificate Transparency compliance

Every fail or warn item includes a "why this is a finding" explanation and a recommended remediation, written so they can be dropped almost as is into a pentest report.

## Scope note on the TLS check

The TLS and certificate check reads what the browser actually negotiated for the current page load, using Chrome's DevTools Protocol. It is not a replacement for testssl.sh or sslyze. It cannot enumerate every protocol version and cipher suite a server would accept, and it cannot probe for implementation bugs like Heartbleed, POODLE, or ROBOT. Those require raw TLS handshakes built outside the browser's own TLS stack, which is not something a browser extension has access to. For that level of testing, run testssl.sh or sslyze against the host directly.

## Installing

1. Download or clone this repository
2. Open `chrome://extensions` in Chrome
3. Enable Developer mode
4. Click "Load unpacked" and select the project folder

## Usage

1. Navigate to the page you want to test
2. Open the extension popup
3. Click Scan

Scan runs the TLS and certificate check first, which reloads the active tab and briefly shows Chrome's "being debugged" banner while the DevTools Protocol reads the negotiated connection details. It then runs the header, cookie, page level, and recon checks against that same freshly loaded page. Nothing runs automatically just from opening the popup, only clicking Scan starts it.

### Filtering results

- Click FAIL, WARN, PASS, or INFO at the top to filter by severity
- Click a category chip (Headers, Cookies, Page, Recon, TLS) to filter by subtopic
- When cookies are found, a chip appears for each individual cookie name so you can jump straight to one, for example `__mpx` or `MpSslSecurity`
- Use the search box to match against any finding title, detail text, or category
- All active filters are shown as removable chips, with a "Clear all" option

### Exporting

- Export JSON exports the full result set for the current tab, findings and raw cookie values included
- Scan All Tabs loops every open http and https tab, runs the header, cookie, and page level checks on each, and exports one combined JSON file. The TLS check is single tab only and is not run in bulk, since it reloads the tab being tested

## Permissions

- `activeTab`, `tabs`: identify and act on the tab being tested
- `cookies`: read cookies and their security attributes for the current origin
- `webRequest`: capture response headers for the main document request
- `scripting`: run the page level DOM checks in the context of the tested page
- `storage`: cache captured headers per tab across service worker restarts
- `debugger`: attach Chrome's DevTools Protocol to read the negotiated TLS protocol, cipher, and certificate for the TLS check. This is what triggers the "being debugged" banner and the tab reload
- `host_permissions: <all_urls>`: needed for the checks above to run on whatever site you are testing, and for the robots.txt and security.txt recon fetches

## Data handling

Cookie values and other findings can include live session material. Anything you export from this tool or capture in a screenshot should be handled the same way you would handle a captured session token or credential. Clear exported JSON files out of report drafts once you are done with them.

## Limitations

- Page level checks reflect the DOM at the moment the popup runs them. Content loaded later by a single page application needs a re-scan to be picked up
- The secret pattern scan in page source is regex based and best effort, treat matches as leads to verify manually, not confirmed findings
- The TLS and certificate check only reports what was negotiated for one connection, not the full set of protocols and ciphers the server supports
- Scan All Tabs does not run the TLS check, to avoid reloading every open tab in the browser