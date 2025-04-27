---
title: "How I implemented sticky columns in tables using directives - Angular!"
datePublished: Sun Dec 12 2021 16:56:45 GMT+0000 (Coordinated Universal Time)
cuid: ckx3horai0islzvs1e6szhh8f
slug: implement-sticky-columns-using-directives-angular
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1639328151099/vugsl8iig.jpeg
tags: tutorial, web-development, angular, typescript

---

How to create sticky columns in Angular using directives. Implementing tables with sticky columns can be tricky especially when you have to make multiple columns sticky. Using directives, we can easily implement sticky columns.

I can't emphasize more the power of directives in Angular. I've written a couple of articles showcasing how one can actually use it to implement really cool stuff. You can check some use-cases for directives here:  [Angular Directive Showcase](https://ng-directives.vercel.app/).

![Sticky columns](https://cdn.hashnode.com/res/hashnode/image/upload/v1637340755156/2X26NikwO.gif)

## Tables with sticky columns
We make use of the `position: sticky` CSS property to make an element sticky. Read more about sticky positioning at  [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky_positioning).

```css
.sticky {
  position: sticky;
  left: 0;
}
``` 
For the sticky position to work properly, at least one of `top`, `right`, `bottom`, or `left` should be specified.

## The problem
Making the first column in a table sticky is super simple, you basically add the `sticky` class to the column.

When two columns need to stick to the left, we can't just add the `sticky` class to both of the columns. This is how it looks if you did so:

![Sticky columns](https://cdn.hashnode.com/res/hashnode/image/upload/v1637334396868/4_MzPb15p.gif)

Here you can see the **Manager** column overlapping with the **Company** column. This is because we gave both the columns `left:0`.

To make it work as expected, the styles should be like this:
```css
.company {
  position: sticky;
  left: 0px;
}

.manager {
  position: sticky;
  left: 120px; // <-- width of the company column
}
```
What we did here is we added the offset of the **Manager** column as the `left` property value.

## Sticky Calculations

For calculating the `left` value, we need to find the `x` value of the column. If we look at the first column **Company** and get its offset from the left side.

![Column bounding rect](https://cdn.hashnode.com/res/hashnode/image/upload/v1637337274353/UiOlAV_fBf.png)

We expect the `x` value to be `0` but we get `85` here. This is because the `x` value is calculated from the left side of the window to the column. For getting the left threshold of the column with respect to the table, we need to find the `x` value of the table. Once we get the table's offset, we can subtract it from the offset of the column.

![Offset Calculation](https://cdn.hashnode.com/res/hashnode/image/upload/v1637338173315/3ge9s1bZ2.png)

Example of the calculation:

- Position of Table = (100, 200) // <-- x = 100
- Position of Company = (100, 200) // <-- x with respect to table = 100 - 100 = 0
- Position of Manager = (300, 200) // <-- x with respect to table = 300 - 100 = 200


## Sticky Directive
We are going to create a directive to do just that. The directive can then be placed on the columns which need to be sticky. If you are thinking about why a directive for this particular use case, the calculation of the sticky thresholds can be done easily. Creating a directive makes it easy to re-use the functionality for different elements.

```ts
import { CommonModule } from '@angular/common';
import {
  AfterViewInit,
  Directive,
  ElementRef,
  NgModule,
  Optional,
} from '@angular/core';

@Directive({
  selector: '[stickyTable]',
})
export class StickyTableDirective {
  constructor(private el: ElementRef) {}

  get x() {
    return (this.el.nativeElement as HTMLElement)?.getBoundingClientRect()?.x;
  }
}

@Directive({
  selector: '[sticky]',
})
export class StickyDirective implements AfterViewInit {
  constructor(
    private el: ElementRef,
    @Optional() private table: StickyTableDirective
  ) {}

  ngAfterViewInit() {
    const el = this.el.nativeElement as HTMLElement;
    const { x } = el.getBoundingClientRect();
    el.style.position = 'sticky';
    el.style.left = this.table ? `${x - this.table.x}px` : '0px';
  }
}

@NgModule({
  declarations: [StickyDirective, StickyTableDirective],
  imports: [CommonModule],
  exports: [StickyDirective, StickyTableDirective],
})
export class StickyDirectiveModule {}
```
If you look at the code above, we have two directives:
1. StickyDirective
1. StickyTableDirective

The second directive is really interesting here. Why do we need a second directive?
We have a separate directive that can be placed on the table to get the offset of the table. The directive can then be injected inside the main `StickyDirective`.

```ts
  constructor(
    private el: ElementRef,
    @Optional() private table: StickyTableDirective
  ) {}
```

We mark the `StickyTableDirective` as `@Optional()` so that we can add the `StickyDirective` directive on other elements and it can automatically be sticky with the default value.

Ref: https://angular.io/guide/hierarchical-dependency-injection#optional

Here is how we use it in HTML.

```html
<table stickyTable>
  <tr>
    <th sticky>Company</th>
    <th sticky>Manager</th>
    <th>Employees</th>
    <th>Contractors</th>
    <th>Jobs</th>
    <th>Contracts</th>
    <th>Vacancy</th>
    <th>Offices</th>
  </tr>
  <ng-container *ngFor="let item of data">
    <tr>
      <td sticky style="min-width:200px">{{ item.company }}</td>
      <td sticky>{{ item?.manager }}</td>
      <td> {{ item?.employees }} </td>
      <td> {{ item?.contractors }} </td>
      <td> {{ item?.employees }} </td>
      <td> {{ item?.contractors }} </td>
      <td> {{ item?.employees }} </td>
      <td> {{ item?.contractors }} </td>
    </tr>
  </ng-container>
</table>
```
We add the `stickyTable` directive to the table and the `sticky` directive to the column.

## Demo and Code
<iframe src="https://stackblitz.com/edit/angular-ivy-mcwnrf?embed=1&file=src/app/app.component.html&hideExplorer=1&hideNavigation=1&view=preview" width="100%" height="600px"></iframe>

 [Stackblitz Link](https://stackblitz.com/edit/angular-ivy-mcwnrf?file=src/app/sticky.directive.ts)

## Improvements
A lot of improvements can be made to this directive to make it more re-usable like: 
- Add support for other directions.
- Generalize the `StickyTableDirective` to be able to use it on other elements as well.

For the sake of keeping the example here simple, I've kept it simple.

## Similar Reads
1.  [Implement Heatmaps in a table using directives](https://blog.sreyaj.dev/how-to-implement-heatmap-in-tables-using-directives-in-angular) 
2.  [Highlight text in paragraphs with a simple directive in Angular](https://blog.sreyaj.dev/highlight-text-in-angular-using-directives)
3.  [Fullscreen toggle functionality in Angular using Directives.](https://blog.sreyaj.dev/fullscreen-toggle-angular-using-directives)  


## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cardify](https://cardify.adi.so) - Dynamic SVG Images for Github Readmes


Do add your thoughts in the comments section.
Stay Safe ❤️

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png)](https://www.buymeacoffee.com/adisreyaj)