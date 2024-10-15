## ShallowReadonly<T>

```TS
type ShallowReadonly<T> = {
    readonly [key in keyof T]: T[key];
};

/**
 * Test:
 */
type Point = { x: number; y: number };

type Rect = {
    p0: Point;
    p1: Point;
    n: number;
}

const rect: ShallowReadonly<Rect> = {
    p0: { x: 0, y: 0 },
    p1: { x: 1, y: 1 },
    n: 1
};

rect.n = 2; // ERROR: Cannot assign to 'n' because it is a read-only property.
rect.p0 = { x: 0, y: 0 }; // ERROR: Cannot assign to 'p0' because it is a read-only property.
rect.p0.x = 0; // ok!!!
```

## DeepReadonly<T>

```TS
type DeepReadonly<T> = {
    readonly [key in keyof T]: T[key] extends object ?  DeepReadonly<T[key]> : T[key];
};

/**
 * Test:
 */
const drRect: DeepReadonly<Rect> = {
    p0: { x: 0, y: 0 },
    p1: { x: 1, y: 1 },
    n: 1
};

drRect.p0.x = 0; // ERROR: Cannot assign to 'x' because it is a read-only property.
```

[Playground](https://www.typescriptlang.org/play/?#code/PQKhCgAIUhlALAhgG2QewO4CUCmiAmaAdsgJ4A8AKgHxQjDgAupADjnEqprgcWVdUgBeSAG8okSQCc8hEqUgBtANY4FASyKRVpNADNIlALoAuQyrVGA3OAC+N8KAjRDOAM6MTdBszaQACmiajMJikAAeZkQArgC2AEY4UlaQpFFxiVKQ9uBMrOy4AMYhIuKSkiwADGaBwTblkCwAjDVBRIz15UTpCUk2trmFxB6QMsVmCCjo2LJ8FEWMgqUSFdVhkZCVADSpZpXZWyuNLetmTTtpkE0HR91Xdg5jjAB0WiIATCnAwJAAolhYADyWDMAGFEEQiGgQog3G51ABzLSMNCQADkRDRkEShUQ0Tc7HUIXUbkgiFGsgAtHNGlI0GwpMxnuAns8qqFRBE9hc9tkvj9-kCQZBwZDoWS4Yjkai0VUsTi8QTIETlaTyTICNT5LT6UkmSycMU2ZVnuFQpV+ZA0MoAIR23JOOiQAAiOBwLB4cn4NG8eT8rvdnrmAg5Rw1XoUFg0Wh0+kMpnMOiMkBw4UYOCI+FJaHiACtDSEAPySAMe2byKhRoyCMyUKv9ByOlyUdyeX1DIgjfBSBZmUtBisLJZiI5VMycjbbXabG4NZrjrlXHlXWddM4PXLdhbG03my2C4FgiFQmGSpGQFHo8Lyw2KwnEtUUzU0lh0hlMoA)
