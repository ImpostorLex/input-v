---
{"dg-publish":true,"permalink":"/cards/web/80-of-ui-design-is-typography/"}
---

[[cards/web/images/UI Designing\|UI Designing]]

To make most important element stands out:

![UI Designing.png](/img/user/cards/web/images/UI%20Designing.png)
The picture on the left user interface can be thought of as one group while the one on the right can be thought of two groups:

- Size
- Space
- Font Color (modify the lightness of a font, see _HSL_)
- Font Weight (shape)

Have a look at below:

![UI Designing-1.png](/img/user/cards/web/images/UI%20Designing-1.png)

- **Hue** is the base color we want such as red, green, blue, and more.
- **Saturation** modifies the intensity of the color

![UI Designing-2.png](/img/user/cards/web/images/UI%20Designing-2.png)
- **Lightness** controls the brightness

```
color: hsl(50, 100%, 60%);
```

Sample improved UI:

![UI Designing-3.png|450](/img/user/cards/web/images/UI%20Designing-3.png)

**Note:** user feedback is important here.

The three highlighted typescale is enough combined with the above process.

![UI Designing-4.png|450](/img/user/cards/web/images/UI%20Designing-4.png)

Here is an example:

![UI Designing-5.png](/img/user/cards/web/images/UI%20Designing-5.png)

1. Choose a base size between 14 and 16 px alongside regular weight and 100% lightness.
2. Then try designing everything with that size 
	- If needed to go above or below 2 pixels 
3. To make it easier, create them as variables:

![UI Designing-6.png](/img/user/cards/web/images/UI%20Designing-6.png)
**Note:** It is important to convert px to REM and `h1` should be used on the user will focus on

![UI Designing-7.png](/img/user/cards/web/images/UI%20Designing-7.png)

Converting light mode to dark mode, all you need is `hsl`'s lightness value:

![UI Designing-9.png](/img/user/cards/web/images/UI%20Designing-9.png)https://www.youtube.com/watch?v=9-oefwZ6Z74

https://www.iamsajid.com/colors/