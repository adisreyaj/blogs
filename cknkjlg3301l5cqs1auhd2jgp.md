## Theming your Angular apps using CSS Variables - Easy Solution!

Have you thought about how you can provide your users with different color schemes for your application? or Do you want your application to have a `Dark` theme?

Who doesn't like change? Even on our phones, sometimes we get bored of the look and feel, that we try out some new themes.

## To provide multiple themes or not?

![Theming in Angular](https://cdn.hashnode.com/res/hashnode/image/upload/v1618589961856/St6ne1P5x.png)

Sometimes it is always good to stick with the brand colors, this is especially the way to go for products that cater to consumers directly. Then there are apps that cater to other businesses, in this kind of application, it is good to have the option to customize the look and feel of the application for different businesses.

So the answer would be it depends on a lot of things. To name a few:
- who are audience
- does it bring in value

One really good example for when theming your application would make sense is for a School Management Software. Say the application is used by Teachers, Students, and Parents. We can give a different theme to the application based on the role.

Another good fit for providing custom themes would be applications that can be white-labeled. So for each of the users/businesses, they can set up their own themes to match their brand colors.

Even if you are going to provide multiple color themes for the application, it's probably a good idea to come with a `Dark Mode` for the application. More and more products are now supporting `Dark` theme.

## CSS Custom Properties

The easiest way to theme your application is using `CSS Variables` / `CSS custom properties`. It makes theming so much easier than before when we had to do a lot of stuff, just to change some colors here and there.

But with [CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) is so damn easy.

CSS Preprocessors had the concept of variables for a long time and CSS was still lagging behind on it until a few years back. Now its is supported in all modern browsers.

![CSS Variable - Browser support](https://cdn.hashnode.com/res/hashnode/image/upload/v1618589225549/YdcP-awk8.png)

One really interesting thing about CSS custom properties is that they can be manipulated from JavaScript.

1. Declare variables
```css
--primaryColor: red;
--primaryFont: 'Poppins';
--primaryShadow: 0 100px 80px rgba(0, 0, 0, 0.07);
```
2. Usage
```css
button {
      background: var(--primaryColor);
      font-family: var(--primaryFont);
      box-shadow: var(--primaryShadow);
}
```

This is the most basic thing you should be knowing about CSS Custom Properties.

## Theming in Angular

Starting off with a brand new Angular project with CLI v11.2.9. We now start by declaring some CSS variables for our application.

In the `styles.scss` file:
```scss
:root {
  --primaryColor: hsl(185, 57%, 35%);
  --secondaryColor: hsl(0, 0%, 22%);
  --textOnPrimary: hsl(0, 0%, 100%);
  --textOnSecondary: hsl(0, 0%, 90%);
  --background: hsl(0, 0%, 100%);
}
```

We have declared a few variables and assigned the default colors for all of them. One thing to note, when you are going to be providing different colors is that ** you name the variables in a generic way **. You shouldn't be naming it with the color's name.

### Setting up the themes

I will be creating a `theme.config.ts` file all the themes will be configured. You can always make a static config like this or maybe get the config from an API response.
The latter is the better approach if you make changes to your themes often.

```ts
export const THEMES = {
  default: {
    primaryColor: 'hsl(185, 57%, 35%)',
    secondaryColor: 'hsl(0, 0%, 22%)',
    textOnPrimary: 'hsl(0, 0%, 100%)',
    textOnSecondary: 'hsl(0, 0%, 90%)',
    background: 'hsl(0, 0%, 100%)',
  },
  dark: {
    primaryColor: 'hsl(168deg 100% 29%)',
    secondaryColor: 'hsl(161deg 94% 13%)',
    textOnPrimary: 'hsl(0, 0%, 100%)',
    textOnSecondary: 'hsl(0, 0%, 100%)',
    background: 'hsl(0, 0%, 10%)',
  },
  netflix: {
    primaryColor: 'hsl(357, 92%, 47%)',
    secondaryColor: 'hsl(0, 0%, 8%)',
    textOnPrimary: 'hsl(0, 0%, 100%)',
    textOnSecondary: 'hsl(0, 0%, 100%)',
    background: 'hsl(0deg 0% 33%)',
  },
  spotify: {
    primaryColor: 'hsl(132, 65%, 55%)',
    secondaryColor: 'hsl(0, 0%, 0%)',
    textOnPrimary: 'hsl(229, 61%, 42%)',
    textOnSecondary: 'hsl(0, 0%, 100%)',
    background: 'hsl(0, 0%, 100%)',
  },
};

```

This is the most basic way to do this. Maybe in the future, we can talk about how we can create a theme customizer where the user can completely change the look and feel of the application.

### Theming service

We create a service and call it `ThemeService`. The logic for updating the themes will be handled by this service. We can inject the service into the application and then change the theme using a function we expose from the service.

```ts
import { DOCUMENT } from '@angular/common';
import { Inject, Injectable } from '@angular/core';
import { THEMES } from '../config/theme.config';

@Injectable({
  providedIn: 'root',
})
export class ThemeService {
  constructor(@Inject(DOCUMENT) private document: Document) {}

  setTheme(name = 'default') {
    const theme = THEMES[name];
    Object.keys(theme).forEach((key) => {
      this.document.documentElement.style.setProperty(`--${key}`, theme[key]);
    });
  }
}
```
The service is very straightforward. There is a single function that we expose which changes the theme. How this works is basically by overriding the CSS variable values that we have defined in the `styles.scss` file.

We need to get access to the `document`, so we inject the `Document` token in the constructor.

The function takes the name of the theme to apply. What it does it, get the theme variables for the selected theme from our config file and then loop through it wherein we apply the new values to the variables.

Done!

This is one approach to how you can do it. There are a lot of other ways to do the same thing, and in the future, I'll try to write about some other ways to Theme your Angular applications. One other approach as mentioned by @[Dharmen Shah](@shhdharmen) in the comments section is to define all the variables for each theme in their classes and then just change the class appended to the body tag.

## Code and Demo

Click on the buttons to change the themes.

<iframe src="https://codesandbox.io/embed/ng-theming-px9dg?autoresize=1&fontsize=14&hidenavigation=1&theme=dark&view=preview"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="ng-theming"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

[![Repo](https://img.shields.io/badge/Github-View%20Repo-lightgrey?style=for-the-badge&logo=github)](https://github.com/adisreyaj/ng-theming)


Feel free to ping me on Twitter If you would like to connect. Stay Safe ❤️

Do add your thoughts in the comments section.