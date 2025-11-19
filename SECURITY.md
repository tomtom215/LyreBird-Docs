# Security Policy

## Overview

The LyreBirdAudio Documentation project takes security seriously. While this repository contains documentation rather than executable code, we recognize that documentation security matters for:

- **Accuracy**: Preventing malicious documentation that could mislead users
- **Integrity**: Ensuring documentation reflects secure practices
- **Supply Chain**: Protecting the documentation build and deployment pipeline

## Supported Versions

We provide security updates for the following versions of the documentation:

| Version | Supported          | Status                  |
| ------- | ------------------ | ----------------------- |
| 1.x.x   | :white_check_mark: | Current stable release  |
| 0.9.x   | :x:                | No longer supported     |
| < 0.9   | :x:                | No longer supported     |

**Note**: Documentation is continuously deployed from the `main` branch. We recommend always using the latest version available at https://lyrebird-docs.pages.dev

## Security Considerations for Documentation

### 1. Command Safety

All commands and code examples in this documentation are reviewed for:

- **No destructive operations** without clear warnings
- **Proper permissions** (sudo/root only when necessary)
- **Input validation** guidance in examples
- **No hardcoded credentials** or secrets

### 2. External Links

We monitor external links for:

- **HTTPS-only links** where possible
- **Reputable sources** (official documentation, trusted repositories)
- **No link injection** vulnerabilities

### 3. Build Pipeline Security

Our documentation build process uses:

- **Dependency pinning** (exact versions in requirements.txt)
- **Regular dependency updates** for security patches
- **Cloudflare Pages** for secure hosting
- **HTTPS-only** delivery

## Reporting a Vulnerability

### When to Report

Please report security issues if you discover:

1. **Malicious Commands**: Documentation suggesting dangerous or destructive commands without appropriate warnings
2. **Credential Exposure**: Hardcoded passwords, API keys, or secrets in examples
3. **Misleading Security Guidance**: Documentation that suggests insecure practices
4. **Link Hijacking**: Compromised or malicious external links
5. **Dependency Vulnerabilities**: Security issues in Python packages (mkdocs, mkdocs-material, etc.)
6. **Build Pipeline Issues**: Vulnerabilities in the documentation generation or deployment process

### How to Report

**Please DO NOT report security vulnerabilities through public GitHub issues.**

Instead, report security vulnerabilities via:

1. **Email**: Send to tomtom215 [via GitHub email]
   - Subject: `[SECURITY] LyreBird-Docs: Brief Description`
   - Include: Detailed description, steps to reproduce, potential impact

2. **GitHub Security Advisory**: Use the [Security Advisory](https://github.com/tomtom215/LyreBird-Docs/security/advisories/new) feature
   - Private disclosure to maintainers
   - Allows collaborative remediation before public disclosure

### What to Include in Your Report

A good security report includes:

- **Description**: Clear explanation of the vulnerability
- **Location**: Specific file(s) and line number(s) affected
- **Impact**: Potential security impact and severity
- **Reproduction**: Steps to demonstrate the issue
- **Recommendation**: Suggested fix (if you have one)
- **References**: Links to relevant security advisories or documentation

**Example Report:**

```
Subject: [SECURITY] LyreBird-Docs: Unsafe command in installation guide

Location: docs/getting-started/installation.md, Line 47

Description:
The installation guide suggests running a curl | bash command without
verifying the downloaded script's integrity.

Current command:
curl -fsSL https://example.com/install.sh | bash

Impact:
Users could execute malicious code if the download is intercepted or
the source is compromised (MITM attack, DNS hijacking).

Recommendation:
Add checksum verification or suggest downloading the script first,
inspecting it, then executing:

curl -fsSL https://example.com/install.sh -o install.sh
# Verify checksum
sha256sum install.sh
# Inspect the script
less install.sh
# Execute after verification
bash install.sh

Severity: Medium
```

## Response Process

### Timeline

1. **Acknowledgment**: Within 48 hours
   - Confirm receipt of your report
   - Assign a tracking identifier

2. **Initial Assessment**: Within 7 days
   - Evaluate severity and impact
   - Determine affected versions
   - Identify required changes

3. **Remediation**: Depends on severity
   - **Critical**: Immediate (24-48 hours)
   - **High**: 7 days
   - **Medium**: 30 days
   - **Low**: 90 days

4. **Disclosure**: After fix is deployed
   - Coordinated disclosure with reporter
   - Public security advisory published
   - Credit to reporter (if desired)

### Severity Levels

We use the following severity classifications:

| Level    | Impact | Examples |
|----------|--------|----------|
| **Critical** | Immediate risk of harm to users | Commands that delete data without warning, exposed production credentials |
| **High**     | Significant security risk | Insecure configuration recommendations, vulnerable dependencies |
| **Medium**   | Moderate security concern | Missing security warnings, outdated security practices |
| **Low**      | Minor security improvement | Enhanced security documentation, additional warnings |

## Security Best Practices for Contributors

When contributing to LyreBirdAudio documentation:

### Command Examples

- ✅ **DO**: Provide safe, tested commands
- ✅ **DO**: Add warnings for destructive operations
- ✅ **DO**: Use sudo only when necessary
- ❌ **DON'T**: Include rm -rf without clear context and warnings
- ❌ **DON'T**: Suggest disabling security features without explanation

### Code Examples

- ✅ **DO**: Show input validation in examples
- ✅ **DO**: Use placeholders for credentials (e.g., YOUR_API_KEY)
- ✅ **DO**: Demonstrate secure coding practices
- ❌ **DON'T**: Include actual API keys or passwords
- ❌ **DON'T**: Show SQL queries vulnerable to injection

### Configuration Examples

- ✅ **DO**: Recommend secure defaults
- ✅ **DO**: Explain security implications of settings
- ✅ **DO**: Link to official security documentation
- ❌ **DON'T**: Suggest turning off authentication for convenience
- ❌ **DON'T**: Recommend weak encryption or hashing

### External Links

- ✅ **DO**: Use HTTPS links whenever possible
- ✅ **DO**: Link to official, authoritative sources
- ✅ **DO**: Verify links before adding them
- ❌ **DON'T**: Link to untrusted or suspicious sites
- ❌ **DON'T**: Use URL shorteners that obscure destination

## Dependency Security

### Current Dependencies

We regularly update dependencies to address security vulnerabilities:

```text
mkdocs==1.6.1
mkdocs-material==9.7.0
pymdown-extensions==10.17.1
Pygments==2.19.2
Jinja2==3.1.6
PyYAML==6.0.3
requests==2.32.5
```

### Security Updates

- **Automated Scanning**: Dependencies monitored for known vulnerabilities
- **Regular Updates**: Dependencies updated quarterly (minimum)
- **Security Patches**: Applied immediately for critical vulnerabilities
- **Version Pinning**: Exact versions specified in requirements.txt

### Checking for Vulnerabilities

Contributors can check dependencies for known vulnerabilities:

```bash
# Install pip-audit
pip install pip-audit

# Scan dependencies
pip-audit -r requirements.txt
```

## Deployment Security

### Cloudflare Pages

Our documentation is hosted on Cloudflare Pages with:

- **HTTPS Enforcement**: All traffic served over HTTPS
- **DDoS Protection**: Built-in Cloudflare DDoS mitigation
- **CDN**: Content delivered via Cloudflare's global CDN
- **Access Control**: Build and deployment restricted to authorized users

### Build Process

- **Source Verification**: All builds from verified Git commits
- **Isolated Environment**: Builds run in isolated containers
- **No Secrets in Build**: No credentials required for documentation build
- **Automated Deployment**: Deployed only from main branch

## Security Disclosure Policy

### Public Disclosure

After a security issue is fixed:

1. **Security Advisory**: Published on GitHub
2. **Changelog Update**: Security fix noted in CHANGELOG.md
3. **Credit**: Reporter credited (unless anonymity requested)
4. **Details**: Technical details disclosed after 90 days (for critical issues)

### Coordinated Disclosure

We follow responsible disclosure practices:

- **Advance Notice**: Reporters notified before public disclosure
- **Embargo Period**: Minimum 90 days for critical vulnerabilities
- **Coordination**: Work with reporters on disclosure timing
- **CVE Assignment**: Request CVE for significant vulnerabilities (if applicable)

## Contact

- **Security Email**: Via GitHub (tomtom215)
- **Security Advisories**: https://github.com/tomtom215/LyreBird-Docs/security/advisories
- **General Issues**: https://github.com/tomtom215/LyreBird-Docs/issues (non-security only)

## Additional Resources

- [LyreBirdAudio Main Repository](https://github.com/tomtom215/LyreBirdAudio)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)

---

**Last Updated**: 2025-11-19

Thank you for helping keep LyreBirdAudio documentation safe and secure!
