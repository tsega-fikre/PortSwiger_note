![[Pasted image 20261006212303.png]]

## Lab Description

This lab contains a **DOM-based Cross-Site Scripting (XSS)** vulnerability in an AngularJS expression within the search functionality.

AngularJS processes HTML elements containing the `ng-app` directive and evaluates expressions placed inside double curly braces `{{ }}`. Because of this behavior, an attacker may be able to inject an AngularJS expression that executes JavaScript.

The objective of the lab is to execute an AngularJS expression that calls the JavaScript `alert()` function.

## Objective

Exploit the search functionality to inject an AngularJS expression and execute JavaScript using `alert()`.

## Steps to Reproduce

### 1. Open the vulnerable search functionality

I opened the lab and navigated to the **search bar**.

I first entered a normal search term to confirm that the search functionality was working normally.

### 2. Test for AngularJS expression evaluation

I entered:

```text
{{1+1}}
```

Instead of displaying the input literally, the application evaluated the expression and returned:

```text
2
```

This confirmed that user-controlled input was being interpreted as an **AngularJS expression**.
![[Pasted image 20261006212714.png]]
### 3. Inject a JavaScript-executing AngularJS expression

I then tested the following payload:

```text
{{ constructor.constructor('alert(1)')() }}
```

The application evaluated the expression and executed the JavaScript `alert()` function.

A browser alert dialog appeared containing:

```text
1
```
![[Pasted image 20261006212419.png]]
### 4. Confirm the vulnerability

The successful execution of `alert(1)` demonstrated that an attacker can execute JavaScript through the vulnerable AngularJS expression.

The lab was successfully solved after the alert was triggered.
![[Pasted image 20261006212437.png]]

