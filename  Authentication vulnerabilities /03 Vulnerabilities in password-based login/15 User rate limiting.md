
# User Rate Limiting

## First, the big picture: What's the problem?

Imagine you have a secret password protecting your house. A burglar wants to get in. The dumbest way to do it? Stand at your door and try every key on a giant keychain, one after another, until one works. That's called a **brute-force attack** — just trying tons of passwords really fast until one finally works.

Websites need to stop this. One popular way is called **user rate limiting**.

---

## What is "user rate limiting"?

Think of a bouncer at a club. The bouncer watches how many times you knock on the door.

- Knock once or twice? Fine, you're a normal person.
- Knock 500 times in 10 seconds? The bouncer says, "You're clearly up to something. **You're blocked.** Go away."

That's rate limiting. If you (or your computer) send **too many login attempts in a short time**, the website blocks your **IP address** — which is basically your computer's mailing address on the internet.

So if you try 50 passwords in 30 seconds, the site says: "Nope. That IP is banned for now."

---

## How do you get un-blocked?

The website usually gives you three options:

1. **Wait it out** — After some time (say 15 minutes or an hour), the block lifts automatically. Like a timeout.
2. **Ask the admin** — A human administrator manually unblocks you. Slow, but possible.
3. **Solve a CAPTCHA** — You prove you're a human by clicking "I'm not a robot" or picking all the traffic lights. Then you're free again.

---

## Why do websites like this method?

There's another common defense called **account locking** — where if you fail to log in 5 times, the *account itself* gets locked. Sounds good, right?

But it has two big problems:

- **Username enumeration**: A hacker can figure out which usernames actually exist. If "john123" gets locked after 5 tries but "fakeuser999" doesn't, the hacker learns "john123" is a real account.
- **Denial of Service (DoS)**: A jerk can deliberately fail to log into *your* account 5 times, locking *you* out of your own account. Annoying!

Rate limiting avoids both of those because it's based on the **attacker's IP**, not the victim's account. That's why many sites prefer it.

---

## But wait — it's not perfect. Here's why.

### Problem 1: Hackers can hide their real IP

Remember, the block is based on your IP address — like your home address. But hackers can use tools (VPNs, proxies, Tor, etc.) to make it look like they're coming from a different address each time.

It's like a burglar who puts on a fake mustache, changes his hat, and knocks on your door from a different direction every time. The bouncer thinks, "Huh, different person each time — must be fine!" But it's the same guy.

So a hacker can just keep switching IPs and keep guessing passwords.

### Problem 2: Multiple guesses in one request

Here's the sneaky one. Rate limiting counts **how many requests** you send. But what if you could sneak **many password guesses inside a single request**?

Imagine the bouncer says, "You can only knock 3 times per minute." So instead of knocking 3 times, you knock once — but hand him a list of 100 names and say, "Hey, can you check if any of these people are on the guest list?"

If the website is poorly designed and checks multiple passwords in one request, the rate limiter never sees a "high rate." It only sees one request. The hacker just bypassed the whole defense.

---

## Summary in one breath

**User rate limiting** = "If you try to log in too many times too fast, we block your IP address. You can get unblocked by waiting, asking an admin, or solving a CAPTCHA. It's better than locking accounts because it avoids revealing which usernames are real and avoids letting trolls lock people out. **But** it's not bulletproof — hackers can fake their IP or cram many guesses into a single request to sneak past the limit."

---

## The takeaway

Rate limiting is like a speed bump, not a brick wall. It slows down lazy attackers, but a determined one can find ways around it. That's why good websites use **many layers** of defense at once — not just one.