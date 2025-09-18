# Programming Principles

## Fail Fast
- Detect invalid state immediately.  
- Throw errors early instead of allowing bad data to propagate.  
- Prevents secondary failures that are harder to trace.  

### Example
```js
// Bad - silently patches invalid state
function processOrder(order) {
  if (!order) return 0; // hides null problem
  if (!order.items) order.items = []; // tolerates broken state
  return order.items.reduce((t, i) => t + i.price, 0);
}

// Good - fail fast
function processOrder(order) {
  if (!order) throw new Error("Order cannot be null");
  if (!Array.isArray(order.items)) {
    throw new Error("Order must have an 'items' array");
  }
  return order.items.reduce((t, i) => t + i.price, 0);
}
```

---

## Return Early
- Return immediately when a result is known.  
- Avoid mutable variables that accumulate a final return value.  
- Multiple returns make branching clearer.  

### Example
```js
// Bad - mutable variable
function getDiscount(user) {
  let discount = 0;
  if (!user) {
    discount = 0;
  } else if (user.isStudent) {
    discount = 0.2;
  } else if (user.isMember) {
    discount = 0.1;
  }
  return discount;
}

// Good - return early
function getDiscount(user) {
  if (!user) return 0;
  if (user.isStudent) return 0.2;
  if (user.isMember) return 0.1;
  return 0;
}
```

---

## Never Nester
- Avoid more than **3 levels of indentation**:  
  - Function → Conditional → Loop.  
- More nesting causes unreadable “arrowhead code.”  
- Use guard clauses (`return`, `continue`, `throw`) or extract smaller functions.  

### Example
```js
// Bad - deeply nested (5 levels)
function findActiveUsernames(users) {
  const names = [];
  if (users && users.length > 0) {
    for (const user of users) {
      if (user.isActive) {
        if (user.profile) {
          if (user.profile.username) {
            names.push(user.profile.username);
          }
        }
      }
    }
  }
  return names;
}

// Good - flat (3 levels max)
function findActiveUsernames(users) {
  if (!users?.length) return [];

  const names = [];
  for (const user of users) {
    if (!user.isActive) continue;
    if (!user.profile?.username) continue;
    names.push(user.profile.username);
  }
  return names;
}
```

---

## Single Responsibility Principle (SRP)
- A module/class/function should have **one reason to change**.  
- Controllers handle HTTP mapping, services handle business logic, repositories handle data.  

---

## Separation of Concerns (SoC)
- Different concerns → different layers.  
- Don’t mix validation, routing, business rules, DB access in the same place.  

---

## DRY (Don’t Repeat Yourself)
- Eliminate logic duplication via utilities and shared modules.  
- Don’t over‑abstract prematurely.  

---

## YAGNI (You Aren’t Gonna Need It)
- Don’t add features, hooks, or abstractions until required.  
- Avoid speculative engineering.  

---

## KISS (Keep It Simple, Stupid)
- Prefer simple code over clever tricks.  
- Readability > micro‑optimizations.  

---

## Immutability Preference
- Default to `const`.  
- Transform values instead of mutating them.  

### Example
```js
// Mutable
let total = 0;
for (let i = 0; i < items.length; i++) {
  total += items[i].price;
}

// Immutable/functional
const total = items.reduce((sum, i) => sum + i.price, 0);
```

---

## Composition Over Inheritance
- Prefer combining behaviors via composition instead of class hierarchies.  
- “Has‑a” is usually better than “is‑a.”  
- Looser coupling, easier testing, greater flexibility.  

### Example
```js
// Bad - inheritance misuse
class Logger {
  log(msg) { console.log(msg); }
}

class UserService extends Logger { // ❌ UserService is not a Logger
  createUser(data) {
    this.log("Creating user");
  }
}

// Good - composition
class Logger {
  log(msg) { console.log(msg); }
}

class UserService {
  constructor(logger) {
    this.logger = logger;
  }

  createUser(data) {
    this.logger.log("Creating user");
  }
}

const userService = new UserService(new Logger());
```

---

## Checklist
- [ ] Fail Fast → validate and crash early on invalid state.  
- [ ] Return Early → simplify branches, avoid temp variables.  
- [ ] Never Nester → max 3 indent levels.  
- [ ] Apply SRP + SoC for modular code.  
- [ ] Keep DRY but avoid premature abstraction.  
- [ ] Follow YAGNI → no speculative features.  
- [ ] Follow KISS → simple code preferred.  
- [ ] Prefer immutability for clarity and safety.  
- [ ] Favor composition over inheritance.  

---

## References
- [Martin Fowler — Fail Fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)  
- [Return Early Pattern](https://medium.com/swlh/return-early-pattern-3d18a41bba8)  
- [Never Nester (Tim Ottinger Talk)](https://www.youtube.com/watch?v=CFRhGnuXG-4)  
- [Robert C. Martin — SRP, SoC, SoLID](https://www.oodesign.com/single-responsibility-principle.html)  
- [YAGNI](https://martinfowler.com/bliki/Yagni.html)  
- [KISS Principle](https://en.wikipedia.org/wiki/KISS_principle)  
- [DRY Principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
