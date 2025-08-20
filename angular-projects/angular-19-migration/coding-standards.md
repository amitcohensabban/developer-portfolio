# 🚀 **Angular 19 Coding Standards & Best Practices**

## 📋 **Executive Summary**
This document establishes the **complete coding standards and conventions** for Angular 19 applications following the standalone architecture pattern. These standards apply to **ALL development work** - new features, bug fixes, refactoring, and maintenance.

---

## 🏗️ **Architecture Principles**

### **1. Standalone-First Architecture**
- ✅ **ALWAYS use standalone components** - no NgModules
- ✅ **Import dependencies directly** in component imports array
- ✅ **Use provideRouter, provideHttpClient** for app configuration
- ✅ **Leverage bootstrapApplication** for standalone bootstrapping

### **2. Component Architecture**
- ✅ **Single Responsibility Principle** - one component, one purpose
- ✅ **Smart vs Presentational Components** - clear separation
- ✅ **Input/Output Signals** - use `input()` and `output()` decorators
- ✅ **OnPush Change Detection** - for performance-critical components

---

## 🎯 **Angular 19 Specific Standards**

### **1. Control Flow (New Syntax)**
```typescript
// ✅ NEW Angular 19 Control Flow
@Component({
  template: `
    @if (userData()) {
      <div class="user-profile">
        @for (item of userItems(); track item.id) {
          <app-item [item]="item" />
        }
      </div>
    } @else {
      <app-loading />
    }
  `
})
export class UserProfileComponent {
  userData = input<UserProfile | undefined>(undefined);
  userItems = input<UserItem[]>([]);
}
```

### **2. Signals Usage**
```typescript
// ✅ CORRECT Signal Usage
export class DataService {
  private _data = signal<Data[]>([]);
  private _loading = signal<boolean>(false);
  
  data = computed(() => this._data());
  loading = computed(() => this._loading());
  
  updateData(newData: Data[]) {
    this._data.set(newData);
  }
  
  setLoading(loading: boolean) {
    this._loading.set(loading);
  }
}
```

### **3. Deferred Loading**
```typescript
// ✅ Use @defer for performance
@Component({
  template: `
    <div class="main-content">
      @defer (on viewport) {
        <app-heavy-component />
      } @placeholder {
        <div class="loading-placeholder">Loading...</div>
      } @error {
        <app-error-fallback />
      }
    </div>
  `
})
```

---

## 🔧 **Dependency Injection Standards**

### **1. Service Registration**
```typescript
// ✅ Standalone Service Registration
@Injectable({
  providedIn: 'root'
})
export class UserService {
  // Service implementation
}

// ✅ Component-Level Injection
@Component({
  providers: [UserService, { provide: API_URL, useValue: environment.apiUrl }]
})
```

### **2. Injection Tokens**
```typescript
// ✅ Use Injection Tokens for Configuration
export const API_CONFIG = new InjectionToken<ApiConfig>('api.config');

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_CONFIG, useValue: { baseUrl: 'https://api.example.com' } }
  ]
};
```

---

## 📁 **File Organization Standards**

### **1. Directory Structure**
```
src/
├── app/
│   ├── components/          # Reusable UI components
│   │   ├── ui/             # Basic UI components
│   │   └── feature/        # Feature-specific components
│   ├── pages/              # Route components
│   ├── services/           # Business logic services
│   ├── models/             # TypeScript interfaces/types
│   ├── utils/              # Utility functions
│   └── guards/             # Route guards
├── assets/                 # Static assets
└── environments/           # Environment configurations
```

### **2. File Naming Conventions**
- ✅ **Components**: `user-profile.component.ts`
- ✅ **Services**: `user.service.ts`
- ✅ **Models**: `user.model.ts`
- ✅ **Guards**: `auth.guard.ts`
- ✅ **Interfaces**: `user.interface.ts`

---

## 🎨 **Template Standards**

### **1. HTML Structure**
```html
<!-- ✅ Clean, Semantic HTML -->
<div class="user-profile-card">
  <header class="card-header">
    <h2 class="card-title">{{ userProfile().name }}</h2>
  </header>
  
  <main class="card-content">
    @if (userProfile().avatar) {
      <img [src]="userProfile().avatar" [alt]="userProfile().name" />
    }
  </main>
  
  <footer class="card-actions">
    <button (click)="editProfile()">Edit</button>
  </footer>
</div>
```

### **2. CSS Class Naming**
- ✅ **BEM Methodology**: `.user-profile__avatar--large`
- ✅ **Utility Classes**: Use Tailwind CSS classes
- ✅ **Component Scoping**: Use `:host` for component-specific styles

---

## 🧪 **Testing Standards**

### **1. Unit Testing**
```typescript
// ✅ Component Testing
describe('UserProfileComponent', () => {
  let component: UserProfileComponent;
  let mockUserService: jasmine.SpyObj<UserService>;

  beforeEach(() => {
    mockUserService = jasmine.createSpyObj('UserService', ['getUser']);
    
    component = new UserProfileComponent(mockUserService);
  });

  it('should display user name when data is loaded', () => {
    const mockUser = { name: 'John Doe', email: 'john@example.com' };
    mockUserService.getUser.and.returnValue(of(mockUser));
    
    component.ngOnInit();
    
    expect(component.userName()).toBe('John Doe');
  });
});
```

### **2. Integration Testing**
- ✅ **Test component interactions** with services
- ✅ **Mock external dependencies** (HTTP, localStorage)
- ✅ **Test error handling** and edge cases

---

## ⚡ **Performance Standards**

### **1. Change Detection**
```typescript
// ✅ Use OnPush for performance
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class PerformanceComponent {
  // Component implementation
}

// ✅ Use signals for reactive updates
export class DataComponent {
  data = signal<Data[]>([]);
  
  updateData(newData: Data[]) {
    this.data.set(newData); // Triggers change detection efficiently
  }
}
```

### **2. Lazy Loading**
```typescript
// ✅ Lazy load routes
export const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./pages/users/users.component')
      .then(m => m.UsersComponent)
  }
];
```

---

## 🔒 **Security Standards**

### **1. Input Sanitization**
```typescript
// ✅ Sanitize user input
@Component({
  template: `<div [innerHTML]="sanitizedContent"></div>`
})
export class SafeComponent {
  constructor(private sanitizer: DomSanitizer) {}
  
  sanitizedContent = computed(() => 
    this.sanitizer.bypassSecurityTrustHtml(this.userInput())
  );
}
```

### **2. XSS Prevention**
- ✅ **Never use innerHTML** without sanitization
- ✅ **Use Angular's built-in XSS protection**
- ✅ **Validate all user inputs** before processing

---

## 📚 **Documentation Standards**

### **1. Code Comments**
```typescript
/**
 * User Profile Component
 * 
 * Displays user information with edit capabilities.
 * Uses Angular 19 standalone architecture and signals.
 * 
 * @example
 * <app-user-profile [userId]="123" />
 */
@Component({
  selector: 'app-user-profile',
  standalone: true
})
export class UserProfileComponent {
  // Component implementation
}
```

### **2. README Requirements**
- ✅ **Project overview** and purpose
- ✅ **Setup instructions** with prerequisites
- ✅ **Usage examples** and API documentation
- ✅ **Testing instructions** and coverage requirements

---

## 🚫 **Forbidden Patterns**

### **1. NgModule Usage**
```typescript
// ❌ NEVER use NgModules
@NgModule({
  declarations: [Component],
  imports: [CommonModule]
})
export class FeatureModule { }

// ✅ ALWAYS use standalone components
@Component({
  standalone: true,
  imports: [CommonModule]
})
export class FeatureComponent { }
```

### **2. Deprecated Patterns**
- ❌ **Template-driven forms** (use reactive forms)
- ❌ **$scope usage** (AngularJS pattern)
- ❌ **Manual change detection** (use signals)

---

## 🔍 **Code Review Checklist**

### **Before Submitting Code:**
- [ ] **Standalone components** used exclusively
- [ ] **Signals** implemented for reactive state
- [ ] **Control flow** uses new Angular 19 syntax
- [ ] **Testing coverage** meets minimum requirements
- [ ] **Performance considerations** addressed
- [ ] **Security best practices** followed
- [ ] **Documentation** updated and complete

---

## 📊 **Quality Metrics**

### **Target Standards:**
- **Test Coverage**: Minimum 80% for new features
- **Bundle Size**: Monitor with webpack-bundle-analyzer
- **Performance**: Lighthouse score > 90
- **Accessibility**: WCAG 2.1 AA compliance
- **Security**: No critical vulnerabilities

---

## 🎯 **Implementation Timeline**

### **Phase 1: Foundation (Week 1-2)**
- [ ] Set up standalone component architecture
- [ ] Implement signal-based state management
- [ ] Establish testing framework and coverage

### **Phase 2: Migration (Week 3-4)**
- [ ] Convert existing components to standalone
- [ ] Implement new control flow syntax
- [ ] Add deferred loading for performance

### **Phase 3: Optimization (Week 5-6)**
- [ ] Performance testing and optimization
- [ ] Bundle size analysis and reduction
- [ ] Final testing and documentation

---

## 🚀 **Getting Started**

### **1. Environment Setup**
```bash
# Install Angular 19
npm install -g @angular/cli@19

# Create new project
ng new my-app --standalone --routing --style=css

# Run development server
ng serve
```

### **2. First Component**
```bash
# Generate standalone component
ng generate component user-profile --standalone

# Generate standalone service
ng generate service user --standalone
```

---

## 📞 **Support & Resources**

### **Documentation:**
- [Angular 19 Official Docs](https://angular.dev/)
- [Standalone Components Guide](https://angular.dev/guide/standalone-components)
- [Signals Documentation](https://angular.dev/guide/signals)

### **Team Support:**
- **Code Reviews**: Submit PRs for all changes
- **Architecture Questions**: Consult with senior developers
- **Performance Issues**: Use Angular DevTools for debugging

---

## 🎉 **Success Metrics**

### **Technical Achievements:**
- ✅ **100% Standalone Components** - No NgModules
- ✅ **Signal-Based State Management** - Reactive architecture
- ✅ **Modern Control Flow** - @if, @for, @switch syntax
- ✅ **Performance Optimization** - Deferred loading, tree-shaking

### **Team Impact:**
- ✅ **Faster Development** - Simplified dependency management
- ✅ **Better Testing** - Isolated component testing
- ✅ **Improved Performance** - Bundle size reduction
- ✅ **Modern Standards** - Angular 19 best practices

---

**🎯 Remember: These standards ensure consistency, quality, and maintainability across all Angular 19 projects. Follow them strictly for the best results!**
