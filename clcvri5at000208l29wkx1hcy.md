# Architecting A Highly Dynamic Card List In Angular

Let's look at one way to architect Angular's highly customizable and dynamic card list component. The goal is to make it easier to render different kinds of card lists using one single component.

## Dynamic Card List Component

The component should be dynamic enough to render different designs while maintaining a common structure. The consumer should be able to use the component and provide just the config and a data source, the rest is all gonna be taken care of by our dynamic card list component.

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673551279830/6898ac5f-b772-4a38-8c6a-29cd260d8c32.png align="center")](https://examples.adi.so/examples/dynamic-card-list)

For table-like cards, if you want to render display different data in different designs; after a point, it becomes very difficult with dedicated card lists for each of the use cases. Having a single card list component that is dynamic enough to render a wide variety of designs would make things much easier.

## Features

* Highly customizable 🧱
    
* Re-usable ♻️
    
* Easy to use 🔌
    
* Less boilerplate code to maintain 🛠️
    
* Consistency in design 📐
    

## Concepts

There are a couple of things to be aware of to make sense of how we implement things. We also make good use of Angular's strong Dependency Injection (DI) system. We'll see how we can provide custom components and dynamically create and attach them to the view. We use concepts like Dynamic Component Creation, Injectors, Injection Tokens, etc.

### Renderers

So if we consider the card like a table, then the columns are where we display the data. So each of these columns can display different types of information like text, images, buttons, etc. So we can have different renderers for each of these types. For example, we can have a text renderer that will render text, an image renderer that will render an image, etc.

We introduce another decorator `ListColumnRenderer` that can be used to provide some metadata information about the renderer itself like the `type`. We'll see how the custom decorator helps us later.

#### Core Renderers

We will have some core renderers which will be used by default. For example,

* Text Renderer
    
* Badge Renderer
    

We can identify some common use cases for renderers and create components that become part of Core Renderers.

#### Custom Renderers

If there are use cases that are not covered by the core renderers, then we can create custom renderers. For example, we need to display a user's name with the profile picture. So we can create a custom renderer for this use case.

### Renderers Registry

So we need a way to keep track of the renderers that we will create and use in the application. Users can create a lot of custom renderers according to their use cases. We use a `type` to identify the renderer. So all the renders that are created should have a unique type. We can use this type `type` associated with them.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673635390453/98bf8bd3-c814-441f-b69b-7001203222c2.png align="center")

We use a **Service** as our registry (`ColumnRenderersRegistryService`). We expose a `Map` to store the type to renderer mapping. We expose two methods:

* `registerRenderers` to register renderers
    
* `lookup` to lookup a renderer by type
    

### Registering Renderers

There are methods exposed to register default renderers and custom renderers. The default renderers are registered in the constructor of the `sreyaj-exp-card-list`.

```typescript
export const registerDefaultRenderers = (): void => {
    registerCustomRenderers([
        TextColumnRendererComponent,
        BadgeColumnRendererComponent,
    ]);
};

export const registerCustomRenderers = (
    renderers: ListColumnRendererConstructor[]
): void => {
    const rendererLookupService = inject(ColumnRenderersRegistryService);
    rendererLookupService.registerRenderers(renderers);
};
```

We inject the `ColumnRenderersRegistryService` and use it to register the renderers.

### Custom Decorator

We create a custom decorator called `ListColumnRenderer` to add some metadata information about the renderer component like its `type` which is the unique identifier of the renderer in the registry.

We could totally do this without a decorator, but we would have to manually set the type on each renderer component. It would look like this:

```typescript
@Component()
export class TextColumnRendererComponent extends ListColumnRendererBase {
  public type = CoreListColumnRendererType.Text;
}
```

With the decorator, we just remove this explicit variable assignment. Also, the decorator helps us easily identify renderer components from normal components.

Class decorator is a function that receives the constructor as the argument. Since we want to pass in some arguments to the decorator itself we use a decorator factory.

```ts
export function ListColumnRenderer(
  metadata: ListColumnRendererMetadata
) {
  return (constructor: ListColumnRendererConstructor): void => {
    constructor.type = metadata.type;
  };
}
```

We want to have a `type` property for all renderers. The default type of a **Component** is `Type`. For us, we have extra metadata so we say our components are of the type `ListColumnRendererConstructor` which is an extended `Type` with some additional properties like `type` (more can be added later).

```ts
export interface ListColumnRendererConstructor extends Type<unknown>,
    ListColumnRendererMetadata {}

export interface ListColumnRendererMetadata {
  type: string;
}
```

## Implementation

### Main Component

There is a main component `sreyaj-exp-card-list` which is the component the consumers will use. The component takes in two inputs:

1. `columnConfig` - An array of column config that is used to render each column
    
2. `dataSource` - The data source which drives the component
    

The column config is what drives the view part of the component. Here is how you define it:

```typescript
[
  {
    id: "name",
    display: CustomRenderers.NameWithAvatar,
    width: 2
  },
  {
    id: "email",
    display: CoreListColumnRendererType.Text,
    width: 4
  }
]
```

The `id` is what connects the view and the data. The data source should contain an object with the keys mentioned `id` in the column config.

The `display` is the type of renderer. It can be a core renderer or a custom renderer. This key will be used to look up the corresponding renderer to be used for that particular column.

Inside the main component, it just loops through the columns:

```xml
<div>
  <sreyaj-exp-card-list-column-renderer [columnConfig]="column" [data]="item"></sreyaj-exp-card-list-column-renderer>
</div>
```

We pass the column information and the data for the column from the data source to a child component `sreyaj-exp-card-list-column-renderer` which takes care of dynamically creating and attacking the corresponding renderer to the view.

### Attaching Renderers to the View

This magic happens in the `sreyaj-exp-card-list-column-renderer` component. This is how we do it:

```typescript
if (!this.columnConfig || !this.data) {
  return;
}
// Get the corresponding renderer for the display type
const renderer = this.rendererLookupService.lookup(this.columnConfig?.display);
if (renderer) {
  // Dynamically create the component and pass the data with the help of Injection Token
  this.vcr.createComponent(renderer, {
    injector: Injector.create({
      providers: [
        {
          provide: COLUMN_RENDERER_DATA,
          useValue: this.data[this.columnConfig.id]
        }
      ],
      parent: this.parentInjector
    })
  });
}
```

If I were to breakdown what is happening inside the component:

1. Get the renderer corresponding to the `type` passed in the `columnConfig`
    
2. Use `ViewContainerRef` to dynamically create the component using the `createComponent` method.
    
3. To the `createComponent` method, we pass the renderer class along with an `injector` which holds the data.
    
4. `COLUMN_RENDERER_DATA` injection token is provided the data for that particular column.
    

### Inside a Renderer

Let's look at `TextColumnRendererComponent` which is part of the Core Renderers.

```typescript
@Component({
  selector: "sreyaj-exp-text-column-renderer",
  template: `
    <div>
      {{ data }}
      <!-- Use the data here -->
    </div>
  `,
  standalone: true
})
@ListColumnRenderer({
  type: CoreListColumnRendererType.Text
})
export class TextColumnRendererComponent extends ListColumnRendererBase {}
```

The `data` property contains the data passed from the injector, wondering how we get it. For that, let's look at `ListColumnRendererBase` :

```typescript
export abstract class ListColumnRendererBase<ColumnDataType = unknown> {
  public static readonly type: CoreListColumnRendererType;
  // We inject the token here
  public readonly data: ColumnDataType =
    inject<ColumnDataType>(COLUMN_RENDERER_DATA);
}
```

We can see that the `COLUMN_RENDERER_DATA` **InjectionToken** is injected inside the class and so the component gets the value passed while creating the component. If we don't use the new `inject` method, it would look something like this:

```typescript
@Component({
  selector: 'sreyaj-exp-text-column-renderer',
  template: `
    <div>
      {{ data }}
    </div>
  `,
  standalone: true,
})
export class TextColumnRendererComponent {
  constructor(@Inject(COLUMN_RENDERER_DATA) public data: unknown){}
}
```

### References

The idea of this is taken from the OSS project called [Hypertrace](https://www.hypertrace.org?utm_source=sreyaj.dev) which is an Open source distributed tracing platform from [Traceable](https://traceable.ai?utm_source=sreyaj.dev) (The company I'm currently working in).

The project has a table implementation which does exactly this. I've just used the same concept to create a card list instead of a table (also use standalone components).

The table is far more capable as it even takes care of sorting, pagination etc. You can look at the codebase to get an idea of how the [table](https://github.com/hypertrace/hypertrace-ui/tree/main/projects/components/src/table) is implemented.

| Title | Link |
| --- | --- |
| Source Code | [https://github.com/adisreyaj/sreyaj/tree/main/libs/examples/dynamic-card-list](https://github.com/adisreyaj/sreyaj/tree/main/libs/examples/dynamic-card-list) |
| Demo | [https://examples.adi.so/examples/dynamic-card-list](https://examples.adi.so/examples/dynamic-card-list) |
| Hypertrace Repo | [https://github.com/hypertrace/hypertrace-ui](https://github.com/hypertrace/hypertrace-ui) |