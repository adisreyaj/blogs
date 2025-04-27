---
title: "Highlight text in paragraphs with a simple directive in Angular"
seoTitle: "Highlight text in paragraphs with a simple directive in Angular"
seoDescription: "How to highlight text in a paragraph with the help of directives in Angular. Especially helpful in highlighting text matching the search term."
datePublished: Tue Jun 01 2021 04:23:53 GMT+0000 (Coordinated Universal Time)
cuid: ckpdjdak40afub1s175o9abf5
slug: highlight-text-in-angular-using-directives
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1622365240991/9ekX7bfFn.jpeg
tags: javascript, angularjs, web-development, angular-2, typescript

---

How to highlight text in a paragraph with the help of directives in Angular. Especially helpful in highlighting text matching the search term. You could have come across this in your browser or IDE when you search for something, the matching items will be highlighted to point you to the exact place of occurrence.

## Text Highlighting

Here is what we are going to build in this post. A very simple and straightforward highlight directive in Angular. We see something similar in chrome dev tools.

The idea is pretty simple. We just have to match the searched term and somehow wrap the matched text in a `span` or `mark` ( [ref](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/mark) ) tag so that we can style them later according to our needs.

## How to highlight matched text?

We are going to use `Regex` to find matches in our paragraph. Regex makes it very simple to do operations like this on strings. The directive should be ideally added only to elements with text in it.
This is what we are building:

![Highlight text directive](https://cdn.hashnode.com/res/hashnode/image/upload/v1622313456863/JGswufIHm.png)

So let's plan out our directive.
The main input to the directive is the term that needs to be highlighted. So yeah, we will use `@Input()` to pass the term to our directive. I think that is pretty much what we need inside the directive

So now we need to get hold of the actual paragraph to search in. So there is an easy way to get the text from an `HTMLElement`. We can use the `textContent`( [ref](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) ) which should give us the text to search in.

## Building the Highlight directive
As always, I would recommend you create a new module only for the directive. And If you really properly manage your codebase, you can consider creating it as a library within the project as well.

To keep things simple, we put our code in a `lib` folder:
```md
lib/
├── highlight/
│   ├── highlight.module.ts
│   ├── highlight.directive.ts
```
### Highlight Module
This module would be simple declaring our directive and exporting it. Nothing much is needed here.
```ts
import { CommonModule } from "@angular/common";
import { NgModule } from "@angular/core";
import { HighlightDirective } from "./highligh.directive";

@NgModule({
  declarations: [HighlightDirective],
  imports: [CommonModule],
  exports: [HighlightDirective]
})
export class HighlightModule {}
```
### Highlight Directive
Now that our setup is complete, we can start creating our directive where all our magic is going to happen.

```ts
import {
  Directive,
  ElementRef,
  HostBinding,
  Input,
  OnChanges,
  SecurityContext,
  SimpleChanges
} from "@angular/core";
import { DomSanitizer, SafeHtml } from "@angular/platform-browser";

@Directive({
  selector: "[highlight]"
})
export class HighlightDirective implements OnChanges {
  @Input("highlight") searchTerm: string;
  @Input() caseSensitive = false;
  @Input() customClasses = "";

  @HostBinding("innerHtml")
  content: string;
  constructor(private el: ElementRef, private sanitizer: DomSanitizer) {}

  ngOnChanges(changes: SimpleChanges) {
    if (this.el?.nativeElement) {
      if ("searchTerm" in changes || "caseSensitive" in changes) {
        const text = (this.el.nativeElement as HTMLElement).textContent;
        if (this.searchTerm === "") {
          this.content = text;
        } else {
          const regex = new RegExp(
            this.searchTerm,
            this.caseSensitive ? "g" : "gi"
          );
          const newText = text.replace(regex, (match: string) => {
            return `<mark class="highlight ${this.customClasses}">${match}</mark>`;
          });
          const sanitzed = this.sanitizer.sanitize(
            SecurityContext.HTML,
            newText
          );
          this.content = sanitzed;
        }
      }
    }
  }
}

```

Let's do a code breakdown.

The first thing that we need is the Inputs in our directive. We only actually need the search term, but I have added some extra functionalities to our directive. We can provide `customClasses` for the highlighted text, and another flag 'caseSensitive` which will decide whether we have to match the case.

```ts
@Input("highlight") searchTerm: string;
@Input() caseSensitive = false;
@Input() customClasses = "";
```

Next up we add a `HostBinding` ( [ref](https://angular.io/api/core/HostBinding) ) which can be used to add value to a property on the host element.

```
@HostBinding("innerHtml")
 content: string;
```
We bind to the `innerHtml` ( [ref](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) ) property of the host element. We can also do it in this way:
```ts
this.el.nativeElement.innerHtml = 'some text';
```

To get access to the host element, we inject `ElementRef` in the constructor, and also since we are going to be playing around with direct HTML manipulation, I have also injected `DomSanitizer` ( [ref](https://angular.io/api/platform-browser/DomSanitizer) ) to sanitize the HTML before we inject it into the element.

So now we move on to the actual logic which we can write in the `ngOnChanges` ( [ref](https://angular.io/api/core/OnChanges) ) lifecycle hook. So when our search term changes, we can update the highlights. The interesting part is:
```ts
const regex = new RegExp(this.searchTerm,this.caseSensitive ? "g" : "gi");
const newText = text.replace(regex, (match: string) => {
     return `<mark class="highlight ${this.customClasses}">${match}</mark>`;
});
const sanitzed = this.sanitizer.sanitize(
    SecurityContext.HTML,
    newText
);
this.content = sanitzed;
```

First, we set up the regex to help us find the matches. based on the `caseSensitive` condition we just add different Regex Flags:
- g - search for all matches.
- gi -search for all matches while ignoring case.

We just wrap the matches with `mark` tag using the `replace` ( [ref](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace) ) method on the string.
```ts
const newText = text.replace(regex, (match: string) => {
     return `<mark class="highlight ${this.customClasses}">${match}</mark>`;
});
```

After that the newText, which is a HTML string needs to be sanitized before we can bind it to the innerHTML. We use the `sanitize` ( [ref](https://angular.io/api/platform-browser/DomSanitizer#sanitize) ) method on the `DomSanitizer` class:
```ts
const sanitzed = this.sanitizer.sanitize(
    SecurityContext.HTML,
    newText
);
```

Now we just assign the sanitized value to our `content` property which gets added to the `innerHTML` via `HostBinding`.

## Usage
This is how we can use it in our component. Make sure to import our `HighlightModule` to make our directive available for use in the component.
```html
<p [highlight]="searchTerm" [caseSensitive]="true" customClasses="my-highlight-class">
      Lorem Ipsum has been the industry's standard dummy text ever since the
      1500s, when an unknown printer took a galley of type and scrambled it to
      make a type specimen book.
</p>
```

That's all! We've successfully created a very simple text highlighter in Angular using directives. As always, please don't directly reuse the code above, try to optimize it and you can always add or remove features to it.

## Demo and Code
<iframe src="https://codesandbox.io/embed/ng-highlight-11hii?autoresize=1&fontsize=14&hidenavigation=1&module=%2Fsrc%2Fapp%2Flib%2Fhighlight%2Fhighligh.directive.ts&theme=dark&view=preview"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="ng-highlight"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

CodeSandbox: https://codesandbox.io/s/ng-highlight-11hii

## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cartella](https://cartella.sreyaj.dev) - building at the moment

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1618661389599/2B667-okT.png)](https://www.buymeacoffee.com/adisreyaj)

Do add your thoughts in the comments section.
Stay Safe ❤️