#### 💡About These Notes:  These are my personal security engineering study notes, structured and refined with AI assistance. Feel free to use them as a comprehensive reference guide to master Cross-Site Scripting (XSS)!
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

### 🧬 Attack Style: DOM-Based Cross-Site Scripting (The Client-Side Trick)

**The Goal:** Trick a website's own background JavaScript into taking messy data (like a link you clicked) and accidentally running it as code.

---

#### 🧩 Concept: The Source, The Sink, and The Pipe

* **The Source (Where trouble starts):** Any spot where an attacker can slip data into the browser. Most of the time, this is just the **URL** (like the stuff after the `?` mark).
* **The Sink (The danger zone):** A special JavaScript command or HTML updater (like `eval()` or `innerHTML`) that takes text and treats it like real website code.
* **The Pipe (How it flows):** Unlike other attacks where the server gets tricked, **the server never even sees this happening**. The website's own JavaScript running inside your browser takes the bad data from the URL and feeds it straight into the danger zone.

---

#### 🔍 1. How to Test HTML Danger Zones (Visible Inspection)

When the website dumps your URL data straight onto the webpage:

1. **Drop a Test Word:** Put a random word (like `hello123`) into the URL search bar.
2. **Search the Live Page (Not "View Source"):** Open your browser's Developer Tools (`Ctrl+F` or `Cmd+F`) and search for your test word in the **Elements/DOM tab**.
* *Why?* Never right-click and select "View Source"! That only shows the original code sent by the server. You need to see what the live JavaScript is currently doing to the page.


3. **Break Out:** See how your word is wrapped. If it's trapped inside quotation marks, try typing a matching quote to break out of the container and slip your script in.
* *Watch Out:* Modern browsers (like Chrome, Firefox, and Safari) automatically scramble/encode URL symbols, which usually stops these attacks from working easily.



---

#### 🕵️‍♂️ 2. How to Test JavaScript Danger Zones (The Invisible Flow)

Testing commands like `eval()` is trickier because your test word **won't show up on the screen**, meaning you can't just search for it visually.

1. **Search the Code:** Use Developer Tools (`Ctrl+Shift+F` or `Cmd+Option+F`) to scan all the website's JavaScript files for mentions of the URL or input sources.
2. **Use Breakpoints (The Pause Button):** Find where the script reads the URL, and set a "breakpoint" to freeze the code in place while it runs.
3. **Trace the Path:** Step through the code line by line to see where the data travels. Hover your mouse over the variables right before they hit the danger zone to see what they contain, then figure out how to reshape your input to take over!

### 🌊 Attack Style: Sinks & Sources in DOM XSS (The Pipeline)

**The Goal:** Understand how different data pathways (Sources) and destination commands (Sinks) change your hacking strategy, especially when using powerful sinks like `document.write`.

---

#### 🧩 Concept: The Plumbing of DOM XSS

* **The Path Matters:** A website is vulnerable as long as there is an unbroken pipe connecting a **Source** (where you inject data) to a **Sink** (where the data is executed or rendered).
* **Defensive Roadblocks:** Depending on which sink the app uses, or if the developers tried to filter your data along the way, you will need to tweak your payloads to get code execution.

---

#### 🛠️ The `document.write` Sink: The HTML Injector

* **How it works:** The `document.write` sink writes raw data directly into the webpage's document stream while the page is loading.
* **Why it's special:** Unlike some sinks that block script tags, `document.write` happily accepts and executes newly injected `<script>` elements.
* **The Payload:** Because it handles script tags natively, you can use a classic, straightforward payload:
```javascript
document.write('... <script>alert(document.domain)</script> ...');

```


* **The Catch:** If your input lands inside a quoted string or an existing HTML attribute inside that `document.write` statement, you will first need to break out of those quotes and tags before your `<script>` tag can run!

### 🧱 Attack Style: Breaking Out of Context in `document.write` (The Container Trap)

**The Goal:** Master how to escape surrounding HTML tags and quotes so your payload doesn't get trapped inside dead HTML.

---

#### 🧩 Concept: The Locked Box Problem

* **The Trap:** `document.write` rarely drops your input into an empty space. Usually, it wraps your text inside existing HTML code (like a search box, a paragraph tag, or quotation marks).
* **The Result:** If you just inject `<script>alert(1)</script>` blindly, the browser might treat it as normal text or shove it inside an attribute, rendering it harmless.

---

#### 🛠️ How to Escape the Container

* **Close Before You Explode:** You have to manually "break out" of whatever container your input lands in before your JavaScript can run.
* **The Fix:** Use closing characters (like quotation marks `"`, closing brackets `>`, or closing tags like `</script>`) to shut down the existing HTML element *first*, and *then* launch your script.
* **The Mental Model:** Imagine trying to drop a message inside a locked box. If you don't pick the lock (close the container) first, your message just gets stuck inside the walls where nobody can read it!

### 🖼️ Attack Style: The `innerHTML` Sink (Bypassing Modern Browser Defense)

**The Goal:** Force a page to execute JavaScript via `innerHTML` by using HTML elements with event handlers (like `<img>` or `<iframe>`) instead of blocked `<script>` tags.

---

#### 🧩 Concept: The `innerHTML` Security Gate

* **The Blockade:** Modern web browsers explicitly block `<script>` tags from executing when injected into `innerHTML`. Furthermore, inline events like `<svg onload=...>` won't fire either.
* **The Bypass:** While `<script>` tags are disabled, browsers *do* still render injected HTML elements like images and frames. By pairing these allowed elements with **event handlers** (like `onerror` or `onload`), you force the browser to execute JavaScript when the element fails or finishes loading.

---

#### 🛠️ The Implementation & Payload

* **Vulnerable Code:**
```javascript
element.innerHTML = '... ' + untrustedInput + ' ...';

```


* **The Payload (Image Error Trick):**
```html
<img src=1 onerror=alert(document.domain)>

```



---

#### 🎯 Why It Works

1. The browser sees the injected `<img>` tag and attempts to load an image from `src=1`.
2. Since `1` is an invalid image source, the load immediately fails.
3. The browser instantly triggers the `onerror` event handler, running your injected `alert(document.domain)` payload right inside the victim's session!

Some terms which you might not understand is simply given below:

---

### 🧱 Building Blocks: Elements vs. Attributes

#### 1. What is an HTML Element?

* **Simply put:** A complete building block on a webpage (like a button, paragraph, image, or box).
* **How it looks:** It usually starts with an opening tag `<tag>`, holds some content inside, and ends with a closing tag `</tag>`.
* **Example:** `<p>Hello world!</p>` is a text paragraph element.

#### 2. What is an HTML Attribute?

* **Simply put:** A setting or extra detail attached inside an element's opening tag to control how it behaves or where it gets data.
* **How it looks:** Written inside the opening tag as `name="value"`.
* **Example:** In `<a href="[https://google.com](https://google.com)">Click Me</a>`, `href="..."` is the **attribute** telling the link element where to send you when clicked.

---

### 🏷️ The Specific Tags & Sinks

#### 3. What is `<script>`?

* **Simply put:** The "code box" tag used to hold or load executable JavaScript code.
* **Role in Security:** Browsers normally run any code inside a `<script>` tag immediately when loading a page.

#### 4. What is `<img>`?

* **Simply put:** The HTML element used to display an image on a webpage.
* **How it works:** It uses the `src="image_link"` attribute to tell the browser where to fetch the picture.
* **Role in Security:** If the `src` link breaks (e.g., `src=x`), the browser triggers its `onerror="..."` attribute, allowing attackers to sneak JavaScript inside that error handler.

#### 5. What is `<iframe>`?

* **Simply put:** An "Inline Frame"—essentially a mini-browser window embedded inside a webpage that displays a completely separate website or HTML page inside it.
* **Example:** Embedding a YouTube video or Google Map on a blog post uses an `<iframe>`.

#### 6. What is `innerHTML`?

* **Simply put:** A JavaScript property that lets code read or replace the raw HTML content inside an element.
* **How it works:** If JavaScript runs `document.getElementById('box').innerHTML = '<b>Hi</b>'`, it replaces whatever was inside that box with bold text.
* **Role in Security:** It is a high-risk **Sink**. If untrusted user text is given directly to `innerHTML`, the browser turns that text into live HTML elements.

### 🧭 1. What is `location.search`?

* **Simply put:** It is a piece of JavaScript that grabs everything in the URL starting from the **question mark (`?`)** onward.
* **Example:** If the URL is `[https://example.com/shop?item=shoes&color=red](https://example.com/shop?item=shoes&color=red)`, then `location.search` is:
`?item=shoes&color=red`
* **Why hackers care:** It is the #1 **Source** for DOM-based XSS. Attackers use it to inject malicious text directly into a link, knowing the website's JavaScript will read that part of the URL and process it.

---

### ⚡ 2. What is an Event Handler?

* **Simply put:** A built-in trigger attribute attached to an HTML element that waits for a specific action to happen (like clicking, loading, or failing), and then instantly runs JavaScript code.
* **Common Examples:**
* `onclick`: Runs code when the user clicks an element.
* `onload`: Runs code when an element finishes loading.
* `onerror`: Runs code when an element **fails** to load (e.g., a broken image).


* **Why hackers care:** When modern websites block standard `<script>` tags, hackers use event handlers (like `onerror` inside a broken `<img>` tag) as a secret backdoor to force the browser to run their JavaScript anyway.

### 📦 Attack Style: Third-Party Libraries & jQuery (The Plugin Trap)

**The Goal:** Find insecure functions in third-party libraries (like jQuery) that take untrusted URL data and turn normal page elements into clickable JavaScript execution links.

---

#### 🧩 Concept: Plugin Sinks

* **The Hidden Danger:** Modern websites don't just use raw JavaScript; they rely heavily on third-party libraries like jQuery. These libraries come with their own built-in functions that can act as dangerous **Sinks**.
* **The jQuery `attr()` Flaw:** The jQuery `.attr()` function is used to change element attributes (like links or image sources). If a developer uses it to stuff raw URL data into an element attribute, you can hijack it.

---

#### 🛠️ The Implementation & Vulnerable Code

* **Vulnerable Code Example:**
```javascript
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnUrl'));
});

```


*What this code does:* It looks at the URL, grabs whatever text is in the `returnUrl` parameter, and injects it straight into the `href` attribute of a "Back" link.

---

#### 🎯 The Payload & Why It Works

* **The URL Payload:**
```text
?returnUrl=javascript:alert(document.domain)

```


* **Why It Works:**
1. The website reads your malicious `returnUrl` from the URL bar.
2. jQuery's `.attr()` function blindly sets the back link's destination to `javascript:alert(document.domain)`.
3. The moment the user clicks that "Back" link on the page, instead of navigating to a safe page, the browser executes your embedded JavaScript!

Some terms and codes simpler meaning given below which helped me alot before:

---

### 📚 1. What does a "Library" simply mean?

* **Simply put:** A library is a **bundle of pre-written code** created by someone else that developers download and plug into their own websites.
* **The Analogy:** Instead of building a table completely from scratch by cutting down trees and shaping wood, a developer goes to a store and buys a pre-made table leg or kit. Libraries save developers time so they don't have to reinvent the wheel.

---

### ⚡ 2. What is jQuery simply?

* **Simply put:** jQuery is just a **very famous and popular JavaScript library**.
* **Why it exists:** Regular JavaScript can sometimes be long and annoying to write. jQuery was invented to make JavaScript short, fast, and easy. Because it was used on millions of websites for over a decade, security researchers look closely at how developers use its special functions.

---

### 🔍 3. How do I know if a developer is stuffing raw URL data into an attribute?

* **Simply put:** You look at how the website behaves and inspect its JavaScript code.
* **The Clues to look for:**
1. Look at the links on the page (like a "Back" button, a "Next" page link, or a banner).
2. Change something in your URL bar (like adding `?returnUrl=test` at the end) and see if that text instantly appears inside the link's destination (`href`) when you inspect the element.
3. If you look at the website's JavaScript files (using `Ctrl+Shift+F` in DevTools) and see functions like `.attr("href", ...)` pulling data directly from the URL, you've found the dangerous pipeline!



---

### 🧩 4. Breaking down that JavaScript code piece by piece

Let's look at this code line:

```javascript
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnUrl'));
});

```

Here is what every single part is saying in plain English:

1. **`$(function() { ... })`**
* *"Hey browser, wait until the webpage finishes loading, and then run this little script right away."*


2. **`window.location.search`**
* *"Look up at the URL bar in the browser, and grab everything written after the question mark (`?`)."*


3. **`new URLSearchParams(...).get('returnUrl')`**
* *"Scan through that URL data, find the word `returnUrl`, and grab whatever value is attached to it."*
* *(For example, if your URL is `[site.com/page?returnUrl=google.com](https://site.com/page?returnUrl=google.com)`, this part extracts just `google.com`).*


4. **`$('#backLink').attr("href", ...)`**
* *"Find the HTML element on the page that has the ID `backLink` (which is usually a 'Back' button or link), and change its `href` setting (where it sends you when clicked) to whatever value we just grabbed from the URL."*



**The Security Flaw in One Sentence:** Because the code blindly takes whatever someone types into `returnUrl=` in the URL bar and pastes it straight into the "Back" button's destination, an attacker can type `javascript:alert(1)` instead of a normal link, turning a normal button into a weapon!

### ⚙️ Attack Style: The jQuery `$()` Selector Sink (The Auto-Scroll Trap)

**The Goal:** Exploit older versions of jQuery (or vulnerable custom code) where passing user-controlled data into the `$()` selector function turns normal UI features—like auto-scrolling—into an instant code execution trap.

---

#### 🧩 Concept: The Selector Trap

* **What is the `$()` Selector?** In jQuery, the dollar sign function is used to find or create HTML elements on a page (for example, `$('#myButton')` finds an element with that ID).
* **The Classic Flaw:** In older days, developers used this selector alongside `location.hash` (the part of the URL after the `#` symbol) to automatically scroll down to a specific comment or section when a page loaded.
* **The Danger:** If you passed HTML into that selector instead of just an ID name, jQuery would accidentally build that HTML right into the live webpage! (Though modern jQuery patches blocked this if the input starts with a `#`, unpatched or custom code in the wild is still vulnerable).

---

#### 🛠️ The Vulnerable Code & The Exploit

* **Vulnerable Code Example:**
```javascript
$(window).on('hashchange', function() {
    var element = $(location.hash);
    element[0].scrollIntoView();
});

```


*What this code does:* Whenever the URL hash changes (like when clicking an anchor link or scrolling), it takes whatever is in the hash, passes it into jQuery's `$()` selector, and scrolls the browser window to that element.
* **The iframe Exploit (Triggering it Without User Clicks):**
To force the browser to fire this vulnerability automatically without waiting for someone to click around, hackers use a clever `<iframe>` trick:
```html
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">

```


* **How the iframe trick works:** The iframe loads the target page. The moment it finishes loading (`onload`), it instantly slams our XSS payload (`<img src=1 onerror=alert(1)>`) onto the end of the URL hash. This instantly forces a `hashchange` event, bypassing the need for manual interaction and executing the alert box!



---

#### 🎯 Plain-English Breakdown of the Code

1. **`$(window).on('hashchange', ...)`** -> *"Hey website, keep an eye on the URL hash (the part after `#`). If it changes, run this code."*
2. **`var element = $(location.hash);`** -> *"Take whatever is written inside that hash, shove it into jQuery's selector, and treat it like an HTML element."*
3. **`element[0].scrollIntoView();`** -> *"Scroll the webpage down so that this element is front and center on the screen."*
* *The Hack:* Because jQuery tries to build whatever you typed into real HTML, typing a broken image tag with an `onerror` handler forces the browser to run your script instead of just scrolling!

### 🪞 Attack Style: Reflected DOM XSS (The Server-Client Tag Team)

**The Goal:** Exploit hybrid vulnerabilities where the server first echoes your input into the HTML response (reflection), and the browser's local JavaScript then picks it up and feeds it into a dangerous sink.

---

#### 🧩 Concept: When Server Reflection Meets Client Sinks

* **Pure DOM XSS vs. Hybrid:** Pure DOM XSS happens entirely inside the browser without the server ever seeing or touching the payload. But in **Reflected DOM XSS**, the server gets involved first.
* **The Trap Sequence:**
1. You send data to the server (e.g., via a URL parameter).
2. The server echoes that data right back into the HTML response—often hiding it inside a JavaScript variable or a form field.
3. The page's own JavaScript reads that server-reflected data and unsafely passes it to a danger zone (like `eval()`).



---

#### 🛠️ The Vulnerable Code & Execution

* **Vulnerable Code Example:**
```javascript
// The server injects your input directly into a JavaScript string literal:
eval('var data = "reflected string";');

```


*What this code does:* The page takes whatever text the server reflected, wraps it in quotation marks inside an `eval()` statement, and executes it.
* **The Payload / Breakout:**
If you control the "reflected string" portion, you don't just type text; you use quotes and semicolons to **escape** the variable assignment and append your own command:
```text
"; alert(document.domain); //

```


When the server reflects this, the resulting line of code becomes:
```javascript
eval('var data = ""; alert(document.domain); //";');

```


*Boom!* You closed the string, injected your own code, and commented out the rest of the line.

---

#### 🎯 Plain-English Summary

Think of Reflected DOM XSS as a relay race:

1. **The Server** is the first runner—it takes your input from the request and hands it off by printing it onto the webpage.
2. **The Client-Side Script** is the second runner—it picks up that printed data and blindly throws it into a danger zone like `eval()`, finishing the race and executing your attack.

### 💾 Attack Style: Stored DOM XSS (The Database-to-Browser Pipe)

**The Goal:** Exploit hybrid vulnerabilities where you plant data into a server's database first, and the browser's own local JavaScript later reads that stored data and feeds it unsafely into a danger zone.

---

#### 🧩 Concept: When Database Storage Meets Client Sinks

* **The Hybrid Threat:** In standard Stored XSS, the server outputs raw script tags directly. But in **Stored DOM XSS**, the server stores your input (like a profile name or comment), and a local client-side script later reads that stored data and passes it into a dangerous sink (like `innerHTML`).
* **The Trap Sequence:**
1. You send data to the server (e.g., registering an account with a malicious username).
2. The server saves it safely in the database.
3. Later, another user loads the page. The page's own JavaScript fetches that stored username from the server response and blindly drops it into an HTML sink.



---

#### 🛠️ The Vulnerable Code & Execution

* **Vulnerable Code Example:**
```javascript
// The local script takes stored database data and drops it into an element:
element.innerHTML = comment.author;

```


*What this code does:* The browser fetches a stored comment author name from the server and injects it directly into the webpage's live HTML using `innerHTML`.
* **The Exploit / Flow:**
If you signed up with a payload instead of a normal name (such as `<img src=1 onerror=alert(document.domain)>`), the database saves it. The moment another user views your comment, the local script executes `element.innerHTML = "<img src=1 onerror=alert(document.domain)>"`, turning your stored name into a live exploit!

---

#### 🎯 Plain-English Summary

Think of Stored DOM XSS as a delayed-action trap:

1. **The Plant:** You sneak a malicious payload into a server-stored field (like a username, bio, or comment).
2. **The Delivery:** The server hands that stored text down to a victim later.
3. **The Trigger:** The victim's browser runs its own local JavaScript, picks up your stored text, and blindly pushes it into a danger zone like `innerHTML`—firing the attack automatically!

### 🎯 Attack Style: XSS Contexts & HTML Tag Injection (The Open Field)

**The Goal:** Analyze the exact spot where your input lands on a webpage (the "context") and choose the right payload to execute code when your text falls right out in the open between HTML tags.

---

#### 🧩 Concept: What is an XSS Context?

* **The Context:** The precise location within the page's HTML response where your attacker-controlled data finally appears.
* **The Detective Work:** Before throwing random payloads at a website, you must figure out:
1. Where exactly is my input showing up? (Between tags? Inside quotes? Inside a script block?)
2. Is the website filtering, encoding, or chopping up my text?


* **The Strategy:** Once you know the exact context, you can pick a candidate payload specifically designed to break through that exact barrier.

---

#### 🛠️ The Scenario: XSS Between HTML Tags

* **The Setup:** Sometimes, your input lands directly into plain text space—right *between* existing HTML tags, like this:
```html
<p>Welcome back, [YOUR_INPUT_HERE]!</p>

```


* **Why It's Easy:** Because you aren't trapped inside quotation marks or attributes, you don't even need to "break out" of anything. You can just introduce completely **brand new HTML tags** that the browser will instantly render and execute!

---

#### 🎯 The Payloads for Open HTML Spaces

When your text lands safely between open tags, you can use direct execution payloads:

1. **The Classic Script Tag:**
```html
<script>alert(document.domain)</script>

```


* *What it does:* Tells the browser, "Stop reading normal text, execute this JavaScript block immediately."


2. **The Broken Image Trick:**
```html
<img src=1 onerror=alert(1)>

```


* *What it does:* Forces the browser to look for a fake image source (`1`), fail instantly, and run the `onerror` JavaScript command as a backup.

### 🏷️ Attack Style: XSS in HTML Tag Attributes (The Attribute Trap)

**The Goal:** Exploit situations where your input gets swallowed inside an HTML attribute value (like a quote-enclosed form field or link setting), either by breaking completely out of the tag or by smuggling in a secret event handler.

---

#### 🧩 Concept: Trapped Inside the Quotes

* **The Context:** When the website places your input inside an HTML attribute (e.g., `<input type="text" name="username" value="[YOUR_INPUT_HERE]">`).
* **The Barrier:** Because you are stuck inside quotation marks, regular text or raw tags won't work right away—you have to break out of the container first.
* **The Two Battle Strategies:**
1. **The Full Breakout:** If angle brackets (`<` and `>`) are allowed, you close the quote, close the tag entirely, and launch a brand new script tag.
2. **The Event Handler Smuggle:** If angle brackets are blocked or filtered, you stay inside the tag, close the quote, add your own malicious attribute (like an event handler), and gracefully fix the rest of the code.



---

#### 🛠️ The Payloads & How They Work

* **Strategy 1: The Full Tag Breakout**
* **The Payload:**
```html
"><script>alert(document.domain)</script>

```


* **Why it works:** The initial quote (`"`) closes the website's open attribute, the bracket (`>`) closes the HTML tag itself, and your `<script>` tag runs freely right after it.


* **Strategy 2: The Stealth Event Handler (When `< >` are blocked)**
* **The Payload:**
```html
" autofocus onfocus=alert(document.domain) x="

```





---

#### 🎯 Plain-English Breakdown of the Stealth Payload (`" autofocus onfocus=... x="`)

If the website blocks angle brackets `< >`, you can't use script tags. Instead, you smuggle your code inside an attribute:

1. **`"`** -> *"Close the quote on the website's existing attribute so we aren't trapped anymore."*
2. **`autofocus`** -> *"Tell the browser to instantly focus on this HTML element the moment the page loads (so the user doesn't even have to click anything!)."*
3. **`onfocus=alert(document.domain)`** -> *"The exact second the element gains focus, trigger this JavaScript alert code."*
4. **`x="`** -> *"Glue a fake attribute named `x` onto the end and open a dangling quote, just so the website's remaining HTML code doesn't break and crash the page."*

### 🔗 Attack Style: Sinking into Native Scriptable Attributes (The `href` Shortcut)

**The Goal:** Take advantage of special HTML attributes that are already designed to handle actions or links, allowing you to execute JavaScript directly without needing to break out of any quotation marks.

---

#### 🧩 Concept: The Built-In Code Pathway

* **The Special Attributes:** Not all HTML attributes are just plain text. Attributes like `href` (on anchor links) or `src` (on frames/scripts) are built by browsers to handle navigation and protocols.
* **The `javascript:` Shortcut:** Because these attributes expect a link or destination, you can completely skip trying to break out of quotes. Instead of typing a normal website address, you type the word `javascript:` followed by your payload, turning the link itself into a weapon.

---

#### 🛠️ The Payload & Execution

* **The Payload:**
```html
<a href="javascript:alert(document.domain)">Click Me</a>

```


* **Why You Don't Need Quotes:**
If your input lands safely inside an attribute that natively supports JavaScript protocols, you don't need to type `">` to close anything. The browser's engine is already waiting for a link action, making your payload instantly valid.

---

#### 🎯 Plain-English Breakdown

Think of this as a backdoor built right into the browser's architecture:

1. **The Destination:** The `href` attribute tells the browser where to go when someone clicks the link.
2. **The Hijack:** Instead of typing `[https://google.com](https://google.com)`, you feed it `javascript:alert(document.domain)`.
3. **The Trigger:** The moment the victim clicks the link, the browser reads the `javascript:` prefix, stops trying to load a webpage, and runs your code instead!

Some more clearer explanation of terms:

---

### 🔗 1. What is the `href` attribute simply?

* **Simply put:** `href` stands for **Hypertext Reference**. It is an attribute inside an HTML link that tells the browser **where to go** when the user clicks on it.
* **Example:** In `<a href="[https://google.com](https://google.com)">Click</a>`, the `href` is `[https://google.com](https://google.com)`.

---

### ⚓ 2. What are anchor links simply?

* **Simply put:** An "anchor link" is just the technical name for a **clickable link** on a webpage.
* It is created using the `<a>` HTML tag (which stands for "anchor"). It's the classic text, button, or image you click to jump to another page or website.

---

### 💾 3. Where is the "stored" XSS here?

To clarify, an `href` attribute **isn't automatically "stored" XSS** on its own—it can happen in reflected, DOM, or stored scenarios. But if we are talking specifically about **Stored XSS** happening through an `href` link, here is how that scenario plays out:

1. **The Plant (Saving it):** A user goes to their profile settings on a website and types `javascript:alert(1)` into their "Personal Website" or "Portfolio Link" field.
2. **The Database:** The website takes that input and **saves it in its database**, thinking it's a normal link.
3. **The Trigger (The Stored Part):** Later on, a completely different user visits that person's public profile page. The website pulls that saved `javascript:alert(1)` link from the database and injects it into an HTML anchor tag:
```html
<a href="javascript:alert(1)">Check out my profile!</a>

```


4. **The Attack:** The unsuspecting visitor clicks the profile link, and because the `href` contains a `javascript:` command instead of a real web address, the stored exploit instantly fires!

### ⌨️ Attack Style: The `accesskey` Trick (The Keyboard Shortcut Trap)

**The Goal:** Execute JavaScript when angle brackets are blocked and the target tag doesn't fire events automatically, by turning a harmless element into a secret keyboard shortcut that the user triggers for you.

---

#### 🧩 Concept: The Locked Tag Problem

* **The Defense:** The website blocks angle brackets (`<` and `>`), meaning you can't use `<script>` tags. Worse yet, you're stuck inside a passive tag (like a canonical link tag, e.g., `<link rel="canonical" href="...">`) that doesn't load images or fire automatic events on its own.
* **The Keyboard Shortcut Escape Hatch:** HTML has a built-in feature called **`accesskey`** that lets developers assign keyboard shortcuts to elements. If you can inject an `accesskey` and an event handler (like `onclick`) into that tag, you can trick the user's keyboard into firing your exploit!

---

#### 🛠️ The Payload & How It Works

* **The Injected Attribute Payload:**
```html
accesskey="x" onclick="alert(document.domain)"

```


*(Note: Depending on the browser and operating system, pressing a specific key combo like Alt + Shift + X will focus or activate that element).*
* **Why You Need User Interaction:**
Because tags like `<link>` or hidden inputs won't trigger automatically, you (or a social engineering trick) have to get the victim to press the specific shortcut key combination on their keyboard while they are on the page. The moment they do, the browser treats it as a click/activation, and your `onclick` code runs instantly!

---

#### 🎯 Plain-English Summary

Think of this as planting a landmine that requires a secret code:

1. **The Plant:** You inject an `accesskey` attribute into a tag where angle brackets are banned.
2. **The Setup:** You assign a letter (like `x`) and tie it to an `onclick` JavaScript alert.
3. **The Trigger:** Since the tag won't activate on its own, you prompt or instruct the victim to press the key combination. The browser fires the shortcut, and your payload executes right inside their session!

### 🕵️‍♂️ Recon Guide: How to Spot and Test for `accesskey` Vulnerabilities

**The Goal:** Discover if a website reflects your input into HTML attributes, and determine if you can inject or abuse the `accesskey` attribute to trigger an attack.

---

#### 🧩 Concept: The Reflection Hunt

* **What you are looking for:** You aren't usually looking to see if the website *already* uses `accesskey` by default (though you can search for the word `accesskey` in the source code). Instead, you want to see if the website **reflects your typed input inside an attribute tag** where you can inject your own code.
* **The Golden Rule:** If your input gets echoed inside an HTML tag attribute, you test what characters are allowed (are quotes allowed? are angle brackets blocked?). If angle brackets `< >` are blocked but quotes are allowed, `accesskey` becomes your primary weapon.

---

#### 🛠️ Step-by-Step: How to Test for It

1. **Step 1: Send a Unique Probe**
* Go to a search bar, a profile field, or a URL parameter (like `?search=`).
* Type a unique test string, such as `hacktest123`.


2. **Step 2: Inspect the Page Source**
* Right-click the page and select **View Page Source** (or press `Ctrl+U`), or use **Inspect Element** (`Ctrl+Shift+C`).
* Press `Ctrl+F` and search for your test string (`hacktest123`).


3. **Step 3: Analyze Your Location (The Context)**
* Look closely at *where* your `hacktest123` landed:
* *Is it trapped between normal text tags?* (You can use `<script>` tags).
* *Is it sitting inside an attribute?* (Like `<input value="hacktest123">` or `<link rel="canonical" href="hacktest123">`).




4. **Step 4: Test the Injection**
* If it's trapped inside an attribute and angle brackets are blocked, try injecting quotes and an access key to see if the server accepts it:
```text
" accesskey="X" onclick="alert(1)

```


* Reload the page, inspect the element again, and see if your `accesskey="X"` successfully embedded itself into the HTML structure without being stripped or safely encoded by the server!

### ✂️ Attack Style: Script Tag Breakout (Killing the Script Block)

**The Goal:** Break out of an existing JavaScript `<script>` block by prematurely closing it with a closing tag, allowing you to escape the JavaScript environment and inject fresh HTML that executes your code.

---

#### 🧩 Concept: HTML Parsing Beats JavaScript Parsing

* **The Context:** Your input lands inside an existing JavaScript variable or string block (e.g., inside `<script> var input = '...' </script>`).
* **The Browser's Secret Rule:** Browsers parse **HTML first** and **JavaScript second**.
* **The Exploit Logic:** When the browser scans the raw HTML, it is aggressively looking for closing tags. The moment it sees `</script>`, it immediately shuts down the script block—completely ignoring whether the JavaScript code inside was finished or left with broken, unclosed quotation marks! Once the script block is killed, you are back in normal HTML territory where you can run whatever tags you want.

---

#### 🛠️ The Payload & Execution

* **Vulnerable Setup:**
```html
<script>
var input = 'controllable data here';
</script>

```


* **The Payload:**
```html
</script><img src=1 onerror=alert(document.domain)>

```


* **Resulting HTML Code:**
```html
<script>
var input = '</script><img src=1 onerror=alert(document.domain)>';
</script>

```



---

#### 🎯 Plain-English Breakdown of How It Works

1. **`</script>`** -> *"Hey browser, stop reading JavaScript right now; this script block is officially closed!"* (Even though it leaves the original variable string open and technically broken).
2. **`<img src=1 onerror=alert(document.domain)>`** -> *"Now that we are out of the script block and back in open HTML land, render this broken image tag."*
3. **Why it succeeds:** The browser doesn't care that the previous JavaScript code was left messy or broken. It closed the script tag precisely when you told it to, and then immediately parsed and executed your new image error handler!

### ⛓️ Attack Style: Breaking Out of a JavaScript String (The Quote Escape)

**The Goal:** Escape a quoted string literal inside an active JavaScript block without triggering syntax errors, allowing you to execute JavaScript commands directly.

---

#### 🧩 Concept: Trapped Inside Quotes (And Why Syntax Matters)

* **The Context:** Your input lands safely inside a string variable in an existing script, like this:
```javascript
var greeting = 'Hello, [YOUR_INPUT_HERE]!';

```


* **The Problem:** Because you are trapped inside single quotes (`'`), typing `alert(1)` won't work—the browser just thinks you are typing a literal string of text, not code. If you try to close the quote prematurely, you'll leave the rest of the script hanging.
* **The Golden Rule of JS Execution:** JavaScript is strict. If your payload creates a syntax error (like unclosed brackets or dangling quotes), the browser's JavaScript engine crashes, cancels the script, and **nothing executes**. You must mathematically or syntactically **repair** the script after your payload.

---

#### 🛠️ The Payloads & How They Work

* **Option 1: The String Concatenation Trick**
* **The Payload:**
```text
'-alert(document.domain)-'

```


* **How it works:**
1. The first single quote (`'`) closes the website's original string.
2. The hyphen (`-`) tells JavaScript to subtract (or concatenate) your function.
3. Your code `alert(document.domain)` runs.
4. The trailing hyphen and quote (`-'`) cleanly reconnect your payload to the rest of the website's leftover string, preventing any syntax errors!


* *Resulting code:* `var greeting = 'Hello, '-'alert(document.domain)-''!';`




* **Option 2: The Statement Terminator & Comment Trick**
* **The Payload:**
```text
';alert(document.domain)//

```


* **How it works:**
1. The single quote (`'`) closes the website's string.
2. The semicolon (`;`) officially ends that line of JavaScript code.
3. Your code `alert(document.domain)` runs as a brand-new, separate command.
4. The double slash (`//`) turns the rest of the website's original code (including the leftover closing quote) into a **comment**, making the browser completely ignore it!


* *Resulting code:* `var greeting = 'Hello, ';alert(document.domain)//';`





---

#### 🎯 Plain-English Summary

Think of this like performing surgery inside a locked box:

1. **The Breakout:** You use a quote (`'`) to pop out of the string boundary.
2. **The Execution:** You run your code (`alert`).
3. **The Cleanup:** You use a math operator (`-`) or a comment (`//`) to stitch the broken code back together so the browser doesn't notice the hack and throw a crash error!

### 🔄 Attack Style: Backslash Neutralization (The Escape-Char Bypass)

**The Goal:** Defeat backend defense mechanisms that try to neutralize your quotes with a backslash (`\'`), by slipping in your own backslash to cancel theirs out and force a string breakout.

---

#### 🧩 Concept: The Escaped Quote Flaw

* **The Defense:** To prevent XSS, smart applications try to protect string variables by automatically inserting a backslash (`\`) before any single quote you type. This turns `'` into `\'`, telling the JavaScript parser, *"Treat this quote literally, do not close the string."*
* **The Blind Spot:** The application's sanitizer makes a classic rookie mistake: **it fails to escape the backslash itself**.
* **The Counter-Move:** Because the app doesn't sanitize your backslashes, you can type your own backslash right before your quote. When the app adds its own backslash in front of yours, it creates a double-backslash combo (`\\`), which completely neutralizes its own defense!

---

#### 🛠️ The Payload & How It Works

* **Your Input:**
```text
\';alert(document.domain)//

```


* **What the Application Turns It Into:**
```text
\\';alert(document.domain)//

```


* **Why It Works:**
When the JavaScript engine reads `\\'`, the first backslash cancels out the second one (making it a literal, harmless backslash). Because the second backslash is neutralized, **the single quote is left completely naked and exposed**, allowing it to act as a normal string terminator again!

---

#### 🎯 Plain-English Breakdown of the Trick

Think of this as a game of math and cancellation:

1. **You provide:** A backslash and a quote (`\'`).
2. **The app adds:** Its own safety backslash in front of yours, resulting in two backslashes and a quote (`\\`).
3. **The browser reads it:** *"Ah, two backslashes mean a literal slash, which leaves the quote unshielded."*
4. **The Breakout:** The quote successfully closes the string, the semicolon ends the statement, and your `alert()` payload fires!

What does backslash (\) simply really do?
In programming, the backslash (`\`) is known as the **"Escape Character."**

### 🛡️ What it simply does:

It acts like a **shield**. When you place a backslash right before a special symbol, it tells the computer:

> *"Stop treating this symbol as code or a command. Just treat it as normal, boring text."*

---

### 💡 An Everyday Example

Normally, in JavaScript, a single quote (`'`) is a **command** that means: *"End of the text string!"*

* If you write a sentence like `'It's sunny'`, the computer gets confused because the middle quote (`'`) looks like an accidental end of the string.
* To fix this, developers use a backslash as a shield: `'It\'s sunny'`.
* When the computer reads `\'`, it says: *"Ah, that backslash means I should ignore the special rule for this quote. It's just a normal punctuation mark, not the end of the string."*

### 🚨 Attack Style: Function Calling Without Parentheses (The Exception Bypass)

**The Goal:** Execute a JavaScript function (like `alert`) when strict character filters or a Web Application Firewall (WAF) block critical symbols like parentheses (`(` and `)`).

---

#### 🧩 Concept: Bypassing Parentheses with Error Handling

* **The Filter Problem:** Security systems often block parentheses because they are required to call functions (e.g., `alert(1)`). Without them, standard payloads fail.
* **The Exception Trick:** JavaScript has built-in error management. If you register a global error handler (`onerror`) and deliberately crash or "throw" an error, the browser will automatically catch it and hand your thrown value over to your function—completely skipping the need for parentheses!

---

#### 🛠️ The Payload & How It Works

* **The Payload:**
```javascript
onerror=alert;throw 1

```


* **Resulting Action:**
Instead of writing `alert(1)`, you configure the browser's error system to do the work for you under the hood.

---

#### 🎯 Plain-English Breakdown of the Code

1. **`onerror=alert`** -> *"Hey browser, if any unhandled error ever happens on this page, don't crash silently; instead, pass that error straight into the `alert` function."*
2. **`;`** -> *"End of statement."*
3. **`throw 1`** -> *"Deliberately throw an error with the value `1` into the system."*
4. **Why it succeeds:** The browser catches the thrown `1`, instantly triggers your `onerror` rule, and executes `alert(1)` behind the scenes—bypassing the blocked parentheses entirely!

### 🔣 Attack Style: HTML-Encoding Attribute Bypass (The Trojan Entity Trick)

**The Goal:** Bypass server-side character filters on event handlers (like `onclick`) by hiding forbidden characters inside HTML entities, tricking the sanitizer while letting the browser decode and execute them later.

---

#### 🧩 Concept: The Browser's Order of Operations

* **The Context:** Your input lands inside a JavaScript string that is nested inside an HTML attribute (e.g., `<a href="#" onclick="... var input='YOUR_INPUT'; ...">`).
* **The Server's Blind Spot:** The server blocks direct single quotes (`'`) to prevent you from breaking out of the string.
* **The Browser's Secret Timeline:** Browsers don't process everything at once. They parse **HTML first** (converting entities like `&apos;` back into punctuation) and pass code to the **JavaScript engine second**.
* **The Exploit Logic:** If you submit an HTML-encoded character, the server's security filter looks at it and says, *"That's just a harmless text entity, not a quote!"* But the moment it reaches the browser, the HTML decoder translates it right back into a dangerous quote before JavaScript ever touches it.

---

#### 🛠️ The Payload & How It Works

* **The Payload:**
```text
&apos;-alert(document.domain)-&apos;

```


* **Resulting Flow:**
1. **Input Sent:** `&apos;-alert(document.domain)-&apos;`
2. **HTML Decoding Phase:** The browser reads `&apos;` and turns it into `'`.
3. **JavaScript Execution Phase:** The script sees `' - alert(...) - '`, breaks out of the string, and fires the alert!



---

#### 🎯 Plain-English Breakdown of the Trick

Think of this as smuggling contraband past a security checkpoint:

1. **Disguising the Weapon:** Instead of bringing in a real single quote (`'`), you wrap it in a disguise (`&apos;`).
2. **Fooling the Guard:** The server's security filter checks your input, sees the entity, doesn't recognize it as a threat, and lets it straight through.
3. **The Unmasking:** Once inside the browser, the HTML parser strips away the disguise and turns `&apos;` back into a live single quote.
4. **The Breakout:** The quote acts as a string terminator, the math operators stitch the code together cleanly, and your script executes successfully!

### 🕹️ Attack Style: Template Literal Injection (The Backtick Bypass)

**The Goal:** Exploit modern JavaScript template literals by slipping your code directly into a built-in expression wrapper, allowing you to execute commands without even needing to break out of the string.

---

#### 🧩 Concept: The Danger of Dynamic Strings

* **What are Template Literals?** In modern JavaScript, developers use backticks (```) instead of normal quotation marks to create strings. These strings have a powerful built-in feature: any code wrapped inside a `${...}` block will be **automatically evaluated and executed** on the fly.
* **The Flap in the Armor:** Because template literals are specifically designed to process executable expressions inside them, you don't need to struggle with closing quotes or breaking characters. You just need to speak the string's native language!

---

#### 🛠️ The Payload & How It Works

* **Vulnerable Setup:**
```javascript
var input = `controllable data here`;

```


* **The Payload:**
```text
${alert(document.domain)}

```


* **Resulting Code:**
```javascript
var input = `${alert(document.domain)}`;

```



---

#### 🎯 Plain-English Breakdown of the Trick

Think of this as hijacking a feature meant for convenience:

1. **The Native Feature:** The script is built to look inside backtick strings for dynamic variables using the `${...}` syntax.
2. **The Hijack:** Instead of typing a variable name, you drop your own function call right inside the braces.
3. **The Execution:** The moment the JavaScript engine reads your input, it pauses to evaluate the expression inside the brackets, running your payload instantly as part of the string's normal processing!

### 🧩 Attack Style: Client-Side Template Injection (The Framework Hijack)

**The Goal:** Exploit applications that use dynamic front-end frameworks (like AngularJS) by injecting malicious template expressions into spots where the framework automatically evaluates code on the fly.

---

#### 🧩 Concept: When Frameworks Become Accomplices

* **The Framework's Job:** Modern web applications use client-side template frameworks (such as AngularJS) to dynamically build and update web pages. These frameworks actively scan the HTML for special syntax—like double curly braces `{{ ... }}`—and execute whatever code they find inside them.
* **The Flaw:** If a website takes your user input and unsafely drops it into a region managed by one of these frameworks, the framework won't treat your text as boring strings. It will parse it as part of its own application logic and run it!

---

#### 🛠️ The Payload & How It Works

* **The Concept / Proof-of-Concept Expression:**
```text
{{7*7}}

```


*What this does:* When injected into an AngularJS app, the framework evaluates the math inside the curly braces and replaces it on the screen with `49`, proving the template engine is actively running your input.
* **The Full XSS Execution:**
By chaining AngularJS objects and constructors (often bypassing internal framework sandboxes), attackers turn a simple expression into a full code-execution payload:
```text
{{$on.constructor('alert(document.domain)')()}}

```



---

#### 🎯 Plain-English Breakdown of the Trick

Think of this as hijacking an automated assistant living inside the browser:

1. **The Watchful Engine:** The client-side framework is constantly scanning the webpage, looking for special template markers to evaluate.
2. **The Trojan Expression:** You feed input into the page wrapped in the framework's native syntax (like `{{ ... }}`).
3. **The Execution:** The framework finds your input, assumes it's safe developer code, evaluates it, and executes your JavaScript automatically—completely bypassing traditional HTML tags!

### 🔓 Attack Style: AngularJS Sandbox Escape & Advanced Bypasses (Breaking the Cage)

**The Goal:** Defeat legacy AngularJS sandbox restrictions to execute arbitrary JavaScript, even when security filters block quotes or standard execution sinks like `$eval` are missing.

---

#### 🧩 Concept: The Illusion of Security (The AngularJS Sandbox)

* **The Sandbox:** Older versions of AngularJS featured an internal security guard designed to block access to dangerous global objects (`window`, `document`) and restricted properties (`__proto__`).
* **The Flaw:** The sandbox didn't rely on a true security boundary; instead, it used string parsing and function checks (`ensureSafeObject`, `ensureSafeMemberName`) to inspect code *before* running it. Smart researchers found ways to manipulate the framework's own helper functions to trick it into clearing malicious payloads as safe.
* **The Advanced Hurdles:** If a site blocks quotes entirely, you can't type normal strings to build your payload. And if `$eval` is disabled, you have to use alternative framework filters (like `orderBy`) to force execution!

---

#### 🛠️ The Payloads & How They Work

* **Technique 1: The Basic Sandbox Escape (Fooling `isIdent`)**
* **The Payload:**
```text
'a'.constructor.prototype.charAt=[].join

```


* **Why it works:** By overwriting the global `charAt` function with JavaScript's array `join` method, you trick the sandbox's character verification logic (`isIdent`). Instead of checking characters one by one, it reads entire strings as valid identifiers, allowing you to bypass the sandbox rules completely.


* **Technique 2: The Quote-Free & `$eval`-Less Execution (`orderBy` Filter)**
* **The Payload Format:**
```text
[123]|orderBy:'YOUR_PAYLOAD_HERE'

```


* **Why it works:** The pipe symbol (`|`) in AngularJS triggers a **filter operation** rather than a bitwise math check. Passing an array into the `orderBy` filter forces the framework to evaluate whatever string argument you attach to it—acting as a secret alternative execution engine when `$eval` is blocked or unavailable.



---

#### 🎯 Plain-English Breakdown of the Trick

Think of this like breaking out of a high-tech prison cell from the inside:

1. **The Disguise (`charAt` hack):** You tamper with the framework's internal counting tools (`charAt`), forcing it to misread your input and falsely declare your malicious code as safe text.
2. **The Workaround (No Quotes):** If quotes are banned, you use existing string constructors (`[][...].constructor`) to dynamically build letters without typing them directly.
3. **The Execution Switch (`orderBy`):** If the standard trigger (`$eval`) is locked down, you funnel your payload through the `orderBy` filter. The framework accepts the array, processes your string argument, and executes your JavaScript code live in the browser!

### 🛡️ Attack Style: AngularJS CSP Bypass (Smuggling Code Through Framework Events)

**The Goal:** Defeat strict Content Security Policies (CSP) and AngularJS security guards by exploiting custom framework events and hiding your function calls inside array methods.

---

#### 🧩 Concept: Sneaking Past the Guardrails

* **The Defense:** A strict CSP blocks standard script tags, and the AngularJS sandbox blocks you from directly typing dangerous names like `window.alert`.
* **The Event Loophole:** AngularJS has its own built-in events (like `ng-focus`). Inside these events, a special helper called `$event.path` tracks where the event came from—and its final stop is always the powerful global `window` object!
* **The Disguise:** To prevent the sandbox from yelling at you for touching `window`, you feed the path into array tools (like `map()` or `from()`). This triggers your function indirectly, completely bypassing detection.

---

#### 🛠️ The Payloads & How They Work

* **Payload 1: The Path & `from` Trick**
```html
<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">

```


* *Why it works:* It grabs the event's history (`$event.path`), uses the `orderBy` filter to run it, and uses `.from()` to quietly execute `alert` on an array without directly referencing the forbidden `window` object.


* **Payload 2: The Short-Form `map` Bypass**
```text
[1].map(alert)

```


* *Why it works:* Passing `alert` directly into an array's `.map()` function forces JavaScript to run it on every item in the array, executing your code cleanly without setting off sandbox alarms.



---

#### 🎯 Plain-English Breakdown of the Trick

Think of this like smuggling a message through a trusted delivery network:

1. **The Wake-Up Call (`ng-focus`):** You trigger a custom AngularJS event so the framework's engine starts running.
2. **Finding the Master Key (`$event.path`):** You grab the event's tracking path, which secretly leads straight to the global `window` object.
3. **The Hidden Hand-off (`map` / `from`):** Instead of shouting `alert()`, you hand the function over to an array helper (`map` or `from`). The framework thinks it's just doing normal data processing, and your alert pops up safely!

### 🍪 Attack Style: The Cookie Heist (Session Hijacking via XSS)

**The Goal:** Capture a victim's active session cookies through XSS, send them back to a server you control, and clone their login session to impersonate them.

---

#### 🧩 Concept: Stealing the Digital Keycard

* **The Power of Cookies:** Most web apps use session cookies as digital keycards to keep users logged in. If you can read those cookies using JavaScript, you hold the keys to their account.
* **The Exfiltration Trick:** Once your XSS payload runs, it grabs `document.cookie` and secretly sends it over the network to an endpoint you control (like an attacker-owned listener or log server).

---

#### ⚠️ The Reality Check: Why It Isn't Always Easy

While classic in theory, real-world cookie theft faces major hurdles:

1. **The Guest Problem:** The victim might not even be logged into the application when they hit your payload.
2. **The `HttpOnly` Shield:** Secure apps protect critical session cookies by marking them with the `HttpOnly` flag, which makes them **completely invisible** to JavaScript (`document.cookie` won't show them).
3. **IP Locking:** Some apps tie session tokens to a specific IP address; stealing the cookie won't work if your IP doesn't match the victim's.
4. **Timeouts:** Sessions expire quickly, leaving you a tiny window to hijack the account before it dies.

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like stealing a hotel room keycard:

1. **The Entry Point:** Your XSS payload executes inside the victim's browser session.
2. **The Grab:** The script reaches into `document.cookie` to snatch the active session ID.
3. **The Transmission:** It packs the keycard details into a background request and beams it back to your server.
4. **The Impersonation:** You take that stolen cookie, paste it into your own browser, and trick the website into thinking *you* are the legitimate user—unless security defenses like `HttpOnly` block the read!

### 🔑 Attack Style: Password Manager Hijack (The Auto-Fill Trap)

**The Goal:** Capture a user's sensitive credentials by tricking their browser's password manager into auto-filling a fake login field injected via XSS, bypassing cookie protections entirely.

---

#### 🧩 Concept: Exploiting Auto-Fill Convenience

* **The Setup:** Modern browsers and password managers love convenience—the moment they see an input field designated for passwords (`<input type="password">`), they eagerly auto-fill saved credentials.
* **The Trap:** If you use XSS to inject a hidden or disguised password field into the page, the user's password manager might automatically populate it with their real username and password without them even realizing it.
* **Why It Beats Cookie Theft:** Unlike `HttpOnly` cookies (which are hidden from JavaScript), captured plaintext passwords give you the actual keys to the kingdom. Even better, if the user reuses passwords across websites, you get access to all of their other accounts, too!

---

#### ⚠️ The Reality Check: Limitations & Fallbacks

1. **Dependency on Password Managers:** This attack only works if the target actually uses a browser or extension that auto-fills passwords.
2. **The Phishing Fallback:** If nothing gets auto-filled, attackers often fall back to on-site phishing—tricking the user with a realistic fake login prompt that forces them to type their credentials manually.

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like setting a digital mousetrap for credentials:

1. **The Plant:** Your XSS payload dynamically injects a password input field onto the target page.
2. **The Auto-Fill Trigger:** The victim's password manager detects the field and automatically injects their saved login data.
3. **The Snatch:** Your background script grabs the filled values (`.value`) from the input field.
4. **The Exfiltration:** It beams the stolen username and password straight back to your server, giving you permanent access regardless of session timeouts or `HttpOnly` shields!

### 🔄 Attack Style: CSRF Token Heist (Bypassing Anti-CSRF Defenses)

**The Goal:** Defeat anti-CSRF tokens using XSS to execute a "two-way" attack, allowing you to steal hidden protection tokens, force unauthorized actions (like changing an account email), and lock the user out via a password reset.

---

#### 🧩 Concept: Turning a One-Way Street Into Two-Way Control

* **The Traditional CSRF Limit:** Normal CSRF is a "one-way" attack—you can trick a victim's browser into sending a request, but you can't see the response or read hidden anti-CSRF tokens meant to protect the form.
* **The XSS Advantage:** Because XSS executes JavaScript directly inside the victim's session, **CSRF tokens become completely useless**. JavaScript can inspect the page, read hidden form values, and extract those tokens instantly.
* **The Takeover Chain:** If a website lets users change their email without re-entering their current password, an attacker can:
1. Use XSS to fetch the settings page and steal the CSRF token.
2. Use that token to change the victim's email to one controlled by the attacker.
3. Trigger a password reset on the login page to take absolute ownership of the account.



---

#### 🛠️ The Attack Mechanism & Flow

* **The Conceptual JavaScript Payload:**
```javascript
// Step 1: Fetch the sensitive page containing the hidden CSRF token
fetch('/accountSettings').then(res => res.text()).then(html => {
    // Step 2: Extract the CSRF token using a DOM parser or regex
    let parser = new DOMParser();
    let doc = parser.parseFromString(html, 'text/html');
    let csrfToken = doc.querySelector('input[name="csrf"]').value;

    // Step 3: Send a forged request to change the email using the stolen token
    fetch('/accountSettings/update', {
        method: 'POST',
        body: 'csrf=' + csrfToken + '&email=attacker@evil.com'
    });
});

```



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like breaking into a high-security vault by subverting the alarm system:

1. **The Secret Guard (CSRF Tokens):** The website uses unique, hidden tokens on every form to prove a request came from a genuine button click, blocking traditional forged attacks.
2. **The Insider Threat (XSS):** Because your XSS script is running live inside the victim's browser, it doesn't need to guess the token; it can just read it straight off the DOM.
3. **The Silent Takeover:** The script steals the token, quietly updates the account's email address behind the scenes, and triggers a password reset—all without the victim ever noticing until they are locked out!
