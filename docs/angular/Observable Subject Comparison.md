# Observable vs Subject and Related RxJS Types

This reference compares `Observable`, `Subject`, `BehaviorSubject`, `ReplaySubject`, `AsyncSubject`, and Angular's `EventEmitter`, including their behavior, delivery model, examples, and typical use cases.

## Table of Contents

- [Overview](#overview)
- [Key Differences](#key-differences)
  - [Observable](#observable)
  - [Subject](#subject)
  - [BehaviorSubject](#behaviorsubject)
  - [ReplaySubject](#replaysubject)
  - [AsyncSubject](#asyncsubject)
  - [EventEmitter](#eventemitter)
- [Comparison Table](#comparison-table)
- [When to Use Which](#when-to-use-which)
- [Summary](#summary)

## Overview

| Type | Description |
| --- | --- |
| **Observable** | A data stream that emits multiple values over time and can be subscribed to for reactive programming. |
| **Subject** | A special type of `Observable` that allows both publishing and subscribing to events or data streams. |
| **BehaviorSubject** | A `Subject` that stores the latest value and immediately emits it to new subscribers. It requires an initial value. |
| **ReplaySubject** | A `Subject` that replays a specified number of previously emitted values, or all values, to new subscribers. |
| **AsyncSubject** | A `Subject` that emits only the last value from the source, and only after the source completes. |
| **EventEmitter** | An Angular class used for custom event handling and communication between components through emitting and subscribing to events. |

[↑ Back to Top](#table-of-contents)

## Key Differences

### Observable

- Base class for all reactive streams.
- Acts only as an observable.
- **Unicast:** Each subscription creates its own producer.
- Can be classified as a **hot** or **cold** observable.

```text
Observable
  └─ Subscriber A -> Producer A
  └─ Subscriber B -> Producer B
```

[↑ Back to Top](#table-of-contents)

### Subject

- Extends `Observable`.
- Acts as both:
  - An **observer** that can receive values.
  - An **observable** that can emit values.
- **Multicast:** One producer serves multiple subscribers.
- Always a **hot observable**.
- Implements:
  - `next()`
  - `error()`
  - `complete()`
- Commonly used for component-to-component communication.

```text
Producer
    │
    ▼
 Subject
 ├── Subscriber A
 ├── Subscriber B
 └── Subscriber C
```

[↑ Back to Top](#table-of-contents)

### BehaviorSubject

#### Characteristics

- Extends `Subject`.
- Requires an **initial value**.
- Caches the **latest emitted value**.
- New subscribers immediately receive the current value.

```typescript
const subject = new BehaviorSubject<number>(0);

subject.next(1);
subject.next(2);

// New subscriber receives 2 immediately
subject.subscribe(value => console.log(value));
```

#### Use Cases

- Application state management
- User authentication status
- Shared data services

[↑ Back to Top](#table-of-contents)

### ReplaySubject

#### Characteristics

- Extends `Subject`.
- Can cache a specified number of emitted values.
- New subscribers receive all cached values.

```typescript
const subject = new ReplaySubject<number>(2);

subject.next(1);
subject.next(2);
subject.next(3);

// New subscriber receives:
// 2
// 3
```

#### Use Cases

- Chat message history
- Event history tracking
- Audit logs

[↑ Back to Top](#table-of-contents)

### AsyncSubject

#### Characteristics

- Extends `Subject`.
- Emits only the **last value**.
- Emits only when the observable **completes**.

```typescript
const subject = new AsyncSubject<number>();

subject.next(1);
subject.next(2);
subject.next(3);

subject.complete();

// Subscribers receive only:
// 3
```

#### Use Cases

- One-time asynchronous operations
- Final result notifications
- API completion scenarios

[↑ Back to Top](#table-of-contents)

### EventEmitter

#### Characteristics

- Extends `Subject`.
- Provides an additional `emit()` method.
- Primarily used in Angular for child-to-parent component communication.

```typescript
@Output()
save = new EventEmitter<string>();

this.save.emit('Data saved');
```

[↑ Back to Top](#table-of-contents)

## Comparison Table

| Feature | Observable | Subject | BehaviorSubject | ReplaySubject | AsyncSubject | EventEmitter |
| --- | --- | --- | --- | --- | --- | --- |
| Extends `Observable` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Acts as observer | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Multicast | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Requires initial value | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| Stores latest value | ❌ | ❌ | ✅ | ✅ | ✅ (last only) | ❌ |
| Replays previous values | ❌ | ❌ | Latest only | ✅ | Last after completion | ❌ |
| Emits on completion only | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Typical Angular use | Data streams | Component communication | State management | History and event replay | Final result delivery | `@Output()` events |

[↑ Back to Top](#table-of-contents)

## When to Use Which

| Scenario | Recommended Type |
| --- | --- |
| Simple data stream | `Observable` |
| Component-to-component communication | `Subject` |
| Shared application state | `BehaviorSubject` |
| Event history for new subscribers | `ReplaySubject` |
| Only the final value after completion | `AsyncSubject` |
| Angular `@Output()` events | `EventEmitter` |

[↑ Back to Top](#table-of-contents)

## Summary

- Use **`Observable`** when you only need a data source.
- Use **`Subject`** when multiple subscribers need the same stream and you want to manually emit values.
- Use **`BehaviorSubject`** when subscribers should immediately receive the latest value.
- Use **`ReplaySubject`** when subscribers need access to previous emissions.
- Use **`AsyncSubject`** when only the final value matters after completion.
- Use **`EventEmitter`** for Angular component event communication.

[↑ Back to Top](#table-of-contents)
