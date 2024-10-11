# Infer Function Arguments Type

For a function that accepts an argument of an anonymous type where `hobbies` is a tuple:
```TS
function stringifyPerson(person: {
    name: string;
    age: number;
    hobbies: [string, string]; // tuple
}) {
    return `${person.name} is ${person.age} years old and loves ${person.hobbies.join(" and  ")}.`;
}
```
create a similar type:
```TS
const alex = {
    name: 'Alex',
    age: 20,
    hobbies: ['walking', 'cooking'] // here 'hobbies` is array, not tupple: string[] != [string, string]
}
```
trying to call the function with `alex` as argument will cause an error:
```TS
stringifyPerson(alex); // Error: Type string[] is not assignable to type [string, string]
```
Let's declare type allias with tuple:
```TS
type Person = {
    name: string;
    age: number;
    hobbies: [string, string]; // tuple
}
```
and create another value of this type explicitly:
```TS
const alex2: Person = {
    name: 'Alex',
    age: 20,
    hobbies: ['walking', 'cooking'] // now it is tuple = `[string, string]`
}
```
then call the function:
```TS
stringifyPerson(alex2); // now it's Ok
```

**What if you don't want to declare the type explicitly?** 

Instead you may want to *infer* the type from the known function argument. For that you can create a generic type that can infer that type for you.

```TS
type FuncFirstArgumentType<T> = T extends (first: infer TFirst, ...args: any[]) => any
  ? TFirst
  : never;

// .. or alternative variant
type FuncFirstArgumentType<T> = T extends (...args: [infer TFirst, ...any[]]) => any
    ? TFirst
    : never;
```
and get that type:
```TS
type PersonInferredType = FuncFirstArgumentType<typeof stringifyPerson>;
```
The `PersonInferredType` is identical to explicitly delcared `Person` type and can be used the same way:
```TS
const alex3: PersonInferredType = {
    name: 'Alex',
    age: 20,
    hobbies: ['walking', 'cooking'] // it is tuple = `[string, string]`
}
```
calling the function will cause no errors:
```TS
stringifyPerson(alex3); /* No TypeScript errors */ 
```

Still this `FuncFirstArgumentType<T>` type extractor has one litle flaw - it accepts any type:
`type NeverType = FuncFirstArgumentType<number>;`
It would be better to restrict it to functions only. Lets do it:
```
type FuncFirstArgumentTypeFixed<T extends (...args: any[]) => any> = T extends (first: infer TFirst, ...args: any[]) => any
    ? TFirst
    : never;
```

Now it won't accept anythin except function:
```
type CompilerError = FuncFirstArgumentTypeFixed<number>; // Error: Type 'number' does not satisfy the constraint '(...args: any[]) => any'.
```
```
function testFunc(n: number, s: string) { }
type Test2FuncType = FuncFirstArgumentTypeFixed<typeof testFunc>; // `number`
```

## Function with no argumetns
What happens if we pass the type of the function without a single argument?
 We will get `unknown`.
```
function zeroArgsFunc() { }
type zeroArgsFuncType = FuncFirstArgumentTypeFixed<typeof zeroArgsFunc>; // `unknown`
```
It is not exactly what we would like. It's much better to get `never'. 
That's what we'll do:
```
type FuncFirstArgumentType<T extends (...args: any[]) => any> = 
    T extends (...args: []) => any // Step 1: Check if the function has zero arguments..
        ? never             // Step 2: If yes, return `never`.
        : T extends (first: infer TFirst, ...args: any[]) => any // Step 3: Otherwise, try to infer the first argument..
            ? TFirst        // Step 4: If the first argument can be inferred, return its type.
            : never;        // Step 5: If the function doesn't match the pattern, return `never`.
```
or another alternative:
```
type FuncFirstArgumentType<T extends (...args: any[]) => any> = 
    Parameters<T> extends [infer TFirst, ...any[]]  // utility type `Parameters<T>` returns a tuple of the argument types for the function `T`, it ensures that the function has at least one argument by checking that the tuple starts with `TFirst` and can have additional arguments `(...any[])`
        ? TFirst 
        : never;
```

[PLAYGROUND](https://www.typescriptlang.org/play/?#code/PTAEEFQMwVwOwMYBcCWB7OokAsCGTRcEEBTAByQGdDNcAnAcxgFsS4C0oaaMBPZtDGpJeZEqADu2EnXHY0AIwUoS1FNVxYYZADYkAXAChDsRKgyhKSOijgMUUXgAUZlDAAoxdN3H2gA3oagwaBwuKx+VjZ2ANxBIbgMBqEsCjJxIaDySiqUfgDaUbYMADSW1sUAujGgIFq6JIYAvgCUAfHBskgwdJgABgAk-l4+AHRhrE2g6qBDIxijiSRTvCT01Gg6ACY0OzpoAG6qs8OuC9nKqqMAVmi27gBEu8EPLU2jfXFNxnUIsvjiTSUFDMFA6ehYUQGYwIDBWQh6AAeoAAvO1MhNkgBycBIrElDqEJJ+ABMAAYCZkLrkCliJLgdABrYr40BY2FoZl2LGVWpgaSyNnU1R9aYaOh0XC8MpwNAEbpkBqRCp2fK8gCEaMKKtK5WiDEqzR+YGsvGKWDQoAQDJ0WGk0HgyHQmAkKBwCJIyNw4qYrHYkjBtutQkBmBkdDQdCMhiKdgczjOcHcDM9LRqdQAKlC9cU1WLQnLCJRgQwwgo9BbIWJQNr9WVYwbjaAADIkJBY6hbEgIcGCkTVm0ob0B90KvTR-viFzeCxowIY8LJBsZBLElLMNJ0FfBYV5GsN+s66p8+p6I2GOq4OA7P5rJChuUCwiMFhsDhcHAzSegT26FAIN0dF4aNYTgeEU0REk-GnHxUXREJMT8HE8UpVdknJVCd0US493yOkGS5BhWXZNBORZXk6llCRpgIL9tArNE+lrYpD31So+nPOocDYK0bTtcRTCdDBowbeMYI8CCSTTE8qJojtQAAeUZJsAHU8ForheEEUAtgwLECHpf0kEtW8AR-RE-wA+UoQAflAABJMD71wHYtJgUBmClSQr3lS1bCgGR+KrASI2YILGSozBBPMWgXz9JBRlAAAxSM7XwUA3N4zBTPvQhQCSOAZH-YK0oIa1MH8wKcHS78oFStzRmMb8ksdJKUG8JBwDit8szEAAeDMAD44Izcz72vah3CgdqrD8Sq6FADM2o6spRjW+gGD3K9eDVNoUWG7b4jspaZqQeI-EKo4tybK8diSeV1OCids3EuBHICiUSC2XrxDRFrEGWqwut9HqoT6ydOBzONHFewa4gvMAM3tV73vDL6fvzFAu3Yf8GUrX8dH-QDeB0kgdGtWQdlekrbqy0A0lAEMdm4yxF284CYThAgIIAZmgxNUc+77sznQlELZXFPXxQkllJClCV3Wl6SZFkyhIsjuQosA3XzMdftAJiDyhg0OO+BHeJ0Qm7CC6LnQDS3eJDAsfwlSM8mMUSYcTZMkR56TgAAKlAAA5S0foAZT+FAKBdiNvFAAOwCAA)
