---
sr-due: 2024-09-07
sr-interval: 3
sr-ease: 250
---

#sd

**Purpose**: Cross-Site Request Forgery (CSRF) is a type of security vulnerability where an attacker tricks a victim into performing actions on a web application in which the victim is authenticated, without the victim's knowledge or consent

**How It Works**:

- **Victim Authentication**:  The victim logs into a web application (e.g., their bank) and receives a session cookie.
- **Crafted Malicious Request**:   The attacker creates a malicious web page or email containing a request that targets the web application where the victim is authenticated. The request could be a form submission, image tag, script, or AJAX request that triggers an action on the target web application (e.g., transferring money).
- **Victim Interaction**: The victim visits the malicious website or clicks on the malicious link while still logged into the target web application. The victim’s browser includes the session cookie with the request because the browser automatically sends cookies for the relevant domain.
- **Unauthorized Action**: The target web application processes the request using the victim's session cookie, believing it to be a legitimate request from the victim.


**Prevention Techniques**:

- **CSRF Tokens**: Including unique tokens in forms and validating them server-side.
- **SameSite Cookies**: Setting cookies with the `SameSite` attribute to prevent them from being sent with cross-site requests.
- **Referer/Origin Checking**: Validating the `Referer` or `Origin` header to ensure requests come from trusted sources (can also help with [[CORS]])
- **User Interaction**: Requiring explicit user actions for sensitive operations (e.g., CAPTCHAs, re-entering passwords).