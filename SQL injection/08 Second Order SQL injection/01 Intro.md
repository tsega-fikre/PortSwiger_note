Let’s imagine you’re filling out a form on a website.  

**Step 1 – You give some tricky input**  
You type something like `Robert' ; DROP TABLE Students; --` into a box that asks for your name.  

The website takes that and *safely* puts it into its database. It does not mess up anything right now — your input is stored exactly as you typed it, without causing trouble.  

**Step 2 – Later, the website uses your input**  
A few days later, the website runs a report that says “Show me all info for user named Robert' ; DROP TABLE Students; --”. But instead of treating your input as plain text, the website accidentally *runs it as a SQL command*.  

That means:  
- It looks for a user named `Robert'`  
- Then it runs `DROP TABLE Students` (which deletes an important table)  
- The rest is ignored.

**So the attack worked, even though the first time you gave the input, nothing bad happened.**  

---

**Why is it called "second-order"?**  
Because the harm happens not when you first give the bad data, but *second* — when the stored data is used again later.  

**Why do developers mess this up?**  
They think: “We put that data into the database safely, so it must be safe to use.”  
But being *stored safely* doesn’t mean being *used safely*.