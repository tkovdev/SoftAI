# ADR-012: Angular State Management with Signals

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

Angular applications need effective state management for:
- Sharing data between components
- Caching API responses
- Managing application-wide state (auth, user profile, settings)
- Coordinating complex UI interactions
- Optimizing change detection performance

Angular 21 introduces **Signals** as a first-class reactivity primitive, changing the landscape of state management in Angular applications.

Key considerations:
- How to manage local component state
- How to share state between components
- How to integrate with async operations (HTTP calls)
- How to optimize performance and change detection
- Whether to use external state management libraries (NgRx, Akita, etc.)
- How to structure state for maintainability

Without clear state management strategy:
- Components become tightly coupled
- State gets duplicated across components
- Change detection becomes inefficient
- Code becomes hard to maintain and test

---

## Decision

We will use **Angular Signals** as the primary state management approach for Angular 21+, with **NgRx Signal Store** for complex, shared application state requiring advanced features like effects, optimistic updates, and entity management.

### State Management Tiers

**1. Local Component State** → **Signals**  
**2. Shared State (Simple)** → **Signal-based Services**  
**3. Shared State (Complex)** → **NgRx Signal Store**  
**4. Async Data** → **rxResource** (Signal + RxJS)

### Core Principles

1. **Signals-First**: Use Signals for all reactive state
2. **Immutability**: Always create new state, never mutate
3. **Single Source of Truth**: One authoritative source per piece of state
4. **Derived State**: Use `computed()` for calculations based on other signals
5. **Side Effects**: Use `effect()` sparingly, primarily for logging/debugging
6. **Service-Based**: Share state through injectable services, not component inputs
7. **Resource Pattern**: Use `rxResource` for HTTP data fetching

---

## Implementation Patterns

### 1. Local Component State (Signals)

**Use for**: Component-specific state that doesn't need sharing

**Example**:
```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-product-list',
  standalone: true,
  template: `
    <input [value]="searchTerm()" (input)="searchTerm.set($event.target.value)" />
    <p>Filtering {{ filteredProducts().length }} products</p>
    
    @for (product of filteredProducts(); track product.id) {
      <div class="product-item">{{ product.name }}</div>
    }
  `
})
export class ProductListComponent {
  // Writable signals
  products = signal<Product[]>([]);
  searchTerm = signal('');
  
  // Computed (derived) signals
  filteredProducts = computed(() => {
    const term = this.searchTerm().toLowerCase();
    return this.products().filter(p => 
      p.name.toLowerCase().includes(term)
    );
  });
  
  // Effects (use sparingly)
  constructor() {
    effect(() => {
      console.log('Search term changed:', this.searchTerm());
    });
  }
  
  addProduct(product: Product) {
    // Immutable update
    this.products.update(current => [...current, product]);
  }
  
  removeProduct(id: number) {
    this.products.update(current => 
      current.filter(p => p.id !== id)
    );
  }
}
```

**Best Practices**:
- Use `signal()` for mutable state
- Use `computed()` for derived state
- Use `.set()` to replace entire value
- Use `.update()` to transform based on current value
- Avoid `effect()` for business logic (use for debugging/logging only)

### 2. Shared State via Signal Services

**Use for**: Application-wide state (auth, user preferences, shopping cart)

**Example - Auth Service**:
```typescript
import { Injectable, signal, computed, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs/operators';

export interface User {
  id: string;
  email: string;
  name: string;
  roles: string[];
}

@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  
  // Private writable signals
  private accessTokenSignal = signal<string | null>(null);
  private currentUserSignal = signal<User | null>(null);
  private isLoadingSignal = signal(false);
  
  // Public readonly signals
  readonly accessToken = this.accessTokenSignal.asReadonly();
  readonly currentUser = this.currentUserSignal.asReadonly();
  readonly isLoading = this.isLoadingSignal.asReadonly();
  
  // Computed signals
  readonly isAuthenticated = computed(() => !!this.accessTokenSignal());
  readonly isAdmin = computed(() => 
    this.currentUserSignal()?.roles.includes('Admin') ?? false
  );
  
  login(email: string, password: string) {
    this.isLoadingSignal.set(true);
    
    return this.http.post<{ accessToken: string; user: User }>(
      '/api/auth/login',
      { email, password }
    ).pipe(
      tap(response => {
        this.accessTokenSignal.set(response.accessToken);
        this.currentUserSignal.set(response.user);
        this.isLoadingSignal.set(false);
      })
    );
  }
  
  logout() {
    return this.http.post('/api/auth/logout', {}).pipe(
      tap(() => {
        this.accessTokenSignal.set(null);
        this.currentUserSignal.set(null);
      })
    );
  }
  
  updateUser(updates: Partial<User>) {
    this.currentUserSignal.update(current => 
      current ? { ...current, ...updates } : null
    );
  }
}
```

**Usage in Component**:
```typescript
@Component({
  selector: 'app-header',
  template: `
    @if (authService.isAuthenticated()) {
      <span>Welcome, {{ authService.currentUser()?.name }}</span>
      <button (click)="logout()">Logout</button>
    } @else {
      <a routerLink="/login">Login</a>
    }
  `
})
export class HeaderComponent {
  authService = inject(AuthService);
  
  logout() {
    this.authService.logout().subscribe();
  }
}
```

### 3. NgRx Signal Store (Complex State)

**Use for**: Complex shared state with effects, entity management, or advanced patterns

**Installation**:
```bash
npm install @ngrx/signals
```

**Example - Products Store**:
```typescript
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { withEntities, addEntity, updateEntity, removeEntity } from '@ngrx/signals/entities';
import { inject } from '@angular/core';
import { computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs';

export interface Product {
  id: number;
  name: string;
  price: number;
  categoryId: number;
}

interface ProductsState {
  isLoading: boolean;
  filter: string;
  selectedCategoryId: number | null;
}

export const ProductsStore = signalStore(
  { providedIn: 'root' },
  
  // Entity management
  withEntities<Product>(),
  
  // Additional state
  withState<ProductsState>({
    isLoading: false,
    filter: '',
    selectedCategoryId: null
  }),
  
  // Computed selectors
  withComputed((store) => ({
    filteredProducts: computed(() => {
      const filter = store.filter().toLowerCase();
      const categoryId = store.selectedCategoryId();
      
      let products = store.entities();
      
      if (filter) {
        products = products.filter(p => 
          p.name.toLowerCase().includes(filter)
        );
      }
      
      if (categoryId !== null) {
        products = products.filter(p => p.categoryId === categoryId);
      }
      
      return products;
    }),
    
    productCount: computed(() => store.entities().length),
    
    totalValue: computed(() => 
      store.entities().reduce((sum, p) => sum + p.price, 0)
    )
  })),
  
  // Methods (actions)
  withMethods((store, http = inject(HttpClient)) => ({
    loadProducts() {
      patchState(store, { isLoading: true });
      
      return http.get<Product[]>('/api/v1/products').pipe(
        tap(products => {
          patchState(store, { 
            entities: products,
            isLoading: false 
          });
        })
      );
    },
    
    addProduct(product: Product) {
      patchState(store, addEntity(product));
    },
    
    updateProduct(id: number, changes: Partial<Product>) {
      patchState(store, updateEntity({ id, changes }));
    },
    
    removeProduct(id: number) {
      patchState(store, removeEntity(id));
    },
    
    setFilter(filter: string) {
      patchState(store, { filter });
    },
    
    setCategory(categoryId: number | null) {
      patchState(store, { selectedCategoryId: categoryId });
    },
    
    clearFilters() {
      patchState(store, { 
        filter: '', 
        selectedCategoryId: null 
      });
    }
  }))
);
```

**Usage in Component**:
```typescript
@Component({
  selector: 'app-products',
  template: `
    <input 
      [value]="store.filter()" 
      (input)="store.setFilter($any($event.target).value)" 
      placeholder="Search products" />
    
    <select 
      [value]="store.selectedCategoryId() ?? ''" 
      (change)="store.setCategory(+$any($event.target).value)">
      <option value="">All Categories</option>
      <option value="1">Electronics</option>
      <option value="2">Books</option>
    </select>
    
    @if (store.isLoading()) {
      <p>Loading...</p>
    }
    
    <p>Showing {{ store.filteredProducts().length }} of {{ store.productCount() }} products</p>
    <p>Total value: ${{ store.totalValue() }}</p>
    
    @for (product of store.filteredProducts(); track product.id) {
      <div class="product-item">
        <h3>{{ product.name }}</h3>
        <p>{{ product.price | currency }}</p>
        <button (click)="store.removeProduct(product.id)">Remove</button>
      </div>
    }
  `
})
export class ProductsComponent {
  store = inject(ProductsStore);
  
  ngOnInit() {
    this.store.loadProducts().subscribe();
  }
}
```

### 4. Async Data Fetching with rxResource

**Use for**: Loading data from HTTP endpoints with automatic lifecycle management

**Example**:
```typescript
import { Component, signal, resource } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { inject } from '@angular/core';

@Component({
  selector: 'app-product-detail',
  template: `
    @if (productResource.isLoading()) {
      <p>Loading...</p>
    } @else if (productResource.error()) {
      <p>Error: {{ productResource.error() }}</p>
    } @else if (productResource.value(); as product) {
      <h2>{{ product.name }}</h2>
      <p>{{ product.description }}</p>
      <p>Price: {{ product.price | currency }}</p>
      <button (click)="reload()">Reload</button>
    }
  `
})
export class ProductDetailComponent {
  private http = inject(HttpClient);
  
  productId = signal(1);
  
  productResource = resource({
    request: () => ({ id: this.productId() }),
    loader: ({ request }) => 
      this.http.get<Product>(`/api/v1/products/${request.id}`)
  });
  
  reload() {
    this.productResource.reload();
  }
  
  changeProduct(id: number) {
    this.productId.set(id); // Automatically triggers reload
  }
}
```

**Benefits**:
- Automatic loading state management
- Automatic cleanup on component destroy
- Automatic refetch when dependencies change
- Built-in error handling

---

## State Organization Patterns

### Feature-Based Structure

```
src/app/
├── core/
│   └── services/
│       ├── auth.service.ts         // Signal-based service
│       └── user-preferences.service.ts
│
├── features/
│   ├── products/
│   │   ├── store/
│   │   │   └── products.store.ts   // NgRx Signal Store
│   │   ├── services/
│   │   │   └── product.service.ts  // HTTP service
│   │   └── components/
│   │       ├── product-list/
│   │       │   └── product-list.component.ts  // Local signals
│   │       └── product-detail/
│   │           └── product-detail.component.ts
│   │
│   └── orders/
│       ├── store/
│       │   └── orders.store.ts
│       └── components/
│           └── order-list/
│
└── shared/
    └── services/
        └── notification.service.ts  // Signal-based service
```

### State Hierarchy

```
Global State (NgRx Signal Store)
├── Products Store
├── Orders Store
└── Cart Store

Shared Services (Signal-based)
├── Auth Service
├── User Preferences Service
└── Notification Service

Component State (Local Signals)
├── Product List Component
├── Product Form Component
└── Order Detail Component
```

---

## Best Practices

### 1. Signal Naming Conventions

```typescript
// ✅ Good: Descriptive names
const products = signal<Product[]>([]);
const isLoading = signal(false);
const selectedId = signal<number | null>(null);

// ❌ Bad: Vague or misleading names
const data = signal([]);
const flag = signal(false);
const temp = signal(null);
```

### 2. Immutable Updates

```typescript
// ✅ Good: Immutable update
products.update(current => [...current, newProduct]);

// ❌ Bad: Mutation
products().push(newProduct);  // Won't trigger updates!
```

### 3. Computed for Derived State

```typescript
// ✅ Good: Use computed for derived state
const filteredProducts = computed(() => 
  products().filter(p => p.price > minPrice())
);

// ❌ Bad: Manual recalculation
filterProducts() {
  this.filteredProducts.set(
    this.products().filter(p => p.price > this.minPrice())
  );
}
```

### 4. Limit Effect Usage

```typescript
// ✅ Good: Effect for side effects only
effect(() => {
  console.log('User logged in:', currentUser());
  analytics.track('user_login', currentUser());
});

// ❌ Bad: Effect for business logic
effect(() => {
  if (searchTerm()) {
    this.products.set(this.filterProducts());  // Use computed instead!
  }
});
```

### 5. Expose Readonly Signals

```typescript
// ✅ Good: Private writable, public readonly
export class CartService {
  private itemsSignal = signal<CartItem[]>([]);
  readonly items = this.itemsSignal.asReadonly();
  
  addItem(item: CartItem) {
    this.itemsSignal.update(items => [...items, item]);
  }
}

// ❌ Bad: Exposing writable signal
export class CartService {
  items = signal<CartItem[]>([]);  // Can be mutated from outside!
}
```

### 6. Combine Signals with RxJS When Needed

```typescript
// ✅ Good: Use toObservable/toSignal for interop
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

export class ProductService {
  private searchTermSignal = signal('');
  
  // Convert signal to observable for HTTP call
  products$ = toObservable(this.searchTermSignal).pipe(
    debounceTime(300),
    switchMap(term => this.http.get(`/api/products?search=${term}`))
  );
  
  // Convert observable back to signal
  products = toSignal(this.products$, { initialValue: [] });
}
```

---

## Migration from RxJS/NgRx

**Gradual Migration Strategy**:
1. Start with new features using Signals
2. Migrate simple services to Signal-based state
3. Keep existing RxJS/NgRx code until convenient to migrate
4. Use interop utilities (`toSignal`, `toObservable`) for compatibility

**Don't need to migrate immediately** - Signals and RxJS coexist well.

---

## Consequences

### Positive Consequences
- **Performance**: Zone-less change detection, better performance
- **Simplicity**: Easier to understand than RxJS for simple state
- **Type Safety**: Better type inference than observables
- **Less Boilerplate**: No subscriptions to manage
- **Modern**: Aligns with Angular's future direction
- **Flexible**: Can mix with RxJS when needed
- **Testing**: Easier to test than observable-based state

### Negative Consequences
- **Learning Curve**: Team needs to learn Signals paradigm
- **New Patterns**: Different from traditional RxJS patterns
- **Limited Ecosystem**: Fewer third-party libraries support Signals (yet)
- **Migration Effort**: Existing RxJS code may need updates eventually

### Neutral Considerations
- Can coexist with RxJS and traditional state management
- Need to establish team conventions for when to use which approach
- Documentation and examples still evolving
- NgRx Signal Store is newer, less mature than full NgRx

---

## Alternatives Considered

### Alternative 1: Full NgRx (Store + Effects + Entity)
- **Description**: Use traditional NgRx with Redux pattern for all state
- **Pros**: Battle-tested, extensive ecosystem, powerful dev tools, excellent for complex apps
- **Cons**: Heavy boilerplate, steep learning curve, overkill for simple apps, more code to maintain
- **Why rejected**: Signals provide simpler solution for most use cases; NgRx Signal Store available for complex scenarios

### Alternative 2: RxJS + BehaviorSubject Services
- **Description**: Use RxJS Observables and BehaviorSubjects for all state
- **Pros**: Well-understood, powerful, works with existing code
- **Cons**: Manual subscription management, more verbose, doesn't leverage new Angular features
- **Why rejected**: Signals are Angular's future, offer better performance and developer experience

### Alternative 3: Akita
- **Description**: Use Akita state management library
- **Pros**: Simpler than NgRx, good documentation, entity management
- **Cons**: Third-party dependency, smaller ecosystem than NgRx, less future-proof than Signals
- **Why rejected**: Signals are first-party, better long-term investment

### Alternative 4: Component State Only (No Shared State)
- **Description**: Pass all data via @Input/@Output
- **Pros**: Simple, explicit data flow
- **Cons**: Prop drilling, difficult to share state across distant components, lots of boilerplate
- **Why rejected**: Not practical for real applications with complex state sharing needs

---

## Implementation Notes

### Key Packages

```json
{
  "dependencies": {
    "@angular/core": "^21.0.0",
    "@ngrx/signals": "^18.0.0",  // Optional, for complex state
    "rxjs": "^7.8.0"
  }
}
```

### Testing Signals

```typescript
describe('AuthService', () => {
  let service: AuthService;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [AuthService]
    });
    service = TestBed.inject(AuthService);
  });
  
  it('should update authentication state on login', (done) => {
    // Signals can be tested synchronously
    expect(service.isAuthenticated()).toBe(false);
    
    service.login('user@example.com', 'password').subscribe(() => {
      expect(service.isAuthenticated()).toBe(true);
      expect(service.currentUser()).toBeTruthy();
      done();
    });
  });
});
```

---

## References

- [Angular Signals Documentation](https://angular.dev/guide/signals)
- [NgRx Signal Store](https://ngrx.io/guide/signals)
- [Angular rxResource](https://angular.dev/api/core/rxjs-interop/rxResource)
- [Signal-based Components](https://angular.dev/guide/components)
- [Managing State with Signals](https://angular.dev/guide/signals#managing-state-with-signals)
- [Signals vs RxJS](https://angular.dev/guide/signals/rxjs-interop)
