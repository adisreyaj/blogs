## How to enable Tailwind JIT compilation mode in your Angular project.

Tailwind, for those living under a rock, is all the buzz among web devs. It is a utility first CSS framework wherein we add a bunch of classes to HTML elements and the rest is taken care of by Tailwind. 

```html
<div class="w-full flex-none text-sm font-medium text-gray-500 mt-2">
   Test
 </div>
```

This is how your template would look like. I mean not all are a fan of this approach, but for those who like this kind of utility-class approach, Tailwind is the best out there.

I personally have been using it extensively for all my projects, and I totally love it. You will be writing fewer custom styles, which is a relief. 

## Tailwind - Getting to know it

Writing custom styles was totally my thing until I started using  [Tailwind](https://tailwindcss.com/). I was not a fan of bootstrap and always made sure to write plain old CSS for most of the projects. You won't be able to find the real advantage of using utility-first frameworks like these at first or in smaller applications.
The real benefit comes when the application grows and the styles become very difficult to manage. Over time the styles bundle would grow drastically, especially if you are working in a team with a lot of other devs.

If you are using Tailwind, you would be shipping fewer styles as everything which is not used will be purged, meaning only those classes that you have used will be part of the bundle. This will shave a huge chunk from the styles bundle.

## Angular and Tailwind

People really started liking Tailwind and using it with other frameworks was so easy. But in the case of Angular, it was a different story. 
Tailwind is a  [PostCSS](https://postcss.org/)  plugin, so to make it work, we need access to the PostCSS configuration where we can specify tailwind in the plugins list.
Even though Angular uses PostCSS, it didn't expose the bundler config file making it difficult to use Tailwind in Angular.

The only way is to eject the webpack config and then manually configure the bundler to utilize Tailwind. It is not something an average developer would do and is not recommended either.

## Angular Tailwind Schematics

Ngneat came up with a super cool schematic for Angular ( [NgNeat/Tailwind](https://github.com/ngneat/tailwind) ), which would basically do everything and configure all the stuff needed to make Tailwind work with Angular. And it is so damn straightforward.

The schematics can be accessed via:
```sh
ng add @ngneat/tailwind
```
## Angular Official Support for Tailwind
Seeing the hype and the number of user requests, the Angular team was quick to release a version of Angular (`v11.2`) which comes with support for Tailwind. If the CLI detects a tailwind configuration in your project, PostCSS will automatically use the tailwind plugin with this config.

Before Tailwind came up with the JIT compiler, the compiling takes a lot of time. If you have a lot of variants configured in the tailwind config, the styles bundle spitted out in dev mode is too big that it lags the developer console while inspecting elements. 
Purge was recommended to be enabled during production build, otherwise compiling would take even more time.

Here's what you have to do:
### Prerequisite
Make sure the Angular CLI version is >= 11.2

### Setting up Tailwind JIT

Tailwind JIT is release in `v2.1` so make sure you are installing the latest version

1. Install tailwind as a dependency
```sh
npm i tailwindcss
```
2. Create the tailwind config file
```sh
npx tailwindcss init
```
3. Add these base styles to `styles.scss` file:
```scss
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
```
4. Edit the `tailwind.config.js` file to enable `jit` compilation.
5. Update the purge array with the path to your components. If purge is not configured, JIT will not simple work.
```js
module.exports = {
  mode: "jit",
  purge: ["./src/app/**/*.{ts,html}"],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

### Running the project
For the Tailwind JIT compiler to detect changes to your files, you need to set the `NODE_ENV` to `development`. Read More Here: https://tailwindcss.com/docs/just-in-time-mode#watch-mode-and-one-off-builds

You can do that by simply setting the env variable before your serve and build scripts.
I am using `cross-env` so that we avoid the platform-related shenanigans when setting environment variables.

```sh
npm i -D cross-env
```

Update the scripts in `package.json`:
```json
"scripts": {
    "start": "cross-env NODE_ENV=development ng serve",
    "build": "cross-env NODE_ENV=production ng build",
  },
```
We're done! Enjoy super-fast reload times when using Tailwind. No more laggy developer consoles, when inspecting elements on your browser.

Ref: https://tailwindcss.com/docs/just-in-time-mode

## 🌎 Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cartella](https://cartella.sreyaj.dev) - building at the moment

%%[buymeacoffeebutton]

Do add your thoughts in the comments section.
Stay Safe ❤️




