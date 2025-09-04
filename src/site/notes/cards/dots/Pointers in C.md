---
{"dg-publish":true,"permalink":"/cards/dots/pointers-in-c/"}
---

~ [[atlas/red-team\|red-team]] | [[atlas/windows\|windows]]

How memory works:

![Pointers in C.png](/img/user/cards/dots/images/Pointers%20in%20C.png)

`int x = 4;`
Say in a programming language we assigned a int variable **x** with the value of 4 then in memory **0x1000** will hold that value, as shown the image.

So **0x1000** will hold a value of 4. that's it.

`int * pX = &x;`

"integer pointer named pX is set to the address of x"

`int *` when an asterisk is placed near a type (in this case int) it becomes a pointer.

`pX` is the name of the variable that holds the pointer to variable `x` from previous one, but we can name whatever we want but for best practices it should be:

**p for pointer and the variable name**

`&x`

& = address of

in this case "address of x"

**Why this? in this way we can access 'x' by reference and not by value**

`int y = * pX;`

"integer named y is set to the thing pointed to by  pX".

`* pX`

If an asterisk is not near any type (or datatype) it is has now a new function - it is now called a 'd reference'.

Basically its function is "go to the address pointed to by the pointer and grab that value"

**Now that we know how to pass 'X' as reference instead of value? lets know why do we even need to do this**

To keep things clean and scalabe.

Here is an example, the **struct Person** is not accessible based on the snippet provided, so to get around this, we passed the the **Person** as reference instead and now the **Person** can now be edited:

![Pointers in C-1.png|450](/img/user/cards/dots/images/Pointers%20in%20C-1.png)
