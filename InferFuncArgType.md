# Infer Function Arguments Type

## Prerequisite:

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
## Restrict `T` argument to a function

`FuncFirstArgumentType<T>` type extractor has one litle flaw - _it accepts any type_:
`type NeverType = FuncFirstArgumentType<number>;`

It would be better to restrict `T` to functions only. Lets do it:
```
type FuncFirstArgumentTypeFixed<T extends (...args: any[]) => any> = T extends (first: infer TFirst, ...args: any[]) => any
    ? TFirst
    : never;
```

**Now it won't accept anything except function:**
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
## Improve `T` parameter restriction to function with arguments (no solutions!)
You may want make further improvements to restict function type argument to a function with at least 1 argument. Anfortunately there is no sulution for that. 
You may try something like this
```
    type FuncFirstArgumentTypeChecked<T extends (...args: any[]) => any> = 
        T extends (...args: []) => any // Check if the function has zero arguments..
            ? never             // If yes, return `never`.
            : T;

    type FuncFirstArgumentType<T extends FuncFirstArgumentTypeChecked<T>> = T extends (first: infer TFirst, ...args: any[]) => any // Error: Type parameter 'T' has a circular constraint.(2313)
            ? TFirst 
            : never;
```
but the compiler will error: `Type parameter 'T' has a circular constraint.(2313)`

[PLAYGROUND](https://www.typescriptlang.org/play/?#code/PTAEEFQMwVwOwMYBcCWB7OokAsCGTRcEEBTAByQGdDNcAnAcxgFsS4C0oaaMBPZtDGpJeZEqADu2EnXHY0AIwUoS1FNVxYYZADYkAXAChDsRKgyhKSOijgMUUXgAUZlDAAoxdN3H2gA3oagwaBwuKx+VjZ2ANxBIbgMBqEsCjJxIaDySiqUfgDaUbYMADSW1sUAujGgIFq6JIYAvgCUAfHBskgwdJgABgAk-l4+AHRhrE2g6qBDIxijiSRTvCT01Gg6ACY0OzpoAG6qs8OuC9nKqqMAVmi27gBEu8EPLU2jfXFNxnUIsvjiTSUFDMFA6ehYUQGYwIDBWQh6AAeoAAvO1MhNkgBycBIrElDqEJJ+ABMAAYCZkLrkCliJLgdABrYr40BY2FoZl2LGVWpgaSyNnU1R9aYaOh0XC8MpwNAEbpkBqRCp2fK8gCEaMKKtK5WiDEqzR+YGsvGKWDQoAQDJ0WGk0HgyHQmAkKBwCJIyNw4qYrHYkjBtutQkBmBkdDQdCMhiKdgczjOcHcDM9LRqdQAKlC9cU1WLQnLCJRgQwwgo9BbIWJQNr9WVYwbjaAADIkJBY6hbEgIcGCkTVm0ob0B90KvTR-viFzeCxowIY8LJBsZBLElLMNJ0FfBYV5GsN+s66p8+p6I2GOq4OA7P5rJChuUCwiMFhsDhcHAzSegT26FAIN0dF4aNYTgeEU0REk-GnHxUXREJMT8HE8UpVdknJVCd0US493yOkGS5BhWXZNBORZXk6llCRpgIL9tArNE+lrYpD31So+nPOocDYK0bTtcRTCdDBowbeMYI8CCSTTE8qJojtQAAeUZJsAHU8ForheEEUAtgwLECHpf0kEtW8AR-RE-wA+UoQAflAABJMD71wHYtJgUBmClSQr3lS1bCgGR+KrASI2YILGSozBBPMWgXz9JBRlAAAxSM7XwUA3N4zBTPvQhQCSOAZH-YK0oIa1MH8wKcHS78oFStzRmMb8ksdJKUG8JBwDit8szEAAeDMAD44Izcz72vah3CgdqrD8Sq6FADM2o6spRjW+gGD3K9eDVNoUWG7bCTspaZqQQk-EKo4tybNbQFShl716fAUCOUADnoId2CMZrWtOrrfR6qESQG4a0VGz1xq2Sa1sWRhcPmxblqsVb1rgHbKkqPaDrRo7EdO87QhIK64gvMArx2JJ5XU4KJ2zcS4EcgKJRILZevENEWsQJHOu69g2b6ydOBzONHHpwa4m-enGfDFm2ZJODOYQbn-tfPmgYFqEhdE0XE3FpsM3tKW4CZ2RWezGYUC7dh-wZStfx0f9AN4HSSB0a1TdAemSvJrLQDSUAQx2bjLEXbzgJhOECAggBmaDE2l5mzerOdCUQtlcU9fFCSWUkKUJXdaXpJkWTKEiyO5CiwDdfMx3Z0AmIPYWDQ475Sd4nQHbsILoudAMO94kMCx-CVIzyYxtYTGckxj6TgAAKlAAA5S02YAZT+FAKGHiNvFAOfgCbBSenrxXld5pB+aG0VvwhyVkFSvANkK0AHaQCsoHBaiAFoaMIYhyCoDQZ2k5vrZkXkTGQbMFa-Q6ireK-M4CpBkHrNu9kypXj9gJFAiIWZ2hmPScOP0uZ-XPmzNqOCtgDTGmwKGoB3Aww2ltNGu1UTY14KDRa1CJp0Omh1OaxtAonRWqABhcM-DbRYftIBuMhFWAJpddITZl7UWrhIPSUd-5b22p+MMiJSBbx7sJQw34ADCaBmBkDBDIAAoiPBaHMYFWDgYDMQ5CWZ9UQRuZB6YwC2J3n4KBWJPGbixDpNAxxZQEEoM9SgjggqgSiLgWwBAsT0PWmIoBki2FYkaoYzA94rCK3cL4dcm56zKn1G0fwoBvjfgzKoJAJJFZQIccQ2BpCoRuMoYLD8DTFbixPH0YJMgOImEdDFUAAAvGQaB-qUCKVUmpxjswAC0ZlzOadmVpSsSEAzVq47B7ielTPWXDfpPj67wAimgCQcBRl1DQaAVeqB+5EH0YA7aDozC9wgS6N02ATkRlAGQegi5HqUHVG3VsgDprIk-BoagEhXY6CMPOEIRCdntL2RfKEVCIY0Mmrw2aQCUaw02uI5hmNWFAI4eDREkNCWnX4UzPGwjRHksyVSqRh1MiZGOtzQkmQLoQOupkb8ayIwbMdC05KjiebYv5sc6ZkqzmOgGXUPoVzIqih-lYQMf93nCGwDMPJnEwDQuoLC3+xlQBdB6JgBRC06oLWVWgL+G1VYwvGc6VFhIMVnwVbiulDK6HsqYejLGNK4KCpCMGglob0kcqyUAk8zzyCgAAIx+BMdIBAjJpgfntHkrIw5XXPmxZQNaMbeWgDso6mtDa6hprIKAKCDlNKqDKHa3o9dHV9Eag23lASuG0Kmky6YAiFqyKQKSxhFKI3Us+U2+8LbY6KW4nQV0lASBlFNJWBGwciVR3PlWwdg7+WnUHcu9NAAWPw9lC1YI6uWz1vt-bzVNl2ts9qaLCChAOs9NbhXEyvWAZtoAACs97H1fKEpgXSqg4D6Q8vgBAALg4gqQI9OAX7ug9qGSK-txgxXZgKUgRWcyZWn12Z6xVmtemFLVRcoZSC6AcRI9WV1UrEBUblc4-ZJANZiCFlx1ViB1VgAI1dFubdUY7AejIMIqBXpuB0DAGKvqOPiGo1i2jQaR3Q0TeG5N20OHVs9qC1g4KQYGZrAjads7KW8hPOpsEbpgHZj6E4SzbZXAg1FN2sCeVa53Rgx6+KwVLWpUPd6iwfQMx9DKNXNglAejHGqlTASsXMCP0IAQPQ3oODP3C2+P2zs0PdkIqVIKIWrD0EAa6d08Xuaih9uVEtr0XJbDdM6W2JX2DUD6GkxYlKWjscA7W1l8JzNCsJsTYj6LSN9MdJRrZsq2lOI6f1Y5ZHzmDOGWxv12ZRObU2cndbmLNuBu2-RwFsyxMIAk72wjRo6jqne8lcAzZV7WIgBmDM1iACyThRo2tkEUZA9cEuVmLY1gF-WqCacW9WHTV29NiBzZV9xcbuHDbnZyyNpno2Dpx6OsNBQuVsJPJjvNBbu7ZZLdQMtCPK0AcA3WkVE3ggPI7ZQXDP6pMjLZ2egJJMtMXYDejwTpPqCo-lVLmnjJseDVpbZsdfCJ0socyIoz86TNoxPFAkFkorOBSxBmUJuXNAAToAgGAvYrSR0lMk0Y7gSTRwzdHFoM2QgXufT74IwHFFHerLtlbcNeMbfl-A3FO3lvieYwd8bwRvwnfmdKtbcv+M4pu8Jrgae9sar7eeczQA)
