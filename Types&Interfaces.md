# Types & Interfaces

An  interface  can extend a  type  (with some caveats), and a type  can extend an  interface:

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

Interfaces can't extend types that are uniuons:

```TS
type TTypes = string | number;
interface ITest extends TTypes { } // Error: An interface can only extend an object type or intersection of object types with statically known members.
```
