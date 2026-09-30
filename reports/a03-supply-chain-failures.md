# A03:2025 Software Supply Chain Failures

## Summary
Running `npm audit` against the Juice Shop dependency tree revealed 43
known vulnerabilities in third-party packages: 7 critical, 19 high, 14
moderate, and 3 low. These are not bugs in the application's own code,
they exist in libraries the application depends on, which is exactly
what this OWASP category covers.

## Method
1. Cloned the Juice Shop source code (read-only, for dependency
   scanning purposes only, not for running a modified copy of the app):
   `git clone --depth 1 https://github.com/juice-shop/juice-shop.git`
2. Built the dependency tree: `npm install --package-lock-only`
3. Ran `npm audit` and saved both the human-readable and JSON output.

## Key findings (critical severity)

- **crypto-js (<=4.1.1)**: uses a PBKDF2 implementation that is far
  weaker than the original 1993 standard, and 1.3 million times weaker
  than the current standard, undermining any encryption relying on it.
- **decompress**: vulnerable to "Zip Slip", a malicious archive can
  write files outside the intended extraction folder during
  decompression.
- **lodash (<=4.17.23)**: multiple Prototype Pollution and a Command
  Injection vulnerability, both can let an attacker manipulate the
  application's internal objects or run unintended commands.
- **marsdb**: a Command Injection vulnerability with no fix currently
  available, meaning the only mitigation is to stop using the package.
- **tar (<=7.5.20)**: multiple path traversal and hardlink issues that
  can let a malicious archive overwrite arbitrary files during
  extraction.

Full details for all 43 findings are in evidence/a03-npm-audit.txt.

## Impact
Vulnerable dependencies are pulled in automatically as part of the
software supply chain, meaning the application inherits risk from code
its own developers never wrote or reviewed. Several of these findings
(Zip Slip, Command Injection, Prototype Pollution) could lead to
remote code execution or file system compromise if the vulnerable
code paths are reachable from user input.

## Root cause
Dependencies were not kept up to date, and some transitive dependencies
(packages pulled in indirectly by other packages, not chosen directly)
introduce vulnerable versions even when the direct dependency itself
looks fine.

## Recommended fix
- Run `npm audit fix` regularly for non-breaking fixes.
- Review and test `npm audit fix --force` changes for packages with
  breaking updates available, prioritising critical/high severity
  first.
- For packages with no fix available (e.g. marsdb), evaluate whether
  the package is still needed or should be replaced.
- Add automated dependency scanning (e.g. npm audit, Dependabot, or
  Snyk) to CI/CD so new vulnerabilities are caught on every build,
  not just when manually checked.

## OWASP mapping
A03:2025 - Software Supply Chain Failures
