# ADR-007: Authentication & Authorization Strategy

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

Web applications require secure authentication (verifying user identity) and authorization (controlling access to resources). We need to define:
- How users authenticate (login mechanism)
- How we secure API endpoints
- How we manage user sessions and tokens
- How we implement role-based or claims-based authorization
- How the Angular frontend integrates with backend auth

Key requirements:
- Enterprise Single Sign-On (SSO) capability
- Stateless API architecture (no server-side sessions)
- Secure token storage and transmission
- Support for role-based access control (RBAC)
- Ability to extend to claims-based authorization
- Protection against common auth vulnerabilities (CSRF, XSS, token theft)
- Integration with Angular HTTP interceptors
- Centralized user management

---

## Decision

We will implement **Azure AD (Microsoft Entra ID) Single Sign-On** authentication using **Microsoft.Identity.Web** for the backend and **MSAL (Microsoft Authentication Library) for Angular** for the frontend, with **role-based authorization** and claims-based access control.

### Architecture Overview

**Authentication Flow**:
1. User initiates login in Angular app
2. MSAL redirects to Azure AD login page
3. User authenticates with Azure AD credentials
4. Azure AD issues JWT access token and ID token
5. MSAL stores tokens and handles automatic refresh
6. Angular sends access token with API requests
7. Backend validates token with Azure AD
8. Backend authorizes based on roles/claims in token

### Backend Implementation

**Microsoft.Identity.Web Integration**:
- Use `Microsoft.Identity.Web` for Azure AD authentication
- Validate JWT tokens issued by Azure AD
- No custom token generation - Azure AD handles everything
- Access user identity via `HttpContext.User`
- Leverage Azure AD roles and claims for authorization

**Token Validation**:
- Azure AD issues JWT access tokens (default 1 hour expiration)
- Backend validates token signature using Azure AD public keys
- Token contains: user ID (oid), email, roles, groups, custom claims
- Automatic token validation via Microsoft.Identity.Web

**Sample Azure AD Token Claims**:
```json
{
  "oid": "00000000-0000-0000-0000-000000000000",
  "preferred_username": "user@contoso.com",
  "name": "John Doe",
  "roles": ["Admin", "User"],
  "groups": ["product-managers", "developers"],
  "exp": 1234567890,
  "iss": "https://login.microsoftonline.com/{tenant-id}/v2.0",
  "aud": "api://your-api-client-id"
}
```

**Authorization Approach**:
- Use `[Authorize]` attribute on controllers/endpoints
- MSAL for Angular**:
- Use `@azure/msal-angular` and `@azure/msal-browser` packages
- MSAL handles token acquisition, storage, and refresh automatically
- Supports redirect and popup authentication flows
- Built-in HTTP interceptor for adding Bearer tokens
- Automatic token refresh before expiration

**MSAL Configuration**:
```typescript
import { MsalModule, MsalInterceptor, MsalGuard } from '@azure/msal-angular';
import { PublicClientApplication, InteractionType } from '@azure/msal-browser';

export const msalConfig = {
  auth: {
    clientId: 'your-angular-app-client-id',
    authority: 'https://login.microsoftonline.com/your-tenant-id',
    redirectUri: 'http://localhost:4200',
    postLogoutRedirectUri: 'http://localhost:4200'
  },
  cache: {
    cacheLocation: 'localStorage', // or 'sessionStorage'
    storeAuthStateInCookie: false
  }
};

export const protectedResources = {
  api: {
    endpoint: 'https://api.myapp.com',
    scopes: ['api://your-api-client-id/access_as_user']
  }
};
```

**MSAL HTTP Interceptor**:
- Automatically attaches Bearer token to API requests
- Handles token acquisition and refresh
- Configured via `MsalInterceptorConfiguration`

**Route Guards**:
- UseMSAL stores tokens in browser localStorage or sessionStorage
   - Option to use sessionStorage for more security (cleared on browser close)
   - Tokens encrypted at rest by browser
   
2. **Token Transmission**:
   - HTTPS only in production (enforced)
   - Bearer token in `Authorization` header
   - Azure AD tokens signed with Microsoft's private keys
   
3. **Token Validation**:
   - Backend validates token signature using Azure AD public keys
   - Token issuer, audience, and expiration validated automatically
   - Token replay protection via nonce and timestamp claims
   
4. **CORS Configuration**:
   - Whitelist allowed origins for Angular app
   - Configure CORS in ASP.NET Core to allow Azure AD token
   
5. **Conditional Access Policies**:
   - Leverage Azure AD Conditional Access for MFA, IP restrictions, device compliance
   - Managed at Azure AD level, not in application code
   
6. **Token Refresh**:
   - MSAL automatically refreshes tokens before expiration
   - Uses hidden iframe for silent token refresh (no user interaction)
   
7. **Logout**:
   - Local logout: Clear MSAL cache
   - Single sign-out: Redirect to Azure AD logout endpoint to end all sessions
   - Refresh tokens: HttpOnly, Secure, SameSite cookies
   
2. **Token Transmission**:
   - HTTPS only in production
   - Bearer token in `Authorization` header
   
3. **CORS Configuration**:
   -Enterprise SSO**: Users can use existing organizational credentials
- **No Password Management**: Azure AD handles password policies, MFA, password resets
- **Centralized User Management**: User provisioning/deprovisioning managed in Azure AD
- **Enhanced Security**: Leverage Azure AD security features (Conditional Access, MFA, threat detection)
- **Stateless APIs**: JWT tokens enable horizontal scaling without session state
- **Automatic Token Management**: MSAL handles token refresh, storage, and expiration
- **Standards-Based**: Uses OAuth 2.0 and OpenID Connect standards
- **Audit Logging**: Azure AD provides comprehensive authentication audit logs
- **Cross-platform**: Works with web, mobile, desktop applications
- **Reduced Development Time**: No need to build authentication system
- **Compliance**: Azure AD certified for various compliance standards (SOC 2, ISO 27001, etc.)

### Negative Consequences
- **External Dependency**: Requires Azure AD service availability
  - Mitigation: Azure AD has 99.99% SLA
- **Internet Connectivity Required**: Can't authenticate without internet
  - Mitigation: Acceptable for cloud-based enterprise applications
- **Azure Costs**: Requires Azure AD licenses (though Free tier available)
  - Mitigation: Often covered by existing Microsoft 365 licenses
- **Learning Curve**: Team needs to learn Azure AD and MSAL concepts
  - Mitigation: Excellent documentation available
- **Limited Customization**: Login UI controlled by Azure AD (can be branded)
  - Mitigation: Consistent experience across Microsoft ecosystem

### Neutral Considerations
- Need Azure AD tenCustom JWT Authentication with ASP.NET Core Identity
- **Description**: Build custom authentication with JWT tokens and database-backed user store
- **Pros**: Full control, no external dependencies, works offline, no licensing costs
- **Cons**: Must build password management, MFA, security features; no SSO; more code to maintain; security responsibility on us
- **Why rejected**: Azure AD provides superior security and user management out-of-the-box; enterprise SSO is a key requirement

### Alternative 2: Auth0 or Okta
- **Description**: Use third-party identity provider service
- **Pros**: Feature-rich, easy to use, good documentation, supports various identity providers
- **Cons**: Additional vendor dependency, monthly costs per user, less integration with Microsoft ecosystem
- **Why rejected**: Organization already has Azure AD/Microsoft 365; leveraging existing investment makes sense

### Alternative 3: Azure AD B2C
- **Description**: Use Azure AD B2C for consumer-focused identity
- **Pros**: Supports social logins, customizable UI, good for external users
- **Cons**: More complex setup, additional costs, unnecessary for internal/enterprise apps
- **Why rejected**: Regular Azure AD sufficient for enterprise application; can migrate to B2C later if external users needed

### Alternative 4: Session-based Authentication (Cookies)
- **Description**: Traditional server-side sessions with session cookies
- **Pros**: Simple to implement, easy token revocation
- **Cons**: Not stateless, harder to scale horizontally, no SSO, password management required
- **Why rejected**: Violates stateless API principle, no enterprise SSO
### Neutral Considerations
- Need to implement token refresh logic on frontend
- Need to handle concurrent token refresh requests
- Need to implement logout token blacklist for refresh tokens
- CORS configuration required for cookie-based refresh tokens

---Prerequisites

**1. Azure AD Tenant**:
- Organization must have Azure AD tenant
- Can use existing Microsoft 365 tenant
- Free tier available for development/testing

**2. App Registrations**:
- Register backend API in Azure AD
- Register Angular SPA in Azure AD
- Configure redirect URIs, scopes, and permissions

### Backend Setup (ASP.NET Core)

**1. Package Dependencies**:
```xml
<PackageReference Include="Microsoft.Identity.Web" Version="3.0.*" />
```

**2. Configure in Program.cs**:
```csharp
using Microsoft.Identity.Web;

// Add Azure AD Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization(options =>
{
    // Optional: Define custom policies
    options.AddPolicy("RequireAdminRole", policy =>
        policy.RequireRole("Admin"));
    
    options.AddPolicy("RequireProductAccess", policy =>
        policy.RequireClaim("groups", "product-managers"));
});

// In middleware pipeline
app.UseAuthentication();
app.UseAuthorization();
```

**3. Configuration (appsettings.json)**:
```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-api-client-id",
    "Audience": "api://your-api-client-id"
  }
}
```

**4. Controller Example**:
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requires valid Azure AD token
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts()
    {
        // Access user info from token claims
        var userId = User.FindFirst("oid")?.Value;
        var userName = User.FindFirst("name")?.Value;
        var userEmail = User.FindFirst("preferred_username")?.Value;
        
        return Ok(new { userId, userName, userEmail });
    }
    Package Dependencies**:
```bash
npm install @azure/msal-angular @azure/msal-browser
```

**2. App Configuration (app.config.ts)**:
```typescript
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptorsFromDi } from '@angular/common/http';
import { 
  MsalModule, 
  MsalInterceptor, 
  MsalGuard, 
  MsalRedirectComponent 
} from '@azure/msal-angular';
import { 
  PublicClientApplication, 
  InteractionType,
  BrowserCacheLocation 
} from '@azure/msal-browser';
import { HTTP_INTERCEPTORS } from '@angular/common/http';

const msalConfig = {
  auth: {
    clientId: 'your-angular-app-client-id', // From Azure AD app registration
    authority: 'https://login.microsoftonline.com/your-tenant-id',
    redirectUri: 'http://localhost:4200',
    postLogoutRedirectUri: 'http://localhost:4200'
  },
  cache: {
    cacheLocation: BrowserCacheLocation.LocalStorage, // or SessionStorage
    storeAuthStateInCookie: false // Set to true for IE11
  }
};

const msalInterceptorConfig = {
  interactionType: InteractionType.Redirect,
  protectedResourceMap: new Map([
    ['https://api.myapp.com/api/*', ['api://your-api-client-id/access_as_user']],
    ['http://localhost:5000/api/*', ['api://your-api-client-id/access_as_user']]
  ])
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptorsFromDi()),
    importProvidersFrom(
      MsalModule.forRoot(
        new PublicClientApplication(msalConfig),
        null,
        msalInterceptorConfig
      )
    ),
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    },
    Azure AD Setup Steps

**1. Register Backend API**:
- Navigate to Azure Portal → Azure AD → App registrations → New registration
- Name: "MyApp API"
- Supported account types: Single tenant
- No redirect URI needed
- After creation, note the Application (client) ID and Tenant ID
- Expose an API → Add scope: `access_as_user`
- Define app roles if using role-based authorization

**2. Register Angular SPA**:
- Azure AD → App registrations → New registration
- Name: "MyApp Web"
- Supported account types: Single tenant
- Redirect URIs: `http://localhost:4200` (SPA type)
- After creation, note the Application (client) ID
- Authentication → Platform configurations → Add redirect URIs for production
- API permissions → Add `MyApp API` → `access_as_user` scope

**3. Configure App Roles (Optional)**:
- IMicrosoft.Identity.Web Documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/microsoft-identity-web)
- [MSAL Angular Documentation](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular)
- [Azure AD App Registration Guide](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
- [Secure an ASP.NET Core Web API with Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-protected-web-api-overview)
- [Angular SPA with Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/tutorial-v2-angular-auth-code)
- [Azure AD Token Claims Reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/access-tokens)
- [MSAL Configuration Reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-js-initializing-client-applications)
- [OAuth 2.0 and OpenID Connect Protocols](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocol
- **Unit Tests**: Mock MSAL service in Angular component tests, mock HttpContext.User in backend tests
- **Integration Tests**: Test API endpoints with valid/invalid Azure AD tokens
- **Security Tests**: Test unauthorized access, expired tokens, missing scopes
- **E2E Tests**: Configure Playwright with Azure AD test user credentials
- **Manual Testing**: Test login flow, token refresh, logout with real Azure AD
```typescript
import { Component, OnInit, OnDestroy, inject } from '@angular/core';
import { MsalService, MsalBroadcastService } from '@azure/msal-angular';
import { EventMessage, EventType, InteractionStatus } from '@azure/msal-browser';
import { Subject, takeUntil, filter } from 'rxjs';

@Component({
  selector: 'app-root',
  template: `
    <router-outlet></router-outlet>
  `
})
export class AppComponent implements OnInit, OnDestroy {
  private authService = inject(MsalService);
  private msalBroadcastService = inject(MsalBroadcastService);
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // Handle redirect after login
    this.authService.handleRedirectObservable().subscribe();
    
    // Listen for login success
    this.msalBroadcastService.msalSubject$
      .pipe(
        filter((msg: EventMessage) => msg.eventType === EventType.LOGIN_SUCCESS),
        takeUntil(this.destroy$)
      )
      .subscribe((result: EventMessage) => {
        console.log('Login successful', result);
      });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**4. Auth Service with Signals**:
```typescript
import { Injectable, signal, computed, inject } from '@angular/core';
import { MsalService } from '@azure/msal-angular';
import { AccountInfo } from '@azure/msal-browser';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private msalService = inject(MsalService);
  
  // Signal for current account
  private accountSignal = signal<AccountInfo | null>(null);
  
  // Computed signals
  readonly isAuthenticated = computed(() => !!this.accountSignal());
  readonly currentUser = this.accountSignal.asReadonly();
  readonly userName = computed(() => this.accountSignal()?.name ?? '');
  readonly userEmail = computed(() => this.accountSignal()?.username ?? '');
  
  constructor() {
    // Set initial account
    const accounts = this.msalService.instance.getAllAccounts();
    if (accounts.length > 0) {
      this.msalService.instance.setActiveAccount(accounts[0]);
      this.accountSignal.set(accounts[0]);
    }
  }
  
  login() {
    this.msalService.loginRedirect({
      scopes: ['openid', 'profile', 'email']
    });
  }
  
  logout() {
    this.msalService.logoutRedirect({
      postLogoutRedirectUri: 'http://localhost:4200'
    });
  }
  
  getAccessToken(): Observable<string> {
    const account = this.msalService.instance.getActiveAccount();
    if (!account) {
      throw new Error('No active account');
    }
    
    return from(
      this.msalService.instance.acquireTokenSilent({
        scopes: ['api://your-api-client-id/access_as_user'],
        account: account
      }).then(response => response.accessToken)
    );
  }
}
```

**5. Route Guard Configuration**:
```typescript
import { Routes } from '@angular/router';
import { MsalGuard } from '@azure/msal-angular';

export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [MsalGuard] // Requires authentication
  },
  {
    path: 'products',
    component: ProductsComponent,
    canActivate: [MsalGuard]
  },
  {
    path: 'login',
    component: LoginComponent
  }
];
```

**6. Login Component**:
```typescript
import { Component, inject } from '@angular/core';
import { AuthService } from './auth.service';

@Component({
  selector: 'app-login',
  template: `
    <div class="login-container">
      <h1>Welcome</h1>
      <button (click)="login()">Sign in with Microsoft</button>
    </div>
  `
})
export class LoginComponent {
  authService = inject(AuthService);
  
  login() {
    this.authService.login();
  }
}
```

**7. Header Component Example**:
```typescript
import { Component, inject } from '@angular/core';
import { AuthService } from './auth.service';

@Component({
  selector: 'app-header',
  template: `
    @if (authService.isAuthenticated()) {
      <div class="user-info">
        <span>{{ authService.userName() }}</span>
        <span>{{ authService.userEmail() }}</span>
        <button (click)="logout()">Logout</button>
      </div>
    } @else {
      <button (click)="login()">Login</button>
    }
  `
})
export class HeaderComponent {
  authService = inject(AuthService);
  
  login() {
    this.authService.login();
  }
  
  logout() {
    this.authService.logout();
  }
} ClaimsPrincipal GetPrincipalFromExpiredToken(string token);
}
```

**4. Authentication Commands (CQRS)**:
- `RegisterCommand` - Create new user
- `LoginCommand` - Authenticate and return tokens
- `RefreshTokenCommand` - Issue new access token
- `LogoutCommand` - Invalidate refresh token

### Frontend Setup (Angular)

**1. Auth Service with Signals**:
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private accessTokenSignal = signal<string | null>(null);
  private currentUserSignal = signal<User | null>(null);
  
  readonly isAuthenticated = computed(() => !!this.accessTokenSignal());
  readonly currentUser = this.currentUserSignal.asReadonly();
  
  login(credentials: LoginRequest): Observable<void> {
    return this.http.post<LoginResponse>('/api/auth/login', credentials)
      .pipe(
        tap(response => {
          this.accessTokenSignal.set(response.accessToken);
          this.currentUserSignal.set(response.user);
        }),
        map(() => void 0)
      );
  }
  
  logout(): Observable<void> {
    return this.http.post<void>('/api/auth/logout', {})
      .pipe(
        finalize(() => {
          this.accessTokenSignal.set(null);
          this.currentUserSignal.set(null);
        })
      );
  }
}
```

**2. Route Guard**:
```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isAuthenticated()) {
    return true;
  }
  
  return router.createUrlTree(['/login'], { 
    queryParams: { returnUrl: state.url } 
  });
};
```

**3. HTTP Interceptor Registration**:
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])
    )
  ]
};
```

### Testing Requirements

- **Unit Tests**: Test JWT generation, validation, token refresh logic
- **Integration Tests**: Test auth endpoints (register, login, refresh, logout)
- **Security Tests**: Test unauthorized access, expired tokens, invalid tokens
- **E2E Tests**: Test complete login flow, protected route access, logout

---

## References

- [ASP.NET Core Identity documentation](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity)
- [JWT Bearer Authentication in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/jwt-authn)
- [Angular Authentication Best Practices](https://angular.dev/best-practices/security)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [RFC 7519: JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [Angular Functional Route Guards](https://angular.dev/guide/routing/common-router-tasks#preventing-unauthorized-access)
