# Unions

## Unions as type property keys

```TS
type StringKeys = 'firstName' | 'lastName';
type NumberKeys = 'age';

type Person = {
    [Key in StringKeys]: string;
} & {
    [Key in NumberKeys]: number;
}

const person: Person = {
    age: 25,
    firstName: "Mary",
    lastName: "Poppins"
}
```

[PLAYGROUND](https://www.typescriptlang.org/play/?#code/C4TwDgpgBAysBOBLAdgcwNIRAZygXigHIAzRebYAOQEMBbCQqAHyIBtqKb7CBuAKFCQolAK60ARhHiYc+ItVQN+A8NAAKU7AHtkcgN58oRqAG0ZUFLAQoMWbAF0AXFApI0-AL5QAZFAPHTc0tRCSkZB2dkMUl4Tz4+AGMdCihIch1nDXTdAn9jBQhnACYAVgAaQ2NScio6QqgAIgBZangQBoqA9k465wa1LTAwFGwGvg8gA)

## Unions
```TS
type AlwaysString = string | never; // always 'string'
type StringOrNumber = string | number | never; // 'never' always loses
type Any = any | string | number | never; // 'any' always wins
type Unknown = unknown | string | number | never; // 'unknown' always wins
type Any2 = any | unknown | string | number | never; // 'any' wins 'unknown'
```

## Conditional types and unions
```TS
/**
 * In the case of extending a union as a constraint, TypeScript will loop over each member of the union and return a union of its own.
 */
type NonOptional<T> = T extends null | undefined ? never : T;

type OptionalString = string | null | undefined;
type NonOptionalString = NonOptional<OptionalString>; // string

type OptionalManyTypes = string | boolean | number | null | undefined;
type NonOptionalManyTypes = NonOptional<OptionalManyTypes>; // string | number | boolean
```

[PLAYGROUND](https://www.typescriptlang.org/play/?#code/C4TwDgpgBAggNgdwIYgM4GVgCcCWA7AcygF4pVt8iAfKPCANwiwG4oB6NqJRFVKAcnK5C-AFChIUTMIIB5LADkArgFsARkxJkKhKDTyqNWPbQZNWHAXUZZ+XHmihwA9qgipx4aDDwgtSXxMhShMDdU19MxZ2Tn4AkDtuZEcEfA8JaABVPABrPGcEPC0lXPzCoJ1qWkMI0xsLWJK8grxEhz5UvHSvWF8AJn9Amiayoppg3X0a40j6mIF4u06+fhGWsVE2ACot0SgtqABJIuAAC2gAYyQ3KGcAMygIAA9gCDwAExCkKBKcZyLrlwoBd-kIkPhgAAaKAAFS86AuuDAwCgqTgcCczmcYFuNkeSAupygKgg4WM9ygZ2gv3+XA+UCwEGASiwAJ+eD+RQpOGAfBaADo9ls2J5JAp-rJkZzuAAeGEAPi0MMeLze7z4BnRJhK7wgd3wEHeUAA-HVNAAuWHMUSi6CS4DSuDSEKkCZVTUY4YfPUG97WjJQcV4e2O526UhBkP-WVRvDcMMEeUNbQyG0B2PcACy8ThkD4rsqJjUWLgEACoWmFa1Xt1+rofttgYlUujcGzvlz7i0kZbcbgMozbZzXlQSfmborZKLJbLeCAA)
