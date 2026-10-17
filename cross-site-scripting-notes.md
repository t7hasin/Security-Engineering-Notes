### ⚡ Attack Style: Cross-Site Scripting (XSS) Fundamentals

**The Goal:** Force a vulnerable web application to execute arbitrary JavaScript inside a victim's browser, bypassing the Same Origin Policy to hijack user sessions, steal data, or seize total administrative control.

---

#### 🧩 The Concept: The Broken Wall (Same Origin Policy)

* **What is XSS?** Cross-site scripting occurs when an application injects malicious scripts into content sent to trusted users.
* **The Security Violation:** Browsers rely on the **Same Origin Policy (SOP)** to isolate different websites from each other, ensuring a malicious site can't peek at your bank session. XSS completely shatters this security boundary by turning a *trusted* website into a delivery vehicle for malicious code.
* **The Power:** Once executing inside a victim's browser, the attacker can masquerade as that user, read sensitive data, capture credentials, or deploy trojan functionality.

---

#### 🧪 The Proof of Concept (PoC): The `alert()` vs. `print()` Quirk

* **The Classic Test:** To confirm an XSS vulnerability, security professionals typically inject a simple, harmless payload designed to pop up a dialog box in the browser:
```javascript
alert(1)

```


* **The Modern Chrome Quirk:** From Chrome version 92 onward, cross-origin iframes are strictly blocked from calling `alert()`. Because many advanced labs and real-world exploits rely on iframes, hackers now use an alternative PoC payload:
```javascript
print()

```


*(Both functions are short, harmless, and impossible to miss when successfully triggered!)*

---

#### 🗂️ The Big Three Varieties of XSS

XSS vulnerabilities are split into three major architectural classes depending on where the malicious script originates:

1. **Reflected XSS (Non-Persistent):**
* *The Source:* The malicious script comes directly from the current HTTP request (e.g., a clickable link or a search parameter).
* *The Action:* The server echoes the input right back in the response without proper sanitization.


2. **Stored XSS (Persistent):**
* *The Source:* The malicious script comes from the website's database (e.g., a forum comment, user profile bio, or product review).
* *The Action:* The script is permanently stored on the server, waiting to infect *every single user* who loads that page later.


3. **DOM-Based XSS:**
* *The Source:* The vulnerability exists entirely in **client-side JavaScript** (the Document Object Model) rather than server-side code.
* *The Action:* Unsafe client-side scripts dynamically take data from an insecure source (like the URL) and write it straight to a dangerous sink (like `innerHTML`).



---

#### 🎯 Impact: What Can XSS Actually Do?

The real-world damage of an XSS attack depends heavily on the app's context and the victim's privileges:

* **Low Impact:** A public brochureware site where all users are anonymous and data is entirely public.
* **Serious Impact:** Applications handling sensitive information like banking transactions, private emails, or healthcare records.
* **Critical Impact:** If the victim user has elevated administrative privileges, an XSS exploit allows the attacker to take **total control** of the entire web application and compromise every user on the platform!            

### 🪞 Attack Style: Reflected Cross-Site Scripting (The Echo Trap)

**The Goal:** Trick a victim into clicking a malicious link that forces the web server to instantly bounce an arbitrary script back in its HTTP response, executing it directly inside the victim's browser session.

---

#### 🧩 The Concept: The Bounce-Back

* **What makes it "Reflected"?** Unlike stored XSS, reflected XSS is **non-persistent**. The malicious script doesn't live permanently in a database; it originates entirely from a single HTTP request (usually a crafted URL).
* **The Flaw:** It happens when an application receives user data (like a search query or error message) and immediately echoes that data back out in the response without sanitizing or escaping it.

---

#### 🛠️ The Scenario: From Normal Search to Malicious Trap

* **The Normal Request:** Imagine a website with a search box that passes your term through a URL parameter:
```text
https://insecure-website.com/search?term=gift

```


The application safely echoes your search term right back onto the page:
```html
<p>You searched for: gift</p>

```


* **The Malicious URL:** Because the app blindly trusts and reflects the input, an attacker replaces the search term with a raw JavaScript payload:
```text
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>

```


* **The Reflected Response:** The web server dutifully spits the attacker's script straight back into the HTML response:
```html
<p>You searched for: <script>/* Bad stuff here... */</script></p>

```



---

#### 🎯 Why It Works & The Impact

* **Execution in Context:** When an unsuspecting victim clicks the attacker's custom link, their browser loads the page, encounters the unescaped `<script>` tags, and assumes the code came legitimately from the website.
* **The Compromise:** The script executes immediately **in the context of the victim's active session**.
* Because the browser trusts the domain, the malicious script can steal session cookies, capture keystrokes, or perform unauthorized actions on behalf of the user—all because the server blindly echoed untrusted input!

### ⚡ Attack Style: The Damage of Reflected XSS (What Can Happen?)

**The Goal:** Realize the absolute control an attacker gets once a user clicks a malicious link, and why this attack needs a "delivery truck" to work.

---

#### 🧩 1. The God-Mode Takeover (What the Attacker Can Do)

Once the script runs in the victim's browser, the attacker effectively **becomes** that user. They can:

* **Do Anything:** Click buttons, buy items, or change passwords on behalf of the victim.
* **Steal Data:** Read private emails, messages, or financial records the user can see.
* **Spread the Infection:** Send fake messages or attacks to *other* users, making it look like the victim did it!

---

#### 🚚 2. How the Trap is Delivered (The Bait)

Reflected XSS can't hide quietly inside the website. It needs a **delivery mechanism** to reach the victim:

* **Phishing Links:** Slipping a rigged URL into emails, social media posts, or rogue websites.
* **Targeted or Mass Blast:** The attacker can either hunt a specific high-value target or cast a wide net hoping *anyone* clicks.

---

#### ⚖️ 3. Reflected vs. Stored: Why Reflected is "Less" Dangerous

* **Reflected XSS (The Delivery Truck):** The attacker has to actively trick you into clicking an external link. If you don't click, nothing happens.
* **Stored XSS (The Landmine):** The code lives permanently on the website. Anyone who visits the page gets blown up automatically.
* *The Verdict:* Because Reflected requires a separate delivery step (getting you to click a bad link), it is generally considered **less severe** than Stored XSS—though the damage once clicked is still catastrophic!

### 🗺️ Attack Style: Finding & Testing Reflected XSS (The Hunting Blueprint)

**The Goal:** Systematically hunt down and verify reflected XSS vulnerabilities by firing probes, analyzing exactly where they land, and breaking out of different HTML or JavaScript contexts.

---

#### 🧩 Concept: Why Context is King

* **The Trap Location:** Where your injected data pops back out in the HTTP response determines your entire attack strategy. An input landing raw between HTML tags requires a different payload than one trapped inside a quoted attribute or a JavaScript string.
* **The Filter Hurdle:** If the application validates or sanitizes your input, you’ll need to adapt your payloads to bypass those specific filters.

---

#### 🔍 The 6-Step Hunting Checklist (Manual Testing Workflow)

1. **Step 1: Map Every Entry Point**
* Test every single door where data enters the application: URL query parameters, message bodies, file paths, and even HTTP headers.


2. **Step 2: Fire the Random Probe**
* Send a short, unique 8-character random alphanumeric string (e.g., `z7x9q2w1`) into each entry point.
* *Why?* It’s short enough to survive basic validation, but unique enough that accidental matches won't happen. Use automated tools to instantly flag where your random string bounces back.


3. **Step 3: Pinpoint the Reflection Context**
* Examine *how* your random string is reflected. Is it sitting raw as regular text between HTML tags? Is it locked inside a quoted HTML attribute? Or is it hiding inside a JavaScript string?


4. **Step 4: Launch the Candidate Payload**
* Test a basic candidate script using **Burp Repeater**.
* *Pro Tip:* Leave your random probe string right next to your payload in the request. Set that probe as your search term in Burp Repeater's response view to instantly highlight and locate your payload's exact landing spot.


5. **Step 5: Adapt and Overwrite (Alternative Payloads)**
* If the application strips or blocks your candidate payload, don't give up. Pivot to alternative payloads and context-specific bypasses designed to outsmart the app's filters.


6. **Step 6: Verify in a Real Browser**
* Never rely purely on proxy tools. Copy your winning request or URL into a real browser to see if the script actually executes.
* *The Golden Check:* Run a simple trigger like `alert(document.domain)`—if a popup box appears on screen, you have successfully confirmed the XSS vulnerability!

### 💾 Attack Style: Stored Cross-Site Scripting (The Landmine)

**The Goal:** Plant a malicious script permanently into a website's database (like a comment, forum post, or profile bio) so that it automatically explodes and executes inside the browser of *every single user* who visits the page.

---

#### 🧩 The Concept: The Persistent Trap

* **What makes it "Stored"?** Unlike Reflected XSS (which vanishes immediately if unclicked), Stored XSS (also known as persistent or second-order XSS) is saved directly onto the server.
* **The Flaw:** The application accepts untrusted data from a user, stores it in a database without proper sanitization, and later outputs it blindly to other unsuspecting visitors.

---

#### 🛠️ The Scenario: From Normal Comment to Global Infection

* **The Normal Submission:** Imagine a blog post where users submit comments via a standard POST request:
```http
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net

```


Normally, the website saves this text and displays it safely to future readers:
```html
<p>This post was extremely helpful.</p>

```


* **The Planted Landmine:** An attacker replaces the normal comment text with a raw JavaScript payload and URL-encodes it:
```text
comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E

```


* **The Permanent Output:** The server saves this script into its database. Now, any user who loads the blog post receives the payload straight from the server response:
```html
<p><script>/* Bad stuff here... */</script></p>

```



---

#### 🎯 Why It Works & The Impact

* **Zero Interaction Needed:** The victim doesn't need to click a suspicious phishing link in an email. They simply navigate to a normal, legitimate page (like a blog post or product review).
* **Automatic Execution:** The moment the page renders, the stored script executes instantly in the background of the victim's active session.
* **The Ultimate Danger:** Because the attack is self-contained and automated by the server itself, Stored XSS is infinitely more dangerous than Reflected XSS—a single planted payload can silently compromise every user, moderator, or administrator who visits the page!

### 💾 Attack Style: The Impact & Hunting Guide for Stored XSS (The Persistent Threat)

**The Goal:** Understand why Stored XSS is a self-contained nuclear option, and learn how to systematically map every hidden entry and exit point to plant your payload.

---

#### 🧩 1. The Impact: Why Stored XSS Beats Reflected

* **The God-Mode Compromise:** Just like reflected XSS, once your script executes in a victim's browser, you take full control of their session.
* **The Self-Contained Trap:** Reflected XSS requires a "delivery truck" (phishing links, social media) to trick someone into clicking. Stored XSS is a **landmine**—you plant it once inside the app, and you just sit back and wait.
* **The Login Guarantee:** For reflected XSS, if the victim isn't logged in when they click your link, the attack fails. For stored XSS, the victim *must* be actively logged in and browsing the page to see your comment or profile, **guaranteeing** they are authenticated when the exploit fires!

---

#### 🔍 2. The Manual Hunting Blueprint (Entry & Exit Points)

Testing for Stored XSS is challenging because data can enter through one door and pop out anywhere. Here is how to track it down:

* **Step 1: Map Every Entry Point (Where Data Goes In)**
* Test URLs, message bodies, file paths, request headers, and **hidden out-of-band routes** (like incoming emails, third-party tweets, or news aggregators).


* **Step 2: Map Every Exit Point (Where Data Comes Out)**
* Exit points are *any* HTTP response visible to *any* user. Your input could end up on an obscure admin audit log, a profile page, or a public comment section.


* **Step 3: Link Entries to Exits (Systematic Probing)**
* Because testing every single combination takes forever, work systematically: submit a unique random value into an entry point, monitor the app, and see if it pops up elsewhere. Verify that it's truly **stored** in the database rather than just instantly reflected.


* **Step 4: Test the Context & Payload**
* Once you find a confirmed link between an entry and exit point, analyze the HTML/JavaScript context of the exit point. Test your candidate XSS payloads to break the container and force execution—just like you would with reflected XSS!
