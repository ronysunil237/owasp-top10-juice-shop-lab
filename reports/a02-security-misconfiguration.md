# A02:2025 Security Misconfiguration

## Summary
Multiple misconfigurations were found: an unauthenticated admin endpoint,
directory listing enabled on a sensitive folder, and verbose error
messages that disclose the backend framework and version.

## Finding 1: Unauthenticated admin configuration endpoint
`GET /rest/admin/application-configuration` returns the full application
configuration with no Authorization header required, including internal
settings (chatbot model, Google OAuth client ID, CTF challenge settings).

Evidence: evidence/a02-admin-config.json

Impact: exposes internal configuration to any unauthenticated visitor,
which can help an attacker map the application's internals and find
further attack surface.

## Finding 2: Directory listing enabled on /ftp/
`GET /ftp/` returns a full directory listing instead of a 403/404. Files
exposed include a KeePass password database (incident-support.kdbx),
backup files (package.json.bak, package-lock.json.bak,
coupons_2013.md.bak), and other internal-looking files
(suspicious_errors.yml, announcement_encrypted.md).

Evidence: screenshots/a02-ftp-listing.png, evidence/a02-ftp-listing.txt

Impact: an attacker can download these files directly and attempt to
crack the password database or mine the backups for old secrets or
credentials.

## Finding 3: Verbose error message discloses framework version
`GET /rest/basket/abc` (invalid ID, no auth) returns an error page
stating "OWASP Juice Shop (Express ^4.22.1)" directly in the response.

Evidence: evidence/a02-error-version-disclosure.txt

Impact: reveals the exact backend framework and version, letting an
attacker search for known vulnerabilities specific to that version.

## Root cause
- The admin endpoint has no authentication middleware applied.
- The web server is configured to serve directory listings for /ftp/
  instead of returning 403 Forbidden.
- Default error handling is left in a verbose mode, printing internal
  details instead of a generic error message.

## Recommended fix
- Require authentication and admin-role authorization on
  /rest/admin/* endpoints.
- Disable directory listing on the web server / static file
  middleware, or remove the /ftp/ folder from the public web root.
- Use a generic error handler in production that returns a plain
  message with no framework name, version, or stack trace.

## OWASP mapping
A02:2025 - Security Misconfiguration
