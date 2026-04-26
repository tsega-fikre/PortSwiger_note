# Finding hidden attack surface for SSRF vulnerabilities


---

## First, what is SSRF in plain English?

Imagine you have a **butler** (the server).  
You can give the butler a note saying: *“Go fetch me something from this address.”*

Normally, the butler only fetches things from inside the house (internal services).  
But if you trick the butler into going somewhere dangerous — like the neighbor's locked basement or the bank vault — that’s SSRF.

In web terms:  
You make the **server** visit a place *you* choose, instead of where it intended to go.

---

## The easy-to-find SSRF (obvious)

Sometimes the app **directly asks you** for a full address.  
Example:  
> “Enter the URL of your profile picture: ___________”

You type: `https://example.com/my-pic.jpg`  
The server goes and gets it.

An attacker thinks: *“What if I type a secret internal address instead?”*  
They type: `http://192.168.1.1/admin` or `http://169.254.169.254/secret`

That’s easy SSRF. The app basically hands you the keys.

---

## The hard-to-find SSRF (hidden)

Now imagine the app **never asks you for a full address**.  
It asks for something else. But that “something else” secretly turns into an address later.

Let me give you real-world-like examples, but simple.

---

### Example 1: The “image upload” trick

The app says: *“Upload your profile picture (JPG or PNG only).”*  
No URL field anywhere. Seems safe, right?

But what if you upload a **special picture** that contains a hidden instruction?  
Some picture formats (like SVG) can say: *“Hey, also go fetch this other image from http://secret-place/”*

The server, trying to be helpful, follows that instruction and visits the secret address.  
**You never typed a URL.** You just uploaded a file, and that file contained a hidden URL.

---

### Example 2: The “username” trick

The app says: *“Choose a username: _______”*  
You choose: `Bob`

Later, the server wants to show your avatar. It builds an address like:  
`http://avatars.com/Bob.jpg`

Now imagine you choose a sneaky username:  
`Bob@evil.com`

Now the server builds:  
`http://avatars.com/Bob@evil.com.jpg`

Depending on how the server handles this, it might actually visit `evil.com` instead of `avatars.com`.  
You never gave a full URL. You just gave a weird username, and the server *constructed* a dangerous URL.

---

### Example 3: The “email notification” trick

The app says: *“Enter your email to get a report: _______”*  
You enter: `me@gmail.com`

Later, the server wants to send you a link. It might build an internal address like:  
`http://notification-server/send?to=me@gmail.com`

Now imagine you enter:  
`me@169.254.169.254`

The server builds:  
`http://notification-server/send?to=me@169.254.169.254`

If the notification server is badly coded, it might try to visit `169.254.169.254` (a secret cloud metadata address).  
Again, you never typed a URL — just an email address.

---

### Example 4: The “import from cloud” trick

The app says: *“Import your file from Google Drive: enter file ID: _______”*  
You enter: `abc123`

The server builds:  
`https://drive.google.com/file/abc123`

But what if the server also supports other “providers” behind the scenes?  
Maybe it also understands:  
`http://169.254.169.254/secret` as a “file ID” if you format it specially.  
The programmer didn’t expect anyone to try that.

---

## Why are these harder to find?

Because as a tester, you’re not looking for a box that says **“Enter URL”**.  
You’re looking for **any box that the server later uses to build an address** — even indirectly.

That could be:
- A username
- A filename
- An email address
- A file upload
- A “callback URL” hidden in settings
- A field that looks like an ID but is actually a path

---

## The detective mindset for hidden SSRF

Ask yourself for **every** input field:

> “Could the server ever turn this into a web address?”

If yes, try putting in:
- `http://169.254.169.254/` (cloud metadata)
- `http://localhost/admin`
- `http://your-own-server.com/ssrf-test`
- `file:///etc/passwd` (read local files)

And watch what happens.

Even if the server doesn’t show you the result, you can sometimes tell by:
- **Time delays** (slow = maybe it tried and failed)
- **Error messages** (“could not connect to 192.168.1.1”)
- **Your own server getting a visit** (use a service like webhook.site or Burp Collaborator)

---

## Summary analogy

| Easy SSRF | Hard SSRF |
|-----------|------------|
| App asks: “What address should I visit?” | App asks: “What’s your username?” |
| You say: “evil.com” | You say: “Bob” |
| Server visits evil.com immediately | Server later builds “avatars.com/Bob.jpg” — but if Bob = “evil.com”, maybe it visits evil.com instead |

Hidden SSRF is about **indirect control**. You don’t give a full address. You give a piece of data that *becomes* part of an address later.

---

## One last simple check

Whenever you see an app doing something *with your input* that might involve fetching —  
- generating a thumbnail  
- sending a webhook  
- importing a file  
- checking if a link is safe  
- resolving a short URL  
- looking up a user by email or ID  

…try to sneak in an address. Even if the field says “not a URL”, test it like one. That’s how hidden SSRF is found.