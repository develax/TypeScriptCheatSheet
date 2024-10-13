# Types & Interfaces

## Extenstion Types & Interfaces

**An interface can extend a type (with some caveats), and a type can extend an interface:**

```TS
type TState = {
    id: number;
}

interface IState {
    id: number;
}

interface IStateWithPop extends TState {
    population: number;
}

type TStateWithPop = IState & { 
    population: number; 
};
```

**Interfaces can't extend types that are uniuons:**

```TS
type TTypes = string | number;
interface ITest extends TTypes { } // Error: An interface can only extend an object type or intersection of object types with statically known members.
```

**Error checking:**

```TS
interface Person {
    name: string;
    age: string;
}

type TPerson = Person & { age: number; };  // no error, unusable type
const person: TPerson = {
    age: 1, // Type 'number' is not assignable to type 'never'.
    name: "Jane"
}

// `interface` and `extends` give a bit more error checking than `type` and `&`
interface IPerson extends Person {
    //      ~~~~~~~ Interface 'IPerson' incorrectly extends interface 'Person'.
    //                Types of property 'age' are incompatible.
    //                  Type 'number' is not assignable to type 'string'.
    age: number;
}
```

[Playground](https://www.typescriptlang.org/play/?#code/PQKhCgAIUhBA7SkCW8AuBTATgMwIYDGGSBeiGAHpvACaR5JoCeADsZABQDuyaAFpADOAewC2xUgDcMeNIICUAGnq16kZmxJlIlanW0p02fEQBcUEMHDgNxACoBlNLOIBeSAG8oSFDVOR4AFdRACNsAG5wAF9rVExcQmIASScXT28kZD8A4LCsSJjwOONEyBTnTAB1Xj4ABWEWHSoMWkFIRwriLx9IFgbAgBtZZGF4fyDQiOjrW3bUqpr6xvdytIAydJ6+lkHh0fHcqajI61AIaDKjBKI20ngAcjQmvXVWDDb+WXosYkD4ZECo0E5mgVlmdjsbza7kEaCwqAA5pAAD45Sb5IpXEzJOzvJ66Fo0NoQqGeSCFM4WSAAUWatERkAAqv8gZAQkx2qSNkksYlBBYrN0fMBgK9NDydk93AAiCWBNAARmlKMgsvgkoATNLIj0RWLiAB5eWSyAyo1oSVKlXS82a7UZfWXE3uDzHB2zW3y02eN0O4rXYgANTw8LwIQGGAAsnhGkKekgANrwPDifyw+HwBEAXX8cqeqM9aB1PkKutFdj4yA+b0gd3gwieYSaLB+gkEGDoPH4hni2IAhO6awA5FMd4Oh8NuTh5lWF+SQDYeAKjtNwhluilgKnUrBYYRYWt8DAEADWiJBlnAcZ7JSIkFq2BEiGvPWTqaEa8zxfj9ARGFXGYIt+5LWD04IPlgT7ehBUGLr+-5onk4TkshkB6vWOi7vuyh-IEghhhG+oOgQQJPGwkH7O0MGjN6L4+Hgf7+Aqyh6pCmj3BMeT3CgbT1k8eBtsgCLJpO6jCI6HEYNIWD3AAdA6r4rqqABSZAYNKDqlsKooAAb+tiOkqHQOkEq0hkIsg0hqCEvCQKI+7ENge4HgQR6ngynyIDptiGWQxlrDpfq8neSTUeQdJEvej40XRSB6vGAB+SXJQlly9qU9yhdFDyGCRu7HmgAwcqZkX6RlYVyQp2k-jVPhse8kDCDgvR7uRzCQPcDEYNxIbEKgJGiCwwyTvJtXxbVtX1R1nHYNxVYBA29CCcJBHEGg4mzPc6aIpVtVdQc6LATEURAA)
