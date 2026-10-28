### 📁 Attack Style: Path Traversal (The File-System Escape)

**The Goal:** Read (and potentially write) sensitive files on the server's filesystem by manipulating file paths to bypass application restrictions and escape into unauthorized directories.

---

#### 🧩 Concept: Wandering Where You Aren't Allowed

* **What is Path Traversal?** Also known as directory traversal, this vulnerability occurs when an application takes user input to load files from disk without properly sanitizing or restricting the path.
* **The Target Data:** Successful exploitation lets an attacker peer deep into the server's filesystem to harvest application code, database credentials, configuration files, and critical operating system files.
* **The Ultimate Escalation:** While reading files is dangerous enough, some path traversal bugs go further—allowing attackers to *write* to arbitrary files on the server, modify app behavior, or achieve full server takeover.

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like sneaking through the unauthorized back hallways of a secure building:

1. **The Intended Route:** The application has a feature designed to fetch specific files from a single, locked-down folder (like pulling up a product image).
2. **The Flawed Direction:** Because the app blindly trusts the filename you hand it, you can slip special navigation commands (like stepping backward out of folders) into your request.
3. **Escaping the Sandbox:** Instead of staying trapped inside the app's designated folder, your path instructions climb backward through the directory tree and slip into restricted parts of the server.
4. **The Score:** You walk away reading system secrets, backend keys, and private data that were never meant to be exposed to the internet!

### 📁 Attack Style: Path Traversal Reconnaissance (Finding Files and Parameters)

**The Goal:** Discover vulnerable file-handling parameters in a web application and select reliable, universal test files to confirm whether a path traversal flaw exists.

---

#### 🧩 Concept: Scouting for Doors That Handle Files

* **The Recon Challenge:** Path traversal vulnerabilities don't appear randomly; they only exist where an application takes user input and uses it behind the scenes to fetch files from the server's disk.
* **Hunting for Parameters:** You need to scan the application for features that deal with files—such as image galleries, document downloads, template renderers, or file managers—and inspect the URL parameters or POST requests driving them.
* **The Universal Test Subject:** When proving a vulnerability for the first time, you don't guess random filenames. You use standardized, universally known files that are guaranteed to exist on default installations of the target operating system.

---

#### 🛠️ The Recon Toolkit & Common Signatures

* **1. Common Parameter Names to Hunt For:**
Keep an eye out for these parameter names in URLs, API requests, and form inputs:
* `filename=` (e.g., `?filename=avatar.png`)
* `file=` (e.g., `?file=report.pdf`)
* `path=` (e.g., `?path=images/logo.jpg`)
* `document=` or `doc=`
* `folder=` or `root=`
* `include=` or `template=`


* **2. Choosing Your First Proof-of-Concept (PoC) File:**
Once you find a promising file-handling parameter, swap the normal filename with a classic, high-certainty target:
* **On Linux / Unix Servers:** Test with `/etc/passwd`. It is a standard text file containing registered user accounts, and it almost always has world-readable permissions.
* *Payload:* `?filename=../../../../etc/passwd`


* **On Windows Servers:** Test with `windows\win.ini` or `boot.ini`. These are foundational configuration files present across virtually all legacy and modern Windows systems.
* *Payload:* `?filename=..\..\..\windows\win.ini`





---

#### 🎯 Plain-English Breakdown of the Technique

Think of this like scouting a facility for doors that accept physical keys before trying your picklocks:

1. **Spotting the Features:** You look for web pages that load dynamic content—like clicking a profile picture, downloading a PDF receipt, or viewing a help manual.
2. **Reading the URL Clues:** You inspect your browser's address bar or proxy history to find parameters like `?file=` or `?path=` that act as a bridge between the website and the server's storage room.
3. **Inserting the Universal Key:** Instead of guessing what files exist in the app, you inject a traversal path pointing straight to a world-famous landmark file like `/etc/passwd`.
4. **The Confirmation:** If the browser suddenly prints out system usernames and configurations instead of an image or a document, you instantly know you've found a path traversal entry point!

### 📁 Attack Style: Path Traversal via Directory Climbing (Stealing Files from the Root)

**The Goal:** Read sensitive system files outside of a web application's intended directory by injecting dot-dot-slash sequences (`../`) to force the filesystem to step backward through folder levels.

---

#### 🧩 Concept: Stepping Backward Out of the Trap

* **The Backend Flaw:** When an application loads a file (like an image) by taking user input and directly gluing it onto a base directory path (e.g., `/var/www/images/ + filename`), it often fails to check if the user tries to escape the folder.
* **The Power of `../`:** In file systems across almost all operating systems, the special sequence `../` means *"step up one level in the directory structure."*
* **Escaping the Sandbox:** By chaining multiple `../` sequences together, you can climb out of the application's restricted image folder, navigate all the way up to the filesystem root (`/`), and target sensitive files like `/etc/passwd` on Unix or `win.ini` on Windows.

---

#### 🛠️ The Payload & How It Works

* **The Vulnerable Parameter URL:**
```text
https://insecure-website.com/loadImage?filename=../../../etc/passwd

```


* **How the Backend Appends the Path:**
The server takes your input and combines it with its base folder:
```text
/var/www/images/../../../etc/passwd

```


* **The Directory Climb Breakdown (Unix):**
1. `images/` $\rightarrow$ Steps back to `/var/www/`
2. `../` $\rightarrow$ Steps back to `/var/`
3. `../` $\rightarrow$ Steps back to the filesystem root `/`
4. `etc/passwd` $\rightarrow$ Navigates down into the system directory to grab the user account file.


* **The Windows Equivalent:**
Windows systems support both forward slashes and backslashes, utilizing `..\` to achieve the same climb:
```text
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini

```



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like sneaking out of a hotel room using the service stairwells:

1. **The Locked Hallway:** The application intends for you to stay locked inside a single room (the `/var/www/images/` folder) to grab decorations.
2. **The Exit Trick:** Instead of a normal filename, you type a series of backtracking steps (`../../`) into the door slot.
3. **Climbing the Stairs:** Each `../` forces the building's directory system to walk backward out of the current room, down the hallway, and out into the main lobby.
4. **Stealing the Records:** Once you reach the main office files (like `/etc/passwd`), the application blindly reads the confidential document and hands it right back to your browser!

### 📁 Attack Style: Path Traversal Bypass via Absolute Paths (The Shortcut Leap)

**The Goal:** Read sensitive files on a target server even when the application blocks or strips directory traversal sequences (`../`), by skipping the climb entirely and providing an absolute path.

---

#### 🧩 Concept: Skipping the Maze When You Know the Address

* **The Defense Block:** Many developers attempt to stop path traversal by installing a filter that searches for and deletes any `../` or `..\` sequences found in user input.
* **The Absolute Path Flaw:** If the application simply appends your input to a base directory without checking if you are supplying a relative or absolute path, you can bypass the block entirely. Instead of stepping backward from a folder, you provide the full, absolute path starting directly from the filesystem root.
* **Why It Works on Unix:** On Linux systems, passing a path that starts with a slash (like `/etc/passwd`) overrides or bypasses many basic concatenation assumptions, and systems treat multiple consecutive slashes (like `/var/www/images//etc/passwd`) as a single slash, landing you right at the target file!

---

#### 🛠️ The Payload & How It Works

* **The Injected Absolute Path URL:**
```text
https://insecure-website.com/loadImage?filename=/etc/passwd

```


* **How the Payload Works:**
* No `../` required $\rightarrow$ Because the filter is only looking for backtracking dots, your payload completely slips past the security check.
* `/etc/passwd` $\rightarrow$ The absolute path pointing straight to the target system file from the filesystem root.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like bypassing a locked maze when you already know the exact street address:

1. **The Blocked Door:** The developer put up security guards to catch anyone trying to walk backward through the hallways using `../`.
2. **Dropping the Maze:** Instead of fighting the guards or trying to sneak backward, you realize you don't need to walk through the hallways at all.
3. **The Direct Teleport:** You type the exact global coordinates (`/etc/passwd`) straight into the input box.
4. **The Bypass:** Because the app's filter was only scanning for backtracking dots and ignored absolute paths, it blindly accepts your input, drops its guard, and hands you the file directly!

### 📁 Attack Style: Path Traversal Bypass via Nested Sequences (The Self-Healing Path)

**The Goal:** Bypass naive security filters that strip directory traversal characters by using nested patterns that automatically collapse back into working `../` commands once filtered.

---

#### 🧩 Concept: Exploiting Non-Recursive Filters

* **The Naive Defense:** Many developers try to block path traversal by writing a simple filter that searches for `../` and deletes it from the user's input.
* **The Flaw (Non-Recursive Stripping):** If the application only scans and strips the string *once* instead of repeatedly until no patterns remain, it opens the door to nested tricks.
* **The Nested Collapse:** If you type a nested sequence like `....//`, the filter deletes the inner `../`. But once that inner piece is gone, the remaining characters on either side snap together to form a brand-new, fully functional `../` sequence!

---

#### 🛠️ The Payload & How It Works

* **The Injected Nested Payload:**
```text
https://insecure-website.com/loadImage?filename=....//....//....//etc/passwd

```


* **How the Filter Breaks It:**
1. The application's filter scans your input and finds the inner `../` sequences.
2. It strips them out cleanly.
3. **The Collapse:** The outer characters (`..` and `/`) left behind stitch themselves together, turning `....//` into a standard `../`.


* **Resulting File Path:**
The sanitized input successfully reverts to standard traversal steps:
```text
/var/www/images/../../../../etc/passwd

```



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like smuggling a banned word past a lazy censor:

1. **The Single-Pass Censor:** The security guard is programmed to read your note once, look for the forbidden word `../`, and cross it out with a marker.
2. **Hiding Inside the Word:** Instead of writing a simple `../`, you double up the letters: `....//`.
3. **The Erasure Trick:** The lazy guard spots the middle `../`, crosses it out, and hands the note back, assuming they successfully cleaned it.
4. **The Automatic Repair:** Because the guard only looked once, the outer pieces of your hidden code snap together after the deletion, leaving a fresh, working traversal command behind to open the back door!

### 📁 Attack Style: Path Traversal Bypass via URL Encoding (The Disguised Traversal)

**The Goal:** Bypass web servers or application filters that strip literal directory traversal characters (`../`) by encoding or double-encoding the payload so it slips past security checks before being decoded by the backend.

---

#### 🧩 Concept: Writing in Secret Code to Fool the Censor

* **The Guard's Blind Spot:** Some web servers, WAFs, or URL parsers are programmed to aggressively search for and delete literal `../` sequences before they ever reach the application.
* **The Encoding Trick:** URL encoding translates characters into hexadecimal codes (e.g., `.` becomes `%2e` and `/` becomes `%2f`). If the security filter only looks for literal dots and slashes, it won't recognize your encoded string and will let it pass right through.
* **Double-Encoding & Quirks:** If a system decodes your input *twice*, single encoding might still get caught. In those cases, double encoding (`%252e%252e%252f`) or non-standard server quirks (`..%c0%af`) can trick the parser into executing the traversal anyway.
* **Automating the Fuzzing:** Testing dozens of encoding variants manually is tedious. In tools like Burp Suite Professional, security testers rely on predefined lists like **Fuzzing - path traversal** in Burp Intruder to blast all variations at once.

---

#### 🛠️ The Encoding Toolkit & Payloads

* **Standard URL Encoding:**
* `%2e%2e%2f` $\rightarrow$ Encoded version of `../`
* *Example:* `[https://insecure-website.com/loadImage?filename=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd](https://insecure-website.com/loadImage?filename=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd)`


* **Double URL Encoding (To survive multiple decode passes):**
* `%252e%252e%252f` $\rightarrow$ Double-encoded version of `../`


* **Non-Standard / Overlong Unicode Encodings:**
* `..%c0%af` or `..%ef%bc%8f` $\rightarrow$ Alternative representations of slashes that some misconfigured servers still interpret as valid file paths.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like slipping a coded message past a suspicious customs inspector:

1. **The Guard's Rulebook:** The security inspector is strictly looking for the exact letters `../` and will tear up any note containing them.
2. **Writing in Code:** Instead of writing the forbidden letters, you translate them into hexadecimal cipher text (like `%2e%2e%2f`).
3. **Passing Inspection:** The inspector looks at your encoded message, sees no forbidden symbols, and waves it straight through the checkpoint.
4. **The Translation at the Desk:** Once your message clears the guard and reaches the final desk, the application decodes the cipher back into normal text, revealing the original traversal commands and granting you access to the files!

### 📁 Attack Style: Path Traversal via Base Folder Prefixing (The Pre-Appended Escape)

**The Goal:** Bypass path validation checks that demand user input start with a specific base folder, by satisfying the prefix requirement first and then immediately climbing backward to target files.

---

#### 🧩 Concept: Saying the Magic Password at the Door

* **The Strict Check:** To stop path traversal, a developer programs the application to verify that any requested filename *must* start with the authorized base folder (e.g., `/var/www/images/`). If your input doesn't start with that exact string, the app rejects it.
* **The Validation Flaw:** Checking a prefix string only ensures your input *begins* correctly; it doesn't stop you from navigating away *afterward*.
* **The Prefix-and-Climb Trick:** If you manually type the required base folder at the front of your payload, you trick the app's initial validation check into passing. Once inside, you append your traversal sequences (`../`) to walk right back out of that folder and access sensitive system files!

---

#### 🛠️ The Payload & How It Works

* **The Injected Prefix Payload URL:**
```text
https://insecure-website.com/loadImage?filename=/var/www/images/../../../etc/passwd

```


* **How the Payload Works:**
* `/var/www/images/` $\rightarrow$ Satisfies the application's strict requirement that the filename must begin with the expected base directory.
* `../../../` $\rightarrow$ Immediately steps backward three levels to escape the image folder and reach the filesystem root.
* `etc/passwd` $\rightarrow$ Navigates down into the system directories to grab the target file.



---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like showing the security guard a required badge just to immediately slip through a secret exit inside the room:

1. **The Guard's Rule:** The application's bouncer checks your ID card and says, *"You can only enter if your path starts with the approved image folder!"*
2. **Playing Along:** Instead of fighting the rule, you hand them an ID that literally starts with `/var/www/images/`.
3. **Passing the Check:** The app sees the correct prefix, assumes you are playing by the rules, and lets your request through.
4. **The Trapdoor:** The moment you are past the check, your trailing traversal steps (`../../`) kick in, walking you straight out of the image room and into the server's confidential files!

### 📁 Attack Style: Path Traversal Bypass via Null Byte Injection (The File Extension Truncator)

**The Goal:** Bypass application filters that demand a specific file extension (like `.png`) by injecting a null byte to trick the backend parser into cutting off the filename early and ignoring the mandatory suffix.

---

#### 🧩 Concept: The Invisible Stop Sign

* **The Extension Check:** To prevent non-image files from being loaded, developers often configure applications to check that any requested filename ends with a specific extension (e.g., `.png` or `.jpg`).
* **The Legacy Backend Flaw:** Many older web application languages and underlying operating system APIs (especially those written in C, like older PHP versions) handle strings using null bytes (`\x00`) as strict end-of-string markers.
* **The Truncation Trick:** If you insert a URL-encoded null byte (`%00`) right after your desired target file, the backend file-handling function stops reading the string the exact moment it hits that byte. It completely ignores the mandatory `.png` extension sitting at the end, while the file system successfully opens the file you actually wanted!

---

#### 🛠️ The Payload & How It Works

* **The Injected Null Byte Payload URL:**
```text
https://insecure-website.com/loadImage?filename=../../../etc/passwd%00.png

```


* **How the Payload Works:**
* `../../../etc/passwd` $\rightarrow$ The standard traversal sequence navigating out of the image directory to the target system file.
* `%00` $\rightarrow$ The URL-encoded null byte (representing `\x00`). This acts as an invisible instruction to the backend code saying, *"Stop reading the filename right here."*
* `.png` $\rightarrow$ The required file extension placed at the very end to satisfy the application's security validation check.


* **The Parsing Result:**
The application's validation logic sees `.png` at the end and approves the request. However, when the backend code passes the string to the operating system's file reader, the null byte cuts the string short, resulting in the system opening `/etc/passwd` instead!

---

#### 🎯 Plain-English Breakdown of the Attack

Think of this like writing an invisible "stop reading here" line on a document to fool a lazy editor:

1. **The Extension Rule:** The application's editor demands that every single file submission must end with an approved `.png` label, or it gets thrown in the trash.
2. **Hiding the Divider:** You write your secret target (`/etc/passwd`), drop an invisible terminal marker (`%00`) right behind it, and then casually append `.png` at the very end.
3. **Passing the Editor's Check:** The application checks the end of your filename, spots the familiar `.png` label, checks off the requirement, and assumes everything is safe.
4. **The Truncation:** When the backend machine actually processes the file request, it hits your invisible stop marker, cuts off the sentence immediately, and hands over the confidential system file instead of an image!
