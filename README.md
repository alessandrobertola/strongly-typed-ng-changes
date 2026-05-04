# NgChanges

A strongly-typed replacement for Angular's `SimpleChanges`, giving you full type safety inside `ngOnChanges`.

## The problem

Angular's built-in `SimpleChanges` is untyped — `previousValue` and `currentValue` are both `any`, so typos and type mismatches go unnoticed at compile time.

## The solution

```typescript
type AnyFunction = (...args: unknown[]) => unknown;

type MarkFunctionProperties<Component> = {
  [Key in keyof Component]: Component[Key] extends AnyFunction ? never : Key;
};
type ExcludeFunctionPropertyNames<T> = MarkFunctionProperties<T>[keyof T];
type ExcludeFunctions<T> = Pick<T, ExcludeFunctionPropertyNames<T>>;

export type NgChanges<Component, Props = ExcludeFunctions<Component>> = {
  [Key in keyof Props]: {
    previousValue: Props[Key];
    currentValue: Props[Key];
    firstChange: boolean;
    isFirstChange(): boolean;
  };
};
```

`NgChanges<T>` automatically excludes methods from `T` and types each property's `previousValue` / `currentValue` correctly.

## Usage

```typescript
import { Component, Input, OnChanges } from '@angular/core';
import { NgChanges } from './ng-changes';

@Component({ selector: 'app-user', template: '' })
export class UserComponent implements OnChanges {
  @Input() userId!: string;
  @Input() role!: 'admin' | 'viewer';

  ngOnChanges(changes: NgChanges<UserComponent>): void {
    if (changes.userId) {
      // ✅ currentValue is typed as string
      console.log(changes.userId.currentValue);
    }

    if (changes.role) {
      // ✅ currentValue is typed as 'admin' | 'viewer'
      console.log(changes.role.currentValue);
    }
  }
}
```

## How it works

| Step | What it does |
|---|---|
| `AnyFunction` | Matches any function signature |
| `MarkFunctionProperties` | Maps each key to itself or `never`, depending on whether it's a function |
| `ExcludeFunctions` | Picks only the non-function keys (i.e. `@Input` properties) |
| `NgChanges` | Builds a `SimpleChanges`-like object typed around those keys |

## License

MIT
