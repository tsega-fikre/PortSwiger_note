Imagine you run a small store. You let customers bring their own boxes to put things in, but you’re worried someone might bring a box full of dangerous stuff (like a bomb or a wild animal).  

So you try to stop them by saying:  
- “No boxes labeled ‘bomb’ or ‘weapon’ allowed.”  
But someone labels their box “photo.jpg” even though inside is actually a bomb. You only look at the label, not what’s really inside.  
- You also forget to ban boxes labeled “.php” or “.exe” — so someone uses that and sneaks in a dangerous file.  

Sometimes you try to check what’s inside, but you just look at the first few items in the box. A clever person arranges the box so the first thing you see is safe (a toy), but underneath is something dangerous.  

Worse, you have two entrances to your store. At one entrance, you check boxes carefully. At the other entrance, you don’t check at all. So the attacker just uses the second door.  

That’s how **file upload vulnerabilities** happen:  
- The website tries to block dangerous files, but its **checks are easy to trick** (checking only extensions, not real content).  
- It uses **blacklists** that miss some risky file types.  
- It doesn’t check **consistently** across all places where files can be uploaded.  

Result: an attacker uploads a file that looks safe but isn’t — and once it’s on the server, they can take control of the website.