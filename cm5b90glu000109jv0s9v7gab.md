---
title: "Figma like input field in Angular using Directives"
datePublished: Mon Dec 30 2024 16:23:07 GMT+0000 (Coordinated Universal Time)
cuid: cm5b90glu000109jv0s9v7gab
slug: figma-like-input-field-in-angular-using-directives
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735573366018/09fd4230-7dce-4609-8e22-2a6288d45663.png
tags: angular, typescript

---

People who are familiar with Figma would have noticed that the input fields support dragging to increase or decrease values. Instead of having to click on the input field first and then type the number in, the dragging feature is really handy as you can easily get the desired value by dragging.

We can build something like that using Angular directives. We’ll use all the latest features of Angular in this experiment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735481574335/9e74a6f1-5a5f-4d87-92c4-2d1c4fd047e0.gif align="center")

Let's see how we can build this.

We can actually do this in multiple ways. We are going to build this using directives. The way we are going to do this is by taking a very generic approach. This way, we can reuse the logic for things like resizing elements or sidebars, etc.

## Scrubber Directive - the core functionality

The main logic of the input can be extracted and encapsulated into a directive. The main objective is to listen to the mouse events and then translate the mouse movements into a usable value. To explain in a bit more detail:

1. When the user clicks the mouse (`mousedown` event).
    
2. We start listening to the mouse movements (`mousemove` events) and use that info to translate it into usable values.
    
3. When the user releases the click, we stop the listener (`mouseup` event).
    

We will use `rxjs` to simplify the logic a bit.

Here's how the pseudocode would look.

```typescript
const mousedown$ = fromEvent<MouseEvent>(target, 'mousedown');
const mousemove$ = fromEvent<MouseEvent>(document, 'mousemove');
const mouseup$ = fromEvent<MouseEvent>(document, 'mouseup');

let startX = 0;
let step = 1;

mousedown$
  .pipe(
     tap((event) => {
       startX = event.clientX; // Initial x co-ordinate where the mouse down happened
    }),
    switchMap(() => mousemove$.pipe(takeUntil(mouseup$))))
  .subscribe((moveEvent) => {
    const delta = startX - moveEvent.clientX;
    const newValue = Math.round(startValueAtTheTimeOfDrag + delta);
  });
```

Looking at the above code, it should be pretty clear what is happening. We basically save the initial `clientX` value, which is the position of the click on the X axis. Once we have that info, when the user moves the mouse, we can calculate the delta from the initial start position and the current X position.

We can further add more customizations like:

1. **Sensitivity** - drag distance to final value will be decided by sensitivity. Higher sensitivity values mean the final value will be big even if the movement is not that much.
    
2. **Step** - sets the *stepping interval* when moving the mouse. If the step value is `1`, the final value is incremented/decremented in steps of `1`.
    
3. **Min** - minimum value that will be emitted.
    
4. **Max** - maximum value that will be emitted.
    

Here’s how the final directive would look:

```typescript
@Directive({
  selector: "[scrubber]",
})
export class ScrubberDirective {
  public readonly scrubberTarget = input.required<HTMLDivElement>({
    alias: "scrubber",
  });

  public readonly step = model<number>(1);
  public readonly min = model<number>(0);
  public readonly max = model<number>(100);
  public readonly startValue = model(0);
  public readonly sensitivity = model(0.1);

  public readonly scrubbing = output<number>();

  private isDragging = signal(false);
  private startX = signal(0);
  private readonly startValueAtTheTimeOfDrag = signal(0);
  private readonly destroyRef = inject(DestroyRef);
  private subs?: Subscription;

  constructor() {
    effect(() => {
      this.subs?.unsubscribe();
      this.subs = this.setupMouseEventListener(this.scrubberTarget());
    });

    this.destroyRef.onDestroy(() => {
      document.body.classList.remove('resizing');
      this.subs?.unsubscribe();
    });
  }

  private setupMouseEventListener(target: HTMLDivElement): Subscription {
    const mousedown$ = fromEvent<MouseEvent>(target, "mousedown");
    const mousemove$ = fromEvent<MouseEvent>(document, "mousemove");
    const mouseup$ = fromEvent<MouseEvent>(document, "mouseup");

    return mousedown$
      .pipe(
        tap((event) => {
          this.isDragging.set(true);
          this.startX.set(event.clientX);
          this.startValueAtTheTimeOfDrag.set(this.startValue());
          document.body.classList.add("resizing");
        }),
        switchMap(() =>
          mousemove$.pipe(
            takeUntil(
              mouseup$.pipe(
                tap(() => {
                  this.isDragging.set(false);
                  document.body.classList.remove("resizing");
                })
              )
            )
          )
        )
      )
      .subscribe((moveEvent) => {
        const delta = moveEvent.clientX - this.startX();
        const deltaWithSensitivityCompensation = delta * this.sensitivity();

        const newValue =
          Math.round(
            (this.startValueAtTheTimeOfDrag() +
              deltaWithSensitivityCompensation) /
              this.step()
          ) * this.step();

        this.emitChange(newValue);
        this.startValue.set(newValue);
      });
  }

  private emitChange(newValue: number): void {
    const clampedValue = Math.min(Math.max(newValue, this.min()), this.max());
    this.scrubbing.emit(clampedValue);
  }
}
```

## How to use the scrubber directive

Now that we have the directive ready, let’s see how we can actually start using it.

```xml
<div #scrubberTarget 
     [scrubber]="scrubberTarget"
     [startValue]="this.roundedNumericValue()"
     [min]="0"
     [max]="100"
     [step]="2"
     [sensitivity]="0.2"
     (scrubbing)="this.updateValue($event)">
</div>
```

Currently, we have marked the `scrubberTarget` input as `input.required`, but we can actually make it optional and automatically use the `elementRef.nativeElement` of the host of the directive, and it would work the same. The `scrubberTarget` is exposed as an input in case you want to set a different element as the target.

We also add a class `resizing` to the body so that we can set the resize cursor correctly.

```scss
.resizing {
  cursor: ew-resize;
  touch-action: none;
  -webkit-user-select: none;
  user-select: none;
}
```

We have used `effect` to start the listener, this would make sure that when in case target element changes, we set the listener on the new element.

## See it in action

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735495813889/e0077ec7-b1dc-4cd4-a173-7dfd8dad3cd4.gif?width=450 align="center")

We have made a super simple Scrubber directive in Angular which can help us build input fields similar to what Figma has. Makes is super easy for user to interact with numeric inputs.

### Code & Demo

[https://stackblitz.com/edit/figma-like-number-input-angular?file=src%2Fscrubber.directive.ts](https://stackblitz.com/edit/figma-like-number-input-angular?file=src%2Fscrubber.directive.ts)

## Connect with me

* [Twitter](https://twitter.com/AdiSreyaj)
    
* [Github](https://github.com/adisreyaj)
    
* [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
    

Do add your thoughts in the comments section. Stay Safe ❤️

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png align="left")](https://www.buymeacoffee.com/adisreyaj)