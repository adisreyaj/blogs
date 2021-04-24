## Fullscreen toggle functionality in Angular using Directives.

We are gonna see how to leverage directives to create a super simple fullscreen toggle functionality.

Let's see the definition for a directive in Angular:

> 
Directives are classes that add additional behavior to elements in your Angular applications. With Angular's built-in directives, you can manage forms, lists, styles, and what users see.

As the definition says, we can use it to add additional behavior to elements. If it is not cool, I don't know what else is. 

## Directives in Angular
For people often starting with Angular, the term directive and how it works might be confusing. And most of them would simply be stopping at using some of the most common directives like 
- NgClass
- NgStyle
- NgIf
- NgFor 

We don't really have to care about how adding this would magically make things happen. We might know that to conditionally render an element, we can use `*ngIf` and only renders the element if the case is true. That is the end of the story!

One piece of advice that I would like to give to those who are getting the hang of Angular is that try to dig into these features and try to find the underlying implementation.

Angular has really good documentation on different types of directives and how to use them. Read Here:
https://angular.io/guide/built-in-directives#built-in-directives

**Tip**: To see how the built-in directives are written, just visit their API documentation page and click on the `code` icon which will take you to the direct file in the repo:

![Angular documentation](https://cdn.hashnode.com/res/hashnode/image/upload/v1619263768936/dG6zFXvDY.png)

## Creating a Fullscreen Directive

![Fullscreen directive in Angular](https://cdn.hashnode.com/res/hashnode/image/upload/v1619269013931/JfNFXQXmu.gif)

So today we are going to build a custom directive that is gonna help us with one single functionality - maximize and minimize an element.

Enough talking, let's get right into it.

### 1. Create a module for our directive

This is actually optional. But I do like to create a module and then the directive will be declared and exported from this module. Then I would import this module wherever I need this functionality.

```ts
import { NgModule } from "@angular/core";
import { MaximizeDirective } from "./maxmimize.directive";

@NgModule({
  declarations: [MaximizeDirective],
  exports: [MaximizeDirective] //<-- Make sure to export it
})
export class MaximizeModule {}
```

### 2. Create the directive

Next up we create the directive. I am calling it the `maximize` directive. Once the directive is created let us add the logic to it.

The idea is to basically make the width and height of the element (where the directive is placed) to say `100vw` and `100vh` so that it spans the entire screen real estate. You might also want to change the positioning of the element in some cases.

```ts
import { Directive, ElementRef, Renderer2 } from "@angular/core";
import { BehaviorSubject } from "rxjs";
import { tap } from "rxjs/operators";
@Directive({
  selector: "[maximize]",
  exportAs: "maximize" // <-- Make not of this here
})
export class MaximizeDirective {
  private isMaximizedSubject = new BehaviorSubject(false);
  isMaximized$ = this.isMaximizedSubject.pipe();
  constructor(private el: ElementRef, private renderer: Renderer2) {}

  toggle() {
    this.isMaximizedSubject?.getValue() ? this.minimize() : this.maximize();
  }
  maximize() {
    if (this.el) {
      this.isMaximizedSubject.next(true);
      this.renderer.addClass(this.el.nativeElement, "fullscreen");
    }
  }
  minimize() {
    if (this.el) {
      this.isMaximizedSubject.next(false);
      this.renderer.removeClass(this.el.nativeElement, "fullscreen");
    }
  }
}

```

Let me break down the code for you guys.

First thing is that we need to get hold of the element on which the directive is applied. We do that by injecting `ElementRef` in the constructor. This would give reference to the element.

Next, we keep a subject to maintain the state of the directive (to know whether we are currently minimized or maximized).

#### Modifying the element property
So I have a CSS class defined which just modifies the height and width:
```css
.fullscreen {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
```

The idea now is to toggle this class when the user wants it to be in fullscreen mode or not. For that, we get hold of `Renderer2`([Doc](https://angular.io/api/core/Renderer2)) which is a great way to manipulate DOM elements in Angular.

So write two functions one for adding the class and one for removing. We update the state when these functions are called.

Another function called `toggle` is added so that the user can perform the operation without having to know about the current state.

## Exporting the directive instance
In the section above, I have marked a particular line of code:
```ts
@Directive({
  selector: "[maximize]",
  exportAs: "maximize" // <-- Make not of this here
})
```
This is a really interesting feature that angular provides. In the directive, we can specify whether we want to make the instance available in the template.

Read here: https://angular.io/api/core/Directive#exportas

What it does is that the directive instance will be made available in the template using the specified name:
```html
<div class="card" maximize #maximize="maximize">
     <p>{{(maximize.isMaximized$ | async)}}</p>
 </div>
```
See how I access the property `isMaximized$` of the directive in the template. Similarly, all the public properties of the directive can be accessed using `maximize`.

## Usage
For using the directive, first I import the module in my `AppModule`. If you have already declared the directive in your `AppModule`, you can skip this step.

```ts
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";

import { AppComponent } from "./app.component";
import { MaximizeModule } from "./maximize/maximize.module";
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, MaximizeModule], // <-- Imported the module
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

In your template, you can now add the directive to any element
```html
 <div class="card" maximize #maximize="maximize">
      <button class="min-button"
          (click)="maximize.minimize()">
      </button>
      <button class="max-button"
          (click)="maximize.maximize()">
      </button>
 </div>
```
So like we've seen in the section where we talked about getting the instance, we can get hold of the `minimize()` and `maximize()` function in the directive to apply/remove the class respectively.

## Bonus - Go Fullscreen in the browser too
So what we did would just maximize the element, but you would also want to make the browser go fullscreen, we can add that too.

1. Install the `screenfull` library
```sh
npm i screenfull
```
2. Update the directive like so:
```ts
...
import * as Fullscreen from "screenfull"; // <-- add the import
...
maximize() {
    if (this.el) {
      this.isMaximizedSubject.next(true);
      this.renderer.addClass(this.el.nativeElement, "fullscreen");
      if (Fullscreen.isEnabled) { 
        Fullscreen.request(); // <-- request fullscreen mode
      }
    }
  }
  minimize() {
    if (this.el) {
      this.isMaximizedSubject.next(false);
      this.renderer.removeClass(this.el.nativeElement, "fullscreen");
      if (Fullscreen.isEnabled) {
        Fullscreen.exit(); // <-- exit fullscreen mode
      }
    }
  }
```
**Note**: You might want to open the sandbox preview in a separate window to get the fullscreen working.

## Code
<iframe src="https://codesandbox.io/embed/ng-maximize-x6epr?fontsize=14&hidenavigation=1&theme=dark&view=preview"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="ng-maximize"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
allowfullscreen webkitallowfullscreen mozallowfullscreen
   ></iframe>

## 🌎 Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cartella](https://cartella.sreyaj.dev) - building at the moment

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1618661389599/2B667-okT.png)](https://www.buymeacoffee.com/adisreyaj)

Do add your thoughts in the comments section.
Stay Safe ❤️