## Implementing Feature Flags in Angular is easier than you thought it would be!

Feature flags are basically a configuration for your application where we specify which features are enabled/disabled. We would normally just comment out that part of the code which we don't want to be made available and then later come back and uncomment it to enable that feature.

Instead of us having to make changes in the code every time we want to enable/disable something in our application, we can make use of a configuration file where we specify if that feature is enabled/disabled.

## Feature flags and why we need it
Feature flags are a very common technique that is widely used in many applications. If we are testing out a particular feature by only enabling it to a particular group of people (A/B Testing) or we need to disable a feature because it has some serious issues that would take time to fix, in these conditions it won't be practical to manually make changes in the code and push it every time we need to enable/disable something in the application.
Instead what we can do is create a configuration outside the application and then use that to turn on/off features in the application with ease. This means you can make changes fast without having to make changes in the code.

Also like I mentioned in the first part If you want to enable a particular feature to only a set of people you can easily do that by sending a different set of config for these people and the default config for all the other users based on some conditions.

## Implementing Feature flags in Angular
The term might make you think this is something really difficult to implement. But it's actually quite easy to do it angular with the help of some inbuilt features that Angular provides like directives, guards, etc.

### Configuration File
Its ideal that this file is managed outside the application and is made available via an API call. In that way, we can easily update the config and the application gets the new file with ease.

We are going to be managing a `JSON` object with the feature as the key and the value will be either `true` or `false`. We are going to keep it simple here, we can always create granular feature flags to get more control of the application. For the sake of this post, I would consider a module as a feature.

```ts
export interface FeatureConfig {
  [key:string]:boolean;
}
```
and the config file will be something like this:
```json
{
  "bookmarks": true,
  "snippets": true,
  "packages": false
}
```

### Application Structure
Our application has 3 modules:
- Bookmarks
- Snippets
- Packages

All these modules are lazy-loaded from the `app.module.ts`. So based on the configuration we have to load the modules.
Also, we have a header component where links to these modules will be added. So we have to manage that as well, ie If the packages module is not enabled, we shouldn't be showing `Packages` in the header.

Here is our routing module:
```ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { RouterModule, Route } from "@angular/router";

const routes: Route[] = [
  {
    path: "snippets",
    loadChildren: () =>
      import("./snippets/snippets.module").then(m => m.SnippetsModule),
    data: {
      feature: "snippets" // <-- key that is specified in the config
    }
  },
  {
    path: "bookmarks",
    loadChildren: () =>
      import("./bookmarks/bookmarks.module").then(m => m.BookmarksModule),
    data: {
      feature: "bookmarks"
    }
  },
  {
    path: "packages",
    loadChildren: () =>
      import("./packages/packages.module").then(m => m.PackagesModule),
    data: {
      feature: "packages"
    }
  }
];

@NgModule({
  imports: [CommonModule, RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}

```
One thing that you can notice is that I've provided the feature name in the `data` attribute so that we can identify which module is being loaded.

### Feature Flag service

We create a feature flag service where we are going to manage all the logic for getting the config and also functions to check if a feature is enabled or not.

```ts
import { HttpClient } from "@angular/common/http";
import { Injectable } from "@angular/core";
import { get, has } from "lodash-es";
import { tap } from "rxjs/operators";
import { FeatureConfig } from "../interfaces/feature.interface";

@Injectable({
  providedIn: "root"
})
export class FeatureFlagsService {
  config: FeatureConfig = null;
  configUrl = ``; // <-- URL for getting the config

  constructor(private http: HttpClient) {}

  /**
   * We convert it to promise so that this function can
   * be called by the APP_INITIALIZER
   */
  loadConfig() {
    return this.http
      .get<FeatureConfig>(this.configUrl)
      .pipe(tap(data => (this.config = data)))
      .toPromise();
  }

  isFeatureEnabled(key: string) {
    if (this.config && has(this.config, key)) {
      return get(this.config, key, false);
    }
   return false;
  }
}

```

We are adding two function inside our service:
- `loadConfig()` - Get the config from an API
-  `isFeatureEnabled(key: string): boolean` - Check if a particular feature is enabled

Now that we have our service ready, we make use of `APP_INITIALIZER`. This is an `Injection Token` provided by Angular where we can provide a function that will be called during app initialization.

Read more: https://angular.io/api/core/APP_INITIALIZER

### Configure `APP_INITIALIZER`

We have to add our provide function so that it will call the API and loads the configuration on startup.

Create a factory that will return the call the `loadConfig()` function in our `FeatureFlagsService`. And add `APP_INITIALIZER` in our providers array
```ts
import { APP_INITIALIZER, NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";

import { AppComponent } from "./app.component";
import { AppRoutingModule } from "./app-routing.module";
import { FeatureFlagsService } from "./core/services/feature-flags.service";
import { HttpClientModule } from "@angular/common/http";

const featureFactory = (featureFlagsService: FeatureFlagsService) => () =>
  featureFlagsService.loadConfig();

@NgModule({
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: featureFactory,
      deps: [FeatureFlagsService],
      multi: true
    }
  ]
})
export class AppModule {}
```

So now when our application gets initialized, the config will be loaded in our `FeatureFlagsService`.

### Route Guard
We now can create a route guard to only load modules if the feature is enabled. For that we create a `canLoad` guard:
```ts
import { Injectable } from '@angular/core';
import { CanLoad, Route, Router, UrlSegment, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';
import { FeatureFlagsService } from '../services/feature-flags.service';

@Injectable({
  providedIn: 'root',
})
export class FeatureGuard implements CanLoad {
  constructor(
    private featureFlagsService: FeatureFlagsService,
    private router: Router
  ) {}
  canLoad(
    route: Route,
    segments: UrlSegment[]
  ):
    | Observable<boolean | UrlTree>
    | Promise<boolean | UrlTree>
    | boolean
    | UrlTree {
    const {
      data: { feature }, // <-- Get the module name from route data
    } = route;
    if (feature) {
      const isEnabled = this.featureFlagsService.isFeatureEnabled(feature);
      if (isEnabled) {
        return true;
      }
    }
    this.router.navigate(['/']);
    return false;
  }
}
```
We can now update the `app-routing.module.ts` file to include our guard:
```ts
const routes: Route[] = [
  {
    path: "snippets",
    loadChildren: () =>
      import("./snippets/snippets.module").then(m => m.SnippetsModule),
    canLoad: [FeatureGuard],
    data: {
      feature: "snippets"
    }
  },
  {
    path: "bookmarks",
    loadChildren: () =>
      import("./bookmarks/bookmarks.module").then(m => m.BookmarksModule),
    canLoad: [FeatureGuard],
    data: {
      feature: "bookmarks"
    }
  },
  {
    path: "packages",
    loadChildren: () =>
      import("./packages/packages.module").then(m => m.PackagesModule),
    canLoad: [FeatureGuard],
    data: {
      feature: "packages"
    }
  }
];
```
So now when someone tries to visit the URL, the guard will check if that particular feature is enabled or not, and only then will allow navigating to that particular module. The first part is now done. The next thing we have to do is to show the header link only when the feature is enabled. For that we will be creating a [Directive](https://angular.io/guide/structural-directives), to be more precise a `Structural Directive`

### Feature flag directive
Directives are a really powerful feature that Angular provides. We will be creating a structural directive for our use-case:
```ts
import {
  Directive,
  Input,
  OnInit,
  TemplateRef,
  ViewContainerRef
} from "@angular/core";
import { FeatureFlagsService } from "../services/feature-flags.service";

@Directive({
  selector: "[featureFlag]"
})
export class FeatureFlagDirective implements OnInit {
  @Input() featureFlag: string;
  constructor(
    private tpl: TemplateRef<any>,
    private vcr: ViewContainerRef,
    private featureFlagService: FeatureFlagsService
  ) {}

  ngOnInit() {
    const isEnabled = this.featureFlagService.isFeatureEnabled(this.feature);
    if (isEnabled) {
      this.vcr.createEmbeddedView(this.tpl);
    }
  }
}
```
So what we are doing here is rendering the template only if the feature is enabled. If not that particular element will not be placed in the DOM.
Note that the name of the directive and the `@Input()` is the same so that we can receive input without having to add another attribute in the HTML.

### Using the directive
This is how we use the directive in HTML:
```html
<header>
  <nav>
    <ng-container *featureFlag="'snippets'">
      <a routerLink="/snippets">Snippets</a>
    </ng-container>
    <ng-container *featureFlag="'bookmarks'">
      <a routerLink="/bookmarks">Bookmarks</a>
    </ng-container>
    <ng-container *featureFlag="'packages'">
      <a routerLink="/packages">Packages</a>
    </ng-container>
  </nav>
</header>
<main>
  <router-outlet></router-outlet>
</main>
```
We add the directive `*featureFlag` in and pass the key for the feature to it.

Done! We have successfully implemented Feature flags in Angular. You might feel like there is a lot of code in here, but in essence, there are 3 main things:
1. Feature Flag Service
2. Feature Flag Guard
3. Feature Flag Directive

### Links
- Stackblitz: https://stackblitz.com/edit/angular-feature-flags
- Repo: https://github.com/adisreyaj/angular-feature-flags

These are the 3 main things that we need. Hope you are now aware of how to implement feature flags in Angular. If something is not clear, just try to read through the code line by line and it'll make sense.

## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cardify](https://cardify.adi.so) - Dynamic SVG Images for Github Readmes


Do add your thoughts in the comments section.
Stay Safe ❤️

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png)](https://www.buymeacoffee.com/adisreyaj)
