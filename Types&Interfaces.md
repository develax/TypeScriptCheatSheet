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
