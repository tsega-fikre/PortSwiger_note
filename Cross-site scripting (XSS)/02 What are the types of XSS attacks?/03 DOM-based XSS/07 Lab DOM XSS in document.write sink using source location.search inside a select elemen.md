![[Pasted image 20261006063801.png]]

# Step 
## Step 1
- I opened the application and choose one product and navigate to it and inside it i found `check stok` functionality
![[Pasted image 20261006065605.png]]
## Step 2
Then   I turn i pressed check functionality butten and see it return number value and from what i inspect there is select choose of the country.
![[Pasted image 20261006070740.png]]
## Step 3
And the request was when i see from developer tool Network tab and found also i intercept it see from burp and it send two param like in the developer tool 
![[Pasted image 20261006071351.png]]
![[Pasted image 20261006071618.png]]
## Step 4
so in order to preform the attack i have to make the location to be break out of the `<select>` and execute the payload that will allow me to do so i choose the payload:

```HTML
</select><img src=error onerro=alert('XSS Success')>
```
in order to excrete this i have to put the location par mater in URL and send it.
```
&storeId=</select><img+src=1+onerror=alert('XSS')>
```

![[Pasted image 20261006073624.png]]
![[Pasted image 20261006073637.png]]

- Note, that in some situations the content that is written to `document.write` includes some surrounding context that you need to take account of in your exploit. For example, you might need to close some existing elements before using your JavaScript payload.