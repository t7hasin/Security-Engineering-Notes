How to protect websites from CSRF, kept super simple and point-by-point:

### 1. Use CSRF Tokens (The Best Defense)

* **What it is:** A secret, random ticket that your server gives the user. Whenever the user tries to do something important (like change their password or send money), they must send this ticket back.
* **Why it works:** An attacker's malicious site cannot guess or steal this secret ticket, so any fake request they send gets blocked.
* **The Rules for Tokens:**
* **Unpredictable:** Generated using strong, secure random math.
* **Tied to the User:** Bound directly to that specific user's active session.
* **Strictly Checked:** The server must double-check this token on *every single request* before allowing the action. If it's missing or wrong, reject it!


* **How to Send It:** Put it in a hidden form field (submitted via POST) right near the top of the HTML code so attackers can't easily steal it. Avoid putting tokens in URLs (where they get logged) and **never put CSRF tokens inside cookies**.

### 2. Use `SameSite=Strict` Cookies

* **What it is:** A flag you set on your cookies telling the browser how to handle them.
* **The Rule:** Default to **`Strict`** whenever possible. This blocks your cookies from being sent on *any* cross-site link or request, giving you maximum protection. Only drop down to `Lax` if you absolutely have to.

### 3. Watch Out for Sibling Subdomains (Cross-Origin, Same-Site)

* **What it is:** Reminding you that SameSite cookies won't save you if the attack comes from inside your own family of subdomains.
* **The Fix:**
* Keep risky things (like user-uploaded files or older, unsecured apps) completely isolated on a **totally separate base domain**, not just a subdomain.
* Audit your entire attack surface—if a secondary subdomain has a security hole (like XSS), hackers can use it to bypass SameSite rules and attack your main app.
