# Web App — Hardening Checklist

## Immediate containment

- [ ] Remove any webshell/backdoor file found (copy aside first for evidence).
- [ ] Identify and close the actual entry point (upload flaw, exposed config, default creds, outdated
      plugin/component) — not just the symptom.
- [ ] Rotate any credentials the application had access to that may have been exposed (DB creds, API
      keys, session secrets).

## Configuration hygiene

- [ ] Remove exposed `.git`, `.env`, `.bak`, `config.php.bak` and similar files from the webroot.
- [ ] Disable directory listing.
- [ ] Turn off verbose error messages / debug mode in production config.
- [ ] Restrict or remove unused admin/management endpoints (e.g. Tomcat Manager, phpMyAdmin) or at
      least change default credentials.

## File upload hardening (if applicable)

- [ ] Validate file extension **and** content-type server-side (not just client-side JS).
- [ ] Store uploaded files outside the webroot, or disable script execution in the upload directory.
- [ ] Re-encode/re-process uploaded images rather than trusting them as-is.

## Verify the service still works

- [ ] Confirm the web application still loads and functions correctly after your changes — the
      guideline explicitly checks that required services keep working.
- [ ] Re-test the specific feature you patched (e.g. the upload form) to confirm it still works for
      legitimate use, not just that the exploit path is closed.

## Documentation

- [ ] Record the vulnerability class (e.g. "unrestricted file upload," "exposed .git directory,"
      "default credentials on admin panel") alongside the specific evidence (request/response, file
      path, timestamp).
