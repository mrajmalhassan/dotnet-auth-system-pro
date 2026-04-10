# .NET Auth System Pro — Production Ready

> Complete authentication & authorization system built with Clean Architecture.

## Architecture

```
AuthSystem/
├── src/
│   ├── AuthSystem.Domain/           # Entities, Enums, Interfaces
│   │   ├── Entities/                # User, Role, RefreshToken, AuditLog...
│   │   ├── Enums/                   # Permission enum (the money feature)
│   │   └── Interfaces/              # IUserRepository, IRoleRepository...
│   │
│   ├── AuthSystem.Application/      # Business logic (no dependencies)
│   │   ├── DTOs/                    # Request/Response models + Result pattern
│   │   ├── Interfaces/              # IAuthService, ITokenService, IEmailService...
│   │   └── Services/                # AuthService, UserService, RoleService, AuditService
│   │
│   ├── AuthSystem.Infrastructure/   # DB, JWT, Email, BCrypt
│   │   ├── Persistence/             # EF Core DbContext + Repositories + DataSeeder
│   │   ├── Services/                # TokenService, EmailService, PasswordHasher
│   │   └── Settings/                # JwtSettings, EmailSettings
│   │
│   └── AuthSystem.API/              # ASP.NET Core Web API
│       ├── Authorization/           # Permission attribute + policy provider + handler
│       ├── Controllers/             # Auth, Users, Roles, Audit
│       └── Middleware/              # Exception handler, Audit middleware
```

## Quick Start

### 1. Configure
Edit `src/AuthSystem.API/appsettings.Development.json`:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=AuthSystemDb;..."
  },
  "JwtSettings": {
    "SecretKey": "your-super-secret-key-min-32-chars"
  }
}
```

### 2. Run Migrations
```bash
cd src/AuthSystem.API
dotnet ef migrations add InitialCreate --project ../AuthSystem.Infrastructure
dotnet ef database update
```

### 3. Run
```bash
dotnet run --project src/AuthSystem.API
```

Swagger UI opens at `https://localhost:7000`

### 4. Default Admin
```
Email:    admin@authsystem.dev
Password: Admin@123456!
```

---

## Permission System

Permissions are integer-backed enum values embedded as JWT claims:

```csharp
// Decorate any endpoint:
[RequirePermission(Permission.UsersCreate)]
public async Task<IActionResult> Create(...)

// Check in services:
if (!_currentUser.HasPermission(Permission.ReportsExport))
    return Forbid();
```

Assign permissions to roles OR directly to users:
```http
PUT /api/users/{id}/permissions
{ "permissions": [100, 101, 200, 300] }
```

## Permission Reference

| Value | Name                  |
|-------|-----------------------|
| 100   | UsersRead             |
| 101   | UsersCreate           |
| 102   | UsersUpdate           |
| 103   | UsersDelete           |
| 104   | UsersImpersonate      |
| 200   | RolesRead             |
| 201   | RolesCreate           |
| 202   | RolesUpdate           |
| 203   | RolesDelete           |
| 204   | RolesAssign           |
| 300   | PermissionsRead       |
| 301   | PermissionsAssign     |
| 400   | AuditLogsRead         |
| 500   | SettingsRead          |
| 501   | SettingsUpdate        |
| 600   | ReportsRead           |
| 601   | ReportsExport         |

## Seeded Data

| Email                  | Password       | Role    |
|------------------------|----------------|---------|
| admin@authsystem.dev   | Admin@123456!  | Admin   |

## API Endpoints

| Method | Path                               | Permission          |
|--------|------------------------------------|---------------------|
| POST   | /api/auth/register                 | Public              |
| POST   | /api/auth/login                    | Public              |
| POST   | /api/auth/refresh-token            | Public              |
| POST   | /api/auth/revoke-token             | Authenticated       |
| POST   | /api/auth/verify-email             | Public              |
| POST   | /api/auth/forgot-password          | Public              |
| POST   | /api/auth/reset-password           | Public              |
| POST   | /api/auth/change-password          | Authenticated       |
| GET    | /api/auth/me                       | Authenticated       |
| GET    | /api/users                         | UsersRead           |
| POST   | /api/users                         | UsersCreate         |
| PUT    | /api/users/{id}                    | UsersUpdate         |
| DELETE | /api/users/{id}                    | UsersDelete         |
| POST   | /api/users/{id}/lock               | UsersUpdate         |
| POST   | /api/users/{id}/unlock             | UsersUpdate         |
| POST   | /api/users/{id}/roles              | RolesAssign         |
| PUT    | /api/users/{id}/permissions        | PermissionsAssign   |
| GET    | /api/roles                         | RolesRead           |
| POST   | /api/roles                         | RolesCreate         |
| PUT    | /api/roles/{id}                    | RolesUpdate         |
| DELETE | /api/roles/{id}                    | RolesDelete         |
| GET    | /api/permissions                   | PermissionsRead     |
| GET    | /api/audit                         | AuditLogsRead       |
| GET    | /api/audit/users/{id}              | AuditLogsRead       |

## Security Features

- **JWT Access Tokens** — 15 min expiry, RS256 signed
- **Refresh Token Rotation** — old token revoked on each refresh
- **Account Lockout** — 5 failed attempts → 15 min lockout
- **BCrypt** — cost factor 12 for password hashing
- **Audit Logging** — every auth event logged with IP + user agent
- **Email Verification** — 24h token expiry
- **Password Reset** — 2h token expiry, revokes all sessions on reset


💰 Get Access
👉 Buy now: https://rajmalhassan.gumroad.com/l/pqwpip
