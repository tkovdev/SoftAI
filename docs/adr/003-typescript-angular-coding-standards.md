# ADR-003: TypeScript/Angular Coding Standards

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

We need to establish consistent TypeScript and Angular coding standards for the Angular 21 frontend using PrimeNG components to ensure:
- Code consistency across all agents and developers
- Proper usage of Angular 21 features (standalone components, signals)
- Effective use of PrimeNG component library
- Automated enforcement through ESLint and Prettier
- Alignment with Angular's official style guide
- Type safety with strict TypeScript

Without clear standards, agents may produce inconsistent code or misuse Angular/PrimeNG features.

---

## Decision

We will adopt **Angular's Official Style Guide** with TypeScript strict mode, Angular 21 standalone components, signals for state management, and PrimeNG for UI components.

### Naming Conventions

**Classes, Interfaces, Types, Enums**:
```typescript
// PascalCase
export class ProductService { }
export interface Product { }
export type ProductId = string | number;
export enum OrderStatus { Pending, Confirmed, Shipped }
```

**Component Classes**:
```typescript
// PascalCase with suffix
export class ProductListComponent { }
export class OrderDetailComponent { }
export class UserProfileComponent { }
```

**Variables, Functions, Methods, Properties**:
```typescript
// camelCase
const productId = '123';
let isLoading = false;

function calculateTotal(items: CartItem[]): number { }

class ProductService {
  getProducts(): Observable<Product[]> { }
  private loadFromCache(): void { }
}
```

**Constants**:
```typescript
// UPPER_SNAKE_CASE or camelCase for objects
export const MAX_RETRY_ATTEMPTS = 3;
export const API_TIMEOUT_MS = 30000;

// camelCase for complex objects
export const apiConfig = {
  baseUrl: 'https://api.example.com',
  timeout: 30000,
  retries: 3
} as const;
```

**Private Members (with #)**:
```typescript
// Use # for truly private members (TypeScript 5.0+)
export class ProductService {
  #cache = new Map<string, Product>();
  #apiKey = '';
  
  // Or use private keyword (still accessible in compiled JS)
  private logger = inject(LoggerService);
}
```

**Boolean Variables**:
```typescript
// Prefix with is, has, should, can
let isLoading = false;
const hasPermission = user.checkPermission('edit');
const shouldRetry = attempts < MAX_RETRY_ATTEMPTS;
const canSubmit = form.valid && !isLoading;
```

### File Organization

**File Naming**:
```
product-list.component.ts
product-list.component.html
product-list.component.scss
product-list.component.spec.ts

product.service.ts
product.service.spec.ts

product.model.ts
product.interface.ts

auth.guard.ts
logging.interceptor.ts
```

**Folder Structure**:
```
src/
├── app/
│   ├── core/                    # Singleton services, guards, interceptors
│   │   ├── services/
│   │   ├── guards/
│   │   └── interceptors/
│   ├── shared/                  # Reusable components, directives, pipes
│   │   ├── components/
│   │   ├── directives/
│   │   └── pipes/
│   ├── features/                # Feature modules
│   │   ├── products/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   └── products.routes.ts
│   │   └── orders/
│   └── app.config.ts            # App configuration
├── assets/
├── environments/
└── styles/
```

**One Component Per File**: Each component, service, directive, pipe in its own file

### Angular 21 Standalone Components

**Always Use Standalone Components**:
```typescript
import { Component, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ButtonModule } from 'primeng/button';
import { TableModule } from 'primeng/table';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [CommonModule, ButtonModule, TableModule],
  templateUrl: './product-list.component.html',
  styleUrl: './product-list.component.scss'
})
export class ProductListComponent {
  products = signal<Product[]>([]);
  loading = signal(false);
  
  // Computed values
  productCount = computed(() => this.products().length);
  hasProducts = computed(() => this.products().length > 0);
}
```

**Inject Dependencies with inject()**:
```typescript
import { Component, inject } from '@angular/core';
import { ProductService } from './product.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  // ...
})
export class ProductListComponent {
  // ✅ Use inject() function (Angular 14+)
  private productService = inject(ProductService);
  private router = inject(Router);
  
  // ❌ Avoid constructor injection in new code
  // constructor(private productService: ProductService) { }
}
```

### Signals for State Management

**Use Signals for Component State**:
```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  // ...
})
export class ProductDetailComponent {
  // Writable signals for state
  product = signal<Product | null>(null);
  isEditing = signal(false);
  quantity = signal(1);
  
  // Computed signals (derived state)
  totalPrice = computed(() => {
    const p = this.product();
    return p ? p.price * this.quantity() : 0;
  });
  
  canCheckout = computed(() => {
    return this.product() !== null && this.quantity() > 0;
  });
  
  // Effects for side effects
  constructor() {
    effect(() => {
      // Runs whenever product() changes
      console.log('Product changed:', this.product());
    });
  }
  
  // Update signals
  incrementQuantity(): void {
    this.quantity.update(q => q + 1);
  }
  
  setProduct(product: Product): void {
    this.product.set(product);
  }
}
```

**Signal-based Services**:
```typescript
import { Injectable, signal, computed } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CartService {
  // Private writable signal
  private itemsSignal = signal<CartItem[]>([]);
  
  // Public read-only access
  items = this.itemsSignal.asReadonly();
  
  // Computed values
  totalItems = computed(() => this.itemsSignal().length);
  totalPrice = computed(() => 
    this.itemsSignal().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  
  addItem(item: CartItem): void {
    this.itemsSignal.update(items => [...items, item]);
  }
  
  removeItem(itemId: string): void {
    this.itemsSignal.update(items => items.filter(i => i.id !== itemId));
  }
}
```

### RxJS Usage Patterns

**Use rxResource for Data Fetching** (Angular 19+):
```typescript
import { Component, resource } from '@angular/core';

@Component({
  selector: 'app-product-list',
  standalone: true,
  // ...
})
export class ProductListComponent {
  private productService = inject(ProductService);
  
  // rxResource handles loading, error, and success states
  productsResource = resource({
    loader: () => this.productService.getProducts()
  });
  
  // Access in template: productsResource.value(), productsResource.isLoading()
}
```

**Manage Subscriptions Properly**:
```typescript
import { Component, OnInit, OnDestroy, inject } from '@angular/core';
import { Subject, takeUntil } from 'rxjs';

@Component({
  selector: 'app-product-search',
  standalone: true,
  // ...
})
export class ProductSearchComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  private productService = inject(ProductService);
  
  ngOnInit(): void {
    // ✅ Use takeUntil to automatically unsubscribe
    this.productService.getProducts()
      .pipe(takeUntil(this.destroy$))
      .subscribe(products => {
        this.products.set(products);
      });
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Or Use Async Pipe** (preferred when possible):
```typescript
// Component
products$ = this.productService.getProducts();

// Template
<div *ngFor="let product of products$ | async">
  {{ product.name }}
</div>
```

### PrimeNG Component Usage

**Import PrimeNG Modules**:
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ButtonModule } from 'primeng/button';
import { TableModule } from 'primeng/table';
import { InputTextModule } from 'primeng/inputtext';
import { DialogModule } from 'primeng/dialog';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [
    CommonModule,
    ButtonModule,
    TableModule,
    InputTextModule,
    DialogModule
  ],
  // ...
})
export class ProductListComponent { }
```

**Use PrimeNG Theming**:
```scss
// styles.scss
@import 'primeng/resources/themes/lara-light-blue/theme.css';
@import 'primeng/resources/primeng.css';
@import 'primeicons/primeicons.css';
```

**Follow PrimeNG Best Practices**:
```html
<!-- Use PrimeNG components consistently -->
<p-table [value]="products()" [loading]="loading()">
  <ng-template pTemplate="header">
    <tr>
      <th>Name</th>
      <th>Price</th>
      <th>Actions</th>
    </tr>
  </ng-template>
  <ng-template pTemplate="body" let-product>
    <tr>
      <td>{{ product.name }}</td>
      <td>{{ product.price | currency }}</td>
      <td>
        <p-button label="Edit" icon="pi pi-pencil" (onClick)="edit(product)"></p-button>
      </td>
    </tr>
  </ng-template>
</p-table>
```

### TypeScript Strict Mode

**Enable All Strict Checks**:
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitAny": true,
    "noImplicitThis": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

**Proper Type Annotations**:
```typescript
// ✅ Explicit types for public APIs
export interface Product {
  id: string;
  name: string;
  price: number;
  category?: string; // Optional
}

export class ProductService {
  getProduct(id: string): Observable<Product | null> { }
  
  createProduct(data: CreateProductDto): Observable<Product> { }
}

// ✅ Type guards
function isProduct(value: unknown): value is Product {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'price' in value
  );
}

// ✅ Use nullish coalescing
const name = product.name ?? 'Unknown';
const quantity = product.quantity ?? 0;

// ✅ Use optional chaining
const category = product.category?.name ?? 'Uncategorized';
```

### Reactive Forms (Signal-based)

**Use Reactive Forms with Signals** (Angular 21):
```typescript
import { Component, signal } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-product-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  // ...
})
export class ProductFormComponent {
  private fb = inject(FormBuilder);
  
  form = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    price: [0, [Validators.required, Validators.min(0)]],
    category: ['']
  });
  
  // Track form state with signals
  isSubmitting = signal(false);
  
  async onSubmit(): Promise<void> {
    if (this.form.invalid) return;
    
    this.isSubmitting.set(true);
    try {
      await this.productService.create(this.form.value);
      this.router.navigate(['/products']);
    } finally {
      this.isSubmitting.set(false);
    }
  }
}
```

### ESLint Configuration

```json
// .eslintrc.json
{
  "root": true,
  "overrides": [
    {
      "files": ["*.ts"],
      "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended",
        "plugin:@angular-eslint/recommended",
        "plugin:@angular-eslint/template/process-inline-templates"
      ],
      "rules": {
        "@angular-eslint/directive-selector": [
          "error",
          { "type": "attribute", "prefix": "app", "style": "camelCase" }
        ],
        "@angular-eslint/component-selector": [
          "error",
          { "type": "element", "prefix": "app", "style": "kebab-case" }
        ],
        "@typescript-eslint/explicit-function-return-type": "warn",
        "@typescript-eslint/no-explicit-any": "error",
        "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
        "no-console": ["warn", { "allow": ["warn", "error"] }]
      }
    },
    {
      "files": ["*.html"],
      "extends": [
        "plugin:@angular-eslint/template/recommended",
        "plugin:@angular-eslint/template/accessibility"
      ]
    }
  ]
}
```

---

## Consequences

### Positive Consequences

- **Modern Angular**: Leverages Angular 21 standalone components and signals
- **Type Safety**: Strict TypeScript catches errors at compile time
- **Consistency**: ESLint enforces uniform code style
- **PrimeNG Integration**: Professional UI components with consistent design
- **Signal Performance**: Better change detection with signals
- **Automated Enforcement**: ESLint and TypeScript compiler catch violations

### Negative Consequences

- **Learning Curve**: Signals are new, agents must learn the pattern
- **Strict Types**: More initial effort to satisfy TypeScript strict mode
- **PrimeNG Dependency**: Tied to PrimeNG for UI components
- **Migration Effort**: If moving from older Angular, requires refactoring

### Neutral Considerations

- **RxJS Still Needed**: For complex async operations, HTTP, routing
- **Signal Store**: May add NgRx Signal Store if state complexity grows

---

## Alternatives Considered

### Alternative 1: Material Design (Angular Material)
- **Pros**: Official Angular UI library, well-documented
- **Cons**: Less feature-rich than PrimeNG, less business-focused
- **Rejected**: PrimeNG chosen for richer component set

### Alternative 2: Class-based State Management (No Signals)
- **Pros**: Matches older Angular patterns, more familiar
- **Cons**: Less performant change detection, outdated for Angular 21
- **Rejected**: Signals are the future of Angular

### Alternative 3: Relaxed TypeScript (strict: false)
- **Pros**: Faster initial development, fewer type errors
- **Cons**: Runtime errors, less maintainable, poor IDE support
- **Rejected**: Type safety is critical for large codebases

---

## Implementation Notes

### Setup Required

1. **Angular 21 Project** with standalone components
2. **Install PrimeNG**:
```bash
npm install primeng primeicons
```

3. **Configure ESLint**:
```bash
ng add @angular-eslint/schematics
```

4. **Update `tsconfig.json`** with strict mode (see above)

5. **Create Component Templates** following these standards

### Reference Examples

See `docs/examples/frontend/` for:
- Standalone component with signals
- Service with signal-based state
- Reactive form with PrimeNG
- Component tests

---

## Related Decisions

- [ADR-002: C# Coding Standards](002-csharp-coding-standards.md) - Backend standards
- [ADR-004: Error Handling Strategy](004-error-handling-strategy.md) - Frontend error handling
- [ADR-006: Technology Stack](006-technology-stack.md) - Angular 21 and PrimeNG versions

---

**Status**: Ready for Review  
**Action Required**: Stakeholder approval to proceed with implementation
