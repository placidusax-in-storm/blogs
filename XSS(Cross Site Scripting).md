XSS (Cross-Site Scripting) is a type of attack where malicious code, typically JavaScript or CSS, is injected into a website.

How it happens:

1. Injecting HTML directly without sanitization

How to prevent it:

1. Input Sanitization
2. CSP (Content Security Policy)

Facts:

- Modern browsers sanitize user input URLs, including hash and search parameters
- Search parameters are encoded when set by URLSearchParams.set
- When using `location.href = 'url'`, the URL is also properly encoded