# Angular Signals

> Angular Signals provide fine-grained reactive state for templates and application logic.

## Overview

Angular Signals track values and notify consumers when those values change. Computed signals derive values, while effect functions perform side effects.

## Example

```typescript
import { computed, signal } from '@angular/core';

const count = signal(0);
const doubled = computed(() => count() * 2);

count.set(3);
console.log(doubled()); // 6
```

## Best Practices

- Use signals for synchronously derived UI state.
- Keep effects narrowly scoped.
- Use observables for asynchronous event streams when appropriate.
- Prefer immutable updates for complex objects.
