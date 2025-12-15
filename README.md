# Application Detection and Response Runbooks

This repository contains detailed runbooks for responding to various security incidents detected by Contrast Security. These runbooks provide step-by-step guidance for security teams to effectively triage and respond to different types of security events.

## Available Runbooks

### Injection Attacks
- [Command Injection](_runbooks/command-injection.md) - Handling command injection attacks attempting to execute arbitrary system commands
- [JNDI Injection](_runbooks/jndi-injection.md) - Responding to JNDI injection attempts targeting Java applications
- [SQL Injection](_runbooks/sql-injection.md) - Managing SQL injection attacks against database systems
- [Expression Language Injection](_runbooks/expression-language-injection.md) - Addressing expression language injection vulnerabilities
- [Code Injection](_runbooks/code-injection.md) - Handling code injection attacks that execute arbitrary code

### Access Control & Traversal
- [Path Traversal](_runbooks/path-traversal.md) - Handling attempts to access files outside intended directories
- [HTTP Method Tampering](_runbooks/http-method-tampering.md) - Managing unauthorized HTTP method manipulation

### Data & Parsing Vulnerabilities  
- [Cross-Site Scripting (XSS)](_runbooks/cross-site-scripting.md) - Responding to XSS attacks injecting malicious scripts
- [XML External Entity Injection](_runbooks/xml-external-entity-injection.md) - Handling XXE attacks against XML parsers
- [Untrusted Deserialization](_runbooks/untrusted-deserialization.md) - Managing deserialization of untrusted data

## Using the Runbooks

1. Identify the type of security event/alert
2. Navigate to the corresponding runbook
3. Follow the decision tree to classify the event
4. Execute the recommended response procedures
5. Document actions taken and complete post-incident activities

## Contributing

See our [Contribution Guidelines](CONTRIBUTING.md) for information on how to contribute to these runbooks.
