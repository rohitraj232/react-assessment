# Frontend Software Engineer Assessment

## 1. TypeScript

1. What’s the difference between `unknown` and `any` in TypeScript?
    - `unknown:` A type-safe counterpart of any. You can assign anything to it, but you cannot use it directly without first checking its type.You use it when you want flexibility but still keep type safety.
    - `any:` A type that turns off type checking. You can assign anything to it and use it in any way without TypeScript giving errors.You use it when you don’t care about type safety.



2. Show how you would use **mapped types** and **conditional types** to transform the following type into one where all fields are `readonly` and optional:

    ```ts
    type User = {
    id: number;
    name: string;
    email: string;
    };
    ```

    **mapped types**  
    ```ts
    type ReadonlyOptional<T> = {
    readonly [K in keyof T]?: T[K];
    };

    type UserTransformed = ReadonlyOptional<User>;
    ```

    **conditional types**  
    ```ts
    type ReadonlyOptionalString<T> = {
    readonly [K in keyof T]: T[K] extends string ? T[K] | undefined : T[K];
    };

    type UserStringTransformed = ReadonlyOptionalString<User>;
    ```


3. Given this type:
```ts
type ApiResponse<T> = 
  | { status: "success"; data: T }
  | { status: "error"; error: string };
```

- How would you write a **conditional utility type** that extracts only the `data` type from a successful response?
```ts
    type ExtractData<T> = T extends { status: "success"; data: infer D } ? D : never;
```

- How would you make a generic function that safely unwraps the `data` if the status is `"success"`?
```ts
    function getData<T>(response: ApiResponse<T>): T | undefined {
        if (response.status === "success") {
            return response.data;
        }
        return undefined;
    }
```


## 2. Component Design

Review the following React component. Identify any design issues or violations of React/Frontend best practices. Explain what problems could arise with this approach and suggest how you would refactor it to be more maintainable and extensible.

```ts
function Form() {
  const [value, setValue] = React.useState("");
  const [error, setError] = React.useState(false);

  function handleSubmit() {
    if (value.length < 3) {
      setError(true);
      alert("Value too short");
    } else {
      fetch("/api/submit", {
        method: "POST",
        body: JSON.stringify({ value }),
      });
      setValue("");
      setError(false);
    }
  }

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      {error && <span style={{ color: "red" }}>Too short!</span>}
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```
### Issues in this component

```jsx
    const [error, setError] = React.useState(false);
```
    - Using a boolean means you can only show one type of error. It’s better to store a string message so you can display different errors to the user.

```jsx
    fetch("/api/submit", ...);

Right now, if the request fails, nothing handles the error. We should wrap it in try/catch or use .then/.catch so failures are handled gracefully.
```

```jsx
    fetch("/api/submit", {
    method: "POST",
    body: JSON.stringify({ value }),
  });
```
Since there’s no await or .then, the code doesn’t wait for the server response and ignores any errors. This can lead to failed submissions without user feedback.

```jsx
    <input onChange={(e) => setValue(e.target.value)} />
    <span style={{ color: "red" }} />
```
    - Inline functions and styles recreate every render. Move them outside or use className for styles.
    - <input> has no label, which is bad for screen readers. 
    - <button> lacks type="button".
    - Not using <form> element.



## 3. Async Programming and Data Fetching

You are implementing a function to fetch user details and posts from a REST API and render them together.

```ts
async function fetchUserAndPosts(userId: string) {
  const user = await fetch(`/api/users/${userId}`).then((r) => r.json());
  const posts = await fetch(`/api/posts?userId=${userId}`).then((r) => r.json());
  return { user, posts };
}
```

- **What issues could arise with this approach in terms of performance and resiliency?**
    1. Right now, the code fetches the user first and then the posts. This means if fetching the user takes 1 second and fetching the posts takes 1 second, the total wait time is 2 seconds. We can make it faster by fetching them at the same time.
    2. If either fetch fails, the function will throw an error and stop. This could break the app and users won’t see a helpful error message.
    3. Every time the function runs, it fetches fresh data from the server, even if it’s the same user. This can be inefficient and use unnecessary network resources.

- **Suggest improvements using modern browser/React features (e.g., concurrency, caching, error handling).**
```ts
    async function fetchUserAndPosts(userId: string) {
        try {
            const [user, posts] = await Promise.all([
                fetch(`/api/users/${userId}`).then((r) => r.json()),
                fetch(`/api/posts?userId=${userId}`).then((r) => r.json()),
            ]);
            return { user, posts };
        } catch (error) {
            console.error("Failed to fetch user or posts:", error);
            throw error;
        }
    }
```


## 4. Resiliency & UI/UX Patterns

Review the following React code. Identify issues and propose which frontend resiliency patterns should be used to address them.

```ts
function UserProfile({ userId }: { userId: string }) {
  const [data, setData] = React.useState<any>(null);

  React.useEffect(() => {
    fetch(`/api/user/${userId}`)
      .then((res) => res.json())
      .then(setData);
  }, [userId]);

  if (!data) return <div>Loading...</div>;

  return <div>{data.name}</div>;
}
```

## Issues in the current code

```ts
    const [data, setData] = React.useState<any>(null);
```
    - Using any removes type safety, making it easy to runtime errors.

```jsx
    fetch(`/api/user/${userId}`)
    .then((res) => res.json())
    .then(setData);
```
    If the network request fails or the server returns an error, the component will break silently. The user will only see the “Loading…” message and won’t know anything went wrong.

```jsx
if (!data) return <div>Loading...</div>;
```
    - Uses data being null to indicate loading. If the fetch fails and data never arrives, the loading state never ends.


    - If the fetch fails, the user has no way to retry the request. No timeout or cancellation logic could leak memory if userId changes quickly.



## 5. Performance & Bundling

You are building a React SPA that has grown very large and now has slow initial load times.
- What techniques would you use to reduce bundle size and improve performance?
    - In order to minimize the bundle size as well as optimize performance, code splitting can be used in which the app can be split into smaller pieces so as to be loaded only if necessary. This can be followed by lazy loading of components as well as images so as to load only when necessary. Lastly, unused code can be stripped along with the compression of all files through caching of assets so as to optimize third-party libraries as well as defer non-critical JavaScript so as to prefetch resources.

- How would you decide between client-side rendering, server-side rendering, and static site generation?
    - It all depends on how your app delivers the content when it comes to choosing a method of rendering. Client-side rendering is best when building highly interactive, dynamic apps like dashboards where SEO isn't critical. Server-side rendering works well for pages that need fast initial loading and good SEO, such as e-commerce or blogs. Static site generation is ideal for mostly unchanging pages, like marketing sites or documentation, because pages are pre-built at build time, load very quickly, and are easy to cache across CDNs, improving performance.
