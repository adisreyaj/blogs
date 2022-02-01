## What the heck is HttpContext in Angular?

Have you heard about HttpContext in Angular? If not, there is such a thing. HttpContext is used to pass additional metadata to HTTP Interceptors in Angular. 

## HttpContext in Angular

HttpContext is used to store additional metadata that can be accessed from HTTP Interceptors. Before this, there was no proper way to configure interceptors on a per request basis. This feature was introduced by [Angular v12](https://github.com/angular/angular/pull/25751).

If you had use cases where you need to treat a particular request differently or override some logic in an HTTP Interceptor, this is what you need to use. 

I came to know about it only recently and actually started using it in one of my recent projects - [Libshare](https://github.com/adisreyaj/libshare/blob/2e2a41ab46e942fbe5b47e7c60fdf7dea870e3a8/src/app/core/interceptors/auth.interceptor.ts#L12). 

## How to use HttpContext in Angular?
Let's take a practical use case for understanding how to use HttpContext.

 I was working on a small application that can be used to curate and share the libraries. Most of the APIs are authenticated, meaning we need to add `Authorization` header with all the API requests.

For pages like Login and Signup, we don't need to pass the token in the headers. Let's see how we can skip certain APIs and only add Bearer tokens to the other API requests.

The way we use it is pretty straightforward. There are two parts to it:
### 1. Create a new `HttpContextToken`
```ts
const IS_PUBLIC_API = new HttpContextToken<boolean>(() => false);
```
### 2. Passing the context while making `http` calls.

When using the `HttpClient` to make requests, you can pass the `context` along with other options.
We create an instance of `HttpContext` class and use the `set` method to pass the value to the token we created above.

```ts
getSomeData(slug: string) {
    return this.http
      .get(<URL>, {
        context: new HttpContext().set(IS_PUBLIC_API, true),
      })
  }
```
### 3. Retrieve the data inside an Interceptor.
We can now retrieve the context data from the interceptor by accessing the `request.context`:
```ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    if (req.context.get(IS_PUBLIC_API)) {
      return next.handle(req);
    }
   // Add token to other requests
  }
}
```
You can check a practical usage here: [Libshare Repo](https://github.com/adisreyaj/libshare/blob/main/src/app/core/interceptors/auth.interceptor.ts)

## Addtional Info

HttpContext is backed by a Map and so has methods like:
```ts
class HttpContext {
  set<T>(token: HttpContextToken<T>, value: T): HttpContext
  get<T>(token: HttpContextToken<T>): T
  delete(token: HttpContextToken<unknown>): HttpContext
  has(token: HttpContextToken<unknown>): boolean
  keys(): IterableIterator<HttpContextToken<unknown>>
}
```
The context is also **type-safe**, which is a good thing.
Also, keep in mind that the context is mutable and is shared between cloned requests unless explicitly specified.

There are a lot of ways this could prove useful like if you want to **cache** only particular requests, or maybe add some **additional headers** conditionally. 

Documentation: https://angular.io/api/common/http/HttpContext

## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cardify](https://cardify.adi.so) - Dynamic SVG Images for Github Readmes


Do add your thoughts in the comments section.
Stay Safe ❤️

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png)](https://www.buymeacoffee.com/adisreyaj)