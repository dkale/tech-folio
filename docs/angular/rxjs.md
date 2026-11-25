# RxJS

> RxJS represents asynchronous events and values as observable streams.

## Example

```typescript
import { debounceTime, distinctUntilChanged } from 'rxjs';

searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged()
).subscribe(query => performSearch(query));
```

Use RxJS for HTTP streams, event sequences, and complex time-based data flows.
