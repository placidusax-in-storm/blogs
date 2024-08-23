When the server responds with HTML, it sets the Content-Security-Policy (CSP) HTTP header.

The browser then respects this policy when loading scripts, CSS, or other resources.

For example, 'script-src 'self'' stops any script from being loaded except those from the current domain.