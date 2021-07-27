title: Fun with async generators and for await
date: 2021-07-26 16:30:51
categories: JavaScript

---

Async generators (and their close friend `for await`) are a really cool feature of modern JavaScript that you can use when a loop requires _asynchronous iteration_. What the heck is that? Well let's look at a regular `for...of` loop:

```js
const array = [1, 2, 3];
for (const el of array) {
  console.log(el);
}

// 1
// 2
// 3
```

Nothing weird here, we're looping over an array and printing each element. An array is a synchronous data structure, so we can loop over it very simply. But what about asynchronous data, like say the `fetch` API to get data from a HTTP endpoint. A simple implementation of looping over that data is not much more complex:

```js
const res = await fetch("https://whatever.com/api");

if (res.ok) {
  const data = await res.json();

  for (const el of data) {
    console.log(el);
  }
}
```

We're still using `for...of` and being synchronous in our loop so let's add one more factor: we're using an API that uses pagination of some sort, which is pretty much every API ever made since it's expensive to load giant datasets. A simple pagination-based iteration might look something like this:

```js
let data = [];
const pageSize = 10;
let page = 1;

do {
  const res = await fetch(
    `https://whatever.com/api?pageSize=${pageSize}&page=${page}`
  );

  if (!res.ok) {
    throw new Error("Badly written error message");
  }

  const data = await res.json();

  for (const el of data) {
    console.log(el);
  }

  page++;
} while (data.length === pageSize);
```

This is sort of async iteration but it suffers from a major shortcoming: you have to comingle the iteration code and the data fetching in the same loop as each page is ephemeral - or else select all the pages and allocate a monster array before you can deal with the data. Neither of those are great options, so let's try an _async generator_ function instead. An async generator is a function that returns an _async iterable_ that you can loop over. Like other iterable and enumerable types (such as `IEnumerable` in C#), async iterables in JS are essentially an object with a `next` function to get the next thing in the iterable. This has several interesting side effects:

- You cannot have random access to an iterable (it is forward-only)
- Because it is forward only, memory need not be allocated for the entire iterable at the same time so it's easily possible to have an iterable that has a nearly unlimited of iterations without running out of memory.
- If you stop iterating, the rest of the elements in the iterable are never evaluated. For our pagination example, this means that if we stop reading at page 2, but page 3 exists, we'll never fetch page 3 from the server.

For my own sanity I'm going to drop into TypeScript here to illustrate the types that we're passing around :)

```ts
// This paginateAsync function implements generic pagination functionality over an arbitrary API,
// using a passed in fetchPage function to tell it how to fetch a new page of data.
// the * means the function is a generator (which returns an iterator)
// and the async means it's an async generator, not a sync generator
// (for C# readers, this is really similar to a method returning IEnumerable and using `yield return`)
async function* paginateAsync<TResult>(
  fetchPage: (offset: number, limit: number) => Promise<TResult[]>,
  pageSize: number
): AsyncIterableIterator<TResult> {
  let offset = 0;
  let pageData: TResult[] = [];

  do {
    // fetch one page of data
    pageData = await fetchPage(offset, pageSize);

    // yield each item in the page (lets the async iterator move through one page of data)
    for (const item of pageData) {
      yield item;
    }

    // increase the offset by the page count so the next iteration fetches fresh data
    offset += pageSize;
  } while (pageData.length === pageSize);
}

// to use this function, we call it (note the lack of `await`; we're starting the iterator but requests don't occur until we loop over it)
// then using the resulting iterator we can iterate over it using the `for await` loop:
const iterator = paginateAsync(
  (offset, limit) =>
    fetch(`https://whatever.com/api?limit=${limit}&offset=${offset}`),
  100
);

for await (const item of iterator) {
  // every time 100 iterations are done here,
  // a new API call will be sent for the next page.
  // this is transparent to your iteration.
  console.log(item);

  // unlike returning a huge array from a single await,
  // this for await construct only ever has 100 items in memory at a time,
  // so you can tune your batch sizes.

  // you can also abort the iteration before it's complete
  // for example if I break the loop after 199 items no request
  // for page 3 will ever be made.
  // if(index === 199) break;
}
```

So now you can call a paginated API and treat the result as if it were a non-paginated loop. Pretty neat, right? Even better, you can use this in all modern browsers. (IE ain't modern, folks...)

Again for my C# readers: this is pretty similar conceptually to C#'s `IAsyncEnumerable` construct and can be used in similar circumstances.

#### References:

- [for-await...of @ MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of#iterating_over_async_generators)
- [Async iteration and generators @ javascript.info](https://javascript.info/async-iterators-generators)
