# Function Types

## Funtion Types declarations

```TS
// 1. Type alias for a function type:
type TFn = (x: number) => string;
const toStrT: TFn = x => '' + x;

// 2. Interface that describes a callable object (function):
interface IFn {
  (x: number): string;
}
const toStrI: IFn = x => '' + x;

// 3. Type alias is used as object-like syntax to describe the function (object with a callable signature).
type TFnAlt = {
  (x: number): string;
};
const toStrTAlt: TFnAlt = x => '' + x;
```

`IFn` & `TFnAlt` are useful  with  overloaded  function  signatures.

## Function Signature Types

### A function wrapper with the same signature

For example, you want to wrap `fetch` function to handle HTTP errors:
```TS
const fetchSafely: typeof fetch = async (input, init) => {
    const response = await fetch(input, init);
    
    if (!response.ok) 
        throw new Error(`Request failed: ${response.status}`);
    
    return response;
}
```


### A function wrapper with the same arguments but different return type

For example, you want to wrap `fetch` function to get numbers:
```TS
async function fetchNumber(...args: Parameters<typeof fetch>): Promise<number> {
    const response = await fetchSafely(...args);
    const result = Number(await response.text());

    if (isNaN(result))
        throw new Error("Response is not a number.");

    return result;
}
```

[Playground](https://www.typescriptlang.org/play/?#code/PQKhCgAIUhlBDAZgU0gdwE7wA7eRyRAewIAMUAXAYwAtSoRhwqiA7AZwsOWpoRQA2ATwBckCkLxFE3XpAC8keOyGsqkABQBLVtgCuFADSQdWigEoFAPkgBvKJEcsOXDMnbY27VIvhp4ZrK02roGxqYWANwOjjEmMhoAhG4eXsgAdEQA1pZxjo4UNBhEaJCsyKUAohjFGBqkAErIAI567lyIAQLIACZiACS2KZ4cGZzwFHrsAL6k5tH5kHFukxiskMNp0dPg4KAQ0JAA0sjI2IR6apDsWgDmrBN6bpAARgaQtPCst6iFqCtPdYSPDpBhMZSqdSIS5UChaNhBGgAOT0AFsXvgNOlsfAMLd2GIAAq4+Conj4dgAHmByGkiKs5iJxVRWm8lNYaIxGBs9kWzk4G3cI28CiU-kClFo-GQwixOLx7HmcX5rncegEXEUKPRmL8AVVqVG6QoyAAHhQNOYlXEtAlWUj4EiNCl1RZzHl8oViqVylUaiQNAAiJqGkWsspELjwMqc-DpQPWxYAtaC9iu7ZAA)
