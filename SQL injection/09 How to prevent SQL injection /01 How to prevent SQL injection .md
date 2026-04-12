
---

## The problem: String concatenation (the unsafe way)

Imagine you run a library. A visitor comes and says:  
> “Show me books in the **‘fiction’** category.”

You write down the request exactly as:  
`"SELECT books FROM library WHERE category = '" + visitor_says + "'"`

That becomes:  
`SELECT books FROM library WHERE category = 'fiction'` — fine.

But a bad visitor says:  
> `fiction' OR '1'='1`  

Now your note becomes:  
`SELECT books FROM library WHERE category = 'fiction' OR '1'='1'`  

That means “show me all books” — not just fiction. Worse, they could delete tables by saying:  
`fiction'; DROP TABLE books; --`

You trusted their words *inside* your command. **That’s the vulnerability.**

---

## The fix: Parameterized queries (the safe way)

Instead of glueing their words into your command, you say:  

> “I have a *fixed* command template, and your word is just a **value** — not part of the command structure.”

Template:  
`SELECT books FROM library WHERE category = ?`

The `?` is a placeholder. You then say:  
“Take whatever the visitor says, treat it *only* as a category name, and fill it into the `?`.”

Even if they say: `fiction' OR '1'='1`  
The database sees it as:  
> “Find me books whose category is exactly the string **`fiction' OR '1'='1`**”

It won’t run it as a command. It treats it like plain text — harmless.

---

## Why this works

- **Without parameters**: User input becomes part of the SQL *instructions*.  
- **With parameters**: User input becomes only *data*. Instructions stay fixed.

---

## In your example

**Vulnerable version:**  
`"SELECT * FROM products WHERE category = '"+ input + "'"`  
If `input = ` `' OR '1'='1`, the query changes structure.

**Fixed version:**  
`"SELECT * FROM products WHERE category = ?"`  
Then `statement.setString(1, input)`  
Even if `input` has SQL characters, they are treated as plain text, not code.

---

## Simple rule to remember

**Never build SQL queries by gluing strings together with user input.**  
Use **prepared statements / parameterized queries** everywhere you can.