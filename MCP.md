# Leantime - Model Context Protocol Documentation

## Project Overview

**Name:** Leantime  
**Version:** 3.5.12  
**Type:** Open Source Project Management System  
**License:** AGPL-3.0-only  
**Primary Language:** PHP 8.2+  
**Database:** MySQL 8.0+ / MariaDB 10.6+  
**Repository:** https://github.com/Leantime/leantime  

### Description

Leantime is an open source project management system designed for non-project managers. It combines strategy, planning, and execution in an easy-to-use interface, making project management accessible to everyone. Built with neurodiversity (ADHD, dyslexia, autism) in mind, it offers a simple interface like Trello but with the power of Jira.

### Core Value Proposition

- **User-Friendly:** Designed for non-project managers
- **Comprehensive:** Combines task management, strategic planning, and knowledge management
- **Inclusive:** Built with neurodiversity considerations
- **Extensible:** Plugin architecture and JSON-RPC API
- **Multi-tenant:** Support for multiple clients and projects with granular permissions

## Architecture Overview

### Technology Stack

#### Backend
- **Framework:** Laravel (extended with custom components)
- **Language:** PHP 8.2+
- **Database:** MySQL 8.0+ / MariaDB 10.6+ (PDO-based, migrating to Doctrine ORM)
- **Authentication:** Laravel Auth + Sanctum + Custom providers (LDAP, OIDC)
- **Session Management:** Laravel sessions (file/database/redis)
- **Queue System:** Laravel queues (database/redis)
- **Cache:** Laravel cache (file/redis)

#### Frontend
- **Build Tool:** Laravel Mix (Webpack)
- **CSS Preprocessor:** Less
- **CSS Framework:** Tailwind CSS (prefixed with `tw-*`)
- **Template Engines:** PHP templates (.tpl.php) + Blade (.blade.php)
- **JavaScript Libraries:**
  - jQuery 3.7.1 (legacy, being phased out)
  - HTMX 1.9.12 (preferred for async updates)
  - FullCalendar 6.1.11
  - Chart.js 3.6.0
  - TinyMCE 5.10.9
  - Shepherd.js 11.2.0 (user onboarding)
  - Mermaid 11.10.0 (diagrams)

#### Infrastructure
- **Web Server:** Apache/Nginx (IIS supported with modifications)
- **File Storage:** Local filesystem or S3-compatible storage
- **Email:** SMTP with MailDev for development
- **Containerization:** Docker + Docker Compose

### Architecture Pattern

Leantime follows a **Domain-Driven Architecture** with three main layers:

1. **Core Layer** (`app/Core/`)
   - Framework extensions and base classes
   - Shared functionality across all domains
   - Infrastructure services

2. **Domain Layer** (`app/Domain/`)
   - Business logic organized by feature/module
   - Each domain is self-contained with its own:
     - Controllers (HTTP endpoints)
     - HxControllers (HTMX-specific endpoints)
     - Services (business logic)
     - Repositories (data access)
     - Models (data structures)
     - Templates (views)
     - Jobs (queue workers)
     - Listeners (event handlers)

3. **Plugin Layer** (`app/Plugins/`)
   - Installable domain modules
   - Can be distributed as folders or .phar files
   - Marketplace integration with license validation

## Directory Structure

```
/
├── app/
│   ├── Core/                    # Framework core & infrastructure
│   │   ├── Application/         # Service providers
│   │   ├── Auth/               # Authentication services
│   │   ├── Configuration/      # Config loader & environment
│   │   ├── Controller/         # Base controllers
│   │   ├── Db/                 # Database abstraction (PDO)
│   │   ├── Events/             # Event system
│   │   ├── Http/               # HTTP request/response handling
│   │   ├── Middleware/         # Request middleware stack
│   │   ├── Plugins/            # Plugin system infrastructure
│   │   ├── Routing/            # Custom frontcontroller
│   │   └── UI/                 # Template handling
│   │
│   ├── Domain/                 # Application domains (55+ modules)
│   │   ├── Api/               # JSON-RPC API
│   │   ├── Auth/              # User authentication
│   │   ├── Calendar/          # Calendar management
│   │   ├── Canvas/            # Strategy canvases (Lean, Business Model, etc.)
│   │   ├── Clients/           # Client/organization management
│   │   ├── Comments/          # Comment system
│   │   ├── Dashboard/         # Project & user dashboards
│   │   ├── Files/             # File management
│   │   ├── Ideas/             # Idea boards
│   │   ├── Menu/              # Navigation menu
│   │   ├── Notifications/     # Notification system
│   │   ├── Projects/          # Project management
│   │   ├── Sprints/           # Sprint management
│   │   ├── Tickets/           # Task/ticket management
│   │   ├── Timesheets/        # Time tracking
│   │   ├── Users/             # User management
│   │   ├── Wiki/              # Documentation/wiki
│   │   └── ...                # 40+ additional domains
│   │
│   ├── Plugins/               # Plugin installations
│   ├── Views/                 # Shared view components
│   │   ├── layouts/           # Page layouts (app, blank, login, etc.)
│   │   ├── components/        # Reusable Blade components
│   │   ├── composers/         # View composers
│   │   └── sections/          # Header, footer, nav sections
│   │
│   ├── Language/              # Internationalization (20+ languages)
│   ├── Command/               # CLI commands
│   └── helpers.php            # Global helper functions
│
├── bootstrap/                  # Laravel bootstrap
├── config/                     # Configuration files
│   ├── sample.env             # Environment template
│   └── configuration.sample.php
├── database/                   # Database migrations & seeds
├── public/                     # Web root
│   ├── assets/                # Static assets (CSS, JS, images)
│   ├── theme/                 # Theme files
│   └── index.php              # Entry point
├── storage/                    # Logs, cache, sessions
├── tests/                      # Test suites
│   ├── Acceptance/            # Codeception acceptance tests
│   └── Unit/                  # PHPUnit unit tests
└── userfiles/                  # User-uploaded files
```

## Core Components

### 1. Request Lifecycle

#### Entry Point
```
public/index.php → bootstrap/app.php → Core/Bootstrap → Frontcontroller
```

#### Middleware Stack (in order)
1. `InitialHeaders` - Sets basic HTTP headers
2. `TrustProxies` - Handles proxy configurations
3. `StartSession` - Initializes session
4. `Installed` - Checks installation status
5. `Updated` - Verifies application is up to date
6. `LoadPlugins` - Loads enabled plugins
7. `Localization` - Sets up language/locale
8. `AuthCheck` - Validates authentication

#### Request Routing
- **Custom Frontcontroller:** Routes based on URL structure to domain controllers
- **Pattern:** `/{domain}/{controller}/{action}/{params}`
- **Example:** `/projects/showProject/123` → `Domain/Projects/Controllers/ShowProject.php`

#### Response Types
- Standard HTTP responses (HTML)
- JSON-RPC API responses
- HTMX partial responses
- API responses (legacy, deprecated)

### 2. Data Layer

#### Current Implementation (Being Refactored)
- **Repository Pattern:** Domain-specific repositories in `{Domain}/Repositories/`
- **Database Abstraction:** Custom PDO wrapper in `Core/Db/Db.php`
- **Models:** Simple data structures with public properties
- **Attributes:** `DbColumn` attribute for column mapping
- **Table Prefix:** All tables use `zp_` prefix (e.g., `zp_projects`, `zp_users`)

#### Migration Target (Doctrine ORM)
- Repositories will use Doctrine's EntityManager
- Models will become proper entities with annotations
- Raw SQL will be replaced with DQL or QueryBuilder
- Automatic relationship management

#### Database Schema
Key tables include:
- `zp_user` - User accounts
- `zp_clients` - Client/organization data
- `zp_projects` - Project information
- `zp_tickets` - Tasks/tickets
- `zp_timesheets` - Time tracking
- `zp_canvas` - Strategy canvases
- `zp_comments` - Comments
- `zp_files` - File metadata

### 3. Service Layer

#### Purpose
- Implements business logic
- Coordinates between repositories
- Handles validation and authorization
- Triggers events for state changes

#### Pattern
```php
namespace Leantime\Domain\{Domain}\Services;

class {Service} implements DomainService
{
    public function __construct(
        private {Repository} $repository,
        // ... other dependencies
    ) {}
    
    /**
     * @api - Methods marked @api are part of stable API
     */
    public function publicMethod($params) {
        // Validation
        // Business logic
        // Event dispatching
        // Repository calls
        return $result;
    }
}
```

### 4. Event System

#### Overview
- Custom extension of Laravel events
- Located in `Core/Events/`
- Supports both events (fire-and-forget) and filters (data transformation)

#### Event Types
1. **Events:** Notifications about state changes
2. **Filters:** Allow modification of data passing through the system

#### Registration
- Events/listeners auto-discovered via `register.php` in domains/plugins
- Supports wildcard patterns and dynamic event names

#### Current Limitation (Refactoring Target)
- Uses string-based event names (brittle when refactoring)
- Moving toward class-based events

#### Example
```php
// Dispatching an event
dispatch_event('project.created', ['project' => $project]);

// Listening to an event
EventHandler::listen('project.created', function($params) {
    // Handle event
});

// Applying a filter
$result = dispatch_filter('project.data', $data);
```

### 5. Authentication & Authorization

#### Authentication Methods
1. **Standard:** Laravel Auth + Sanctum
2. **API Keys:** System-level service accounts (username: key name, password: secret)
3. **Personal Access Tokens:** User-level tokens via Sanctum (requires AdvancedAuth plugin)
4. **LDAP:** Directory integration
5. **OIDC:** OAuth/OpenID Connect
6. **Socialite:** Additional providers via Laravel Socialite

#### User Roles (Hardcoded)
- Owner (50)
- Admin (40)
- Manager (30)
- Editor (20)
- Client Manager (10)
- Commenter (-)

#### Permission Model
- **Global Roles:** System-wide permissions
- **Client Assignment:** Each user belongs to one client
- **Project Access Levels:**
  - Everyone in organization
  - Everyone in client
  - Assigned users only
- **Special Rules:**
  - Admins and Owners can access all projects
  - Client Managers can access all client projects

### 6. API Layer

#### JSON-RPC API (Primary)
- **Endpoint:** `/api/jsonrpc`
- **Protocol:** JSON-RPC 2.0
- **Authentication:** API keys or Personal Access Tokens
- **Structure:** Thin wrapper around service layer methods
- **Methods:** Automatically exposed from service `@api` annotated methods

#### Legacy API Controllers (Deprecated)
- Located in `Domain/Api/Controllers/`
- Return JSON responses
- Being phased out in favor of HTMX and JSON-RPC

### 7. Plugin System

#### Plugin Types
1. **System Plugins**
   - Defined in environment configuration
   - Always loaded, cannot be disabled via UI
   - Load early in the application stack
   
2. **Marketplace Plugins**
   - Distributed as .phar packages
   - Require license keys
   - License validation against marketplace server
   - User-count based licensing
   
3. **Development Plugins**
   - Located in folders in `app/Plugins/`
   - Can be enabled/disabled via admin UI

#### Plugin Structure
```
app/Plugins/{PluginName}/
├── composer.json              # Plugin metadata
├── register.php              # Bootstrap file
├── Controllers/              # HTTP endpoints
├── Services/                 # Business logic
├── Repositories/             # Data access
├── Models/                   # Data structures
├── Templates/                # Views
├── Listeners/                # Event handlers
└── Jobs/                     # Queue workers
```

#### Plugin Registration
```php
// register.php
return [
    'events' => [
        'eventName' => EventListener::class,
    ],
    'filters' => [
        'filterName' => FilterListener::class,
    ],
    'middleware' => [MiddlewareClass::class],
    'menuItems' => [/* menu definitions */],
];
```

#### License Validation
- License keys stored in database
- Regular validation against marketplace.leantime.io
- Checks active user count vs. license allowance
- Plugin disabled if limit exceeded (data preserved)

### 8. Frontend Architecture

#### Template System
1. **PHP Templates (.tpl.php)**
   - Traditional template files
   - Used in most existing views
   - Access to `$tpl` helper object
   
2. **Blade Templates (.blade.php)**
   - Laravel's Blade engine
   - Preferred for new features
   - Located in `Templates/` or `Views/`

#### Component Structure
- **Layouts:** `app/Views/layouts/` - Page skeletons (app, blank, login, etc.)
- **Components:** `app/Views/components/` - Reusable Blade components
- **Sections:** `app/Views/sections/` - Headers, footers, navigation
- **Partials:** Domain-specific partial views for HTMX

#### HTMX Integration (Preferred)
- **Purpose:** Asynchronous content updates without full page reload
- **Controllers:** HxControllers in each domain
- **Pattern:**
  - Main page loads minimal data
  - Content loaded via HTMX endpoints
  - Partials represent small portions of page content
- **Benefits:** Better performance, progressive enhancement

#### JavaScript Strategy
- **HTMX:** For information updates and reloads
- **JavaScript:** For UI interactivity (editors, drag-drop, etc.)
- **Fetch API:** Only when HTMX not feasible
  - Use JSON-RPC endpoint
  - Set `credentials: "include"`
  - Set `X-Requested-With: XMLHttpRequest` header

#### CSS Architecture
- **Main Files:** `public/assets/` included via `main.less`
- **Build Process:** `npx mix` (Laravel Mix/Webpack)
- **Theming:** CSS variables in `public/themes/`
  - Dark/light theme support
  - Variables for colors, shadows, borders, spacing, fonts
- **Tailwind:** All utility classes prefixed with `tw-*`
- **Design Tokens:** Use CSS variables for consistency

### 9. Configuration System

#### Configuration Loading
1. Environment variables or `.env` file
2. Custom config loader in `Core/Configuration/`
3. Maps legacy config parameters
4. Merges with Laravel config
5. Plugin configs loaded dynamically

#### Configuration Files
- **Primary:** `config/.env` (from `sample.env` template)
- **Laravel Config:** `app/Core/Configuration/laravelConfig.php`
- **Service Providers:** Also stored in laravelConfig
- **Naming Convention:** User-editable variables prefixed with `LEAN_*`

#### Service Configuration
- Redis auto-detected and configured for:
  - Cache
  - Queue
  - Sessions
- Configuration via respective ServiceProviders

### 10. CLI Commands

#### Framework
Extended Laravel Artisan with custom commands in `app/Command/`

#### Common Commands
```bash
php bin/leantime system:update          # Update installation
php bin/leantime plugin:enable [name]   # Enable plugin
php bin/leantime plugin:disable [name]  # Disable plugin
php bin/leantime plugin:install [name]  # Install from marketplace
php bin/leantime plugin:list            # List all plugins
php bin/leantime user:add               # Add new user
php bin/leantime setting:save [k] [v]   # Save system setting
```

## Domain Modules

### Core Domains (Required)

#### Projects (`Domain/Projects`)
- Project CRUD operations
- Project settings and permissions
- Project assignment
- Integration with all other domains

#### Users (`Domain/Users`)
- User account management
- Profile management
- User-client relationships
- User-project assignments

#### Auth (`Domain/Auth`)
- Login/logout
- Session management
- Password reset
- Two-factor authentication

#### Clients (`Domain/Clients`)
- Client/organization management
- Client-level data segregation
- Client settings

#### Menu (`Domain/Menu`)
- Dynamic menu generation
- Permission-based menu items
- Plugin menu integration

#### Setting (`Domain/Setting`)
- System configuration
- User preferences
- Client settings
- Project settings

### Task Management Domains

#### Tickets (`Domain/Tickets`)
- Task/ticket CRUD
- Status management
- Priority and effort tracking
- Dependencies and subtasks
- Multiple views: Kanban, Table, List, Gantt, Calendar

#### Sprints (`Domain/Sprints`)
- Sprint planning and management
- Sprint backlog
- Velocity tracking
- Depends on Tickets domain

#### Timesheets (`Domain/Timesheets`)
- Time entry and tracking
- Timesheet reports
- Project/task time allocation

#### Calendar (`Domain/Calendar`)
- Calendar views
- Event management
- Integration with tasks/tickets

### Strategic Planning Domains

#### Canvas Family
Multiple strategy canvas types:
- **Leancanvas** - Lean Canvas
- **Dbmcanvas** - Business Model Canvas
- **Swotcanvas** - SWOT Analysis
- **Riskscanvas** - Risk Analysis
- **Goalcanvas** - Goal Setting
- **Emcanvas** - Empathy Map
- **Valuecanvas** - Value Proposition
- And 10+ more canvas types

Each provides structured strategic planning tools.

### Knowledge Management Domains

#### Wiki (`Domain/Wiki`)
- Documentation creation
- Wiki pages with rich text
- Templates for common documents
- Category organization

#### Ideas (`Domain/Ideas`)
- Idea collection and voting
- Idea boards
- Prioritization

#### Comments (`Domain/Comments`)
- Universal comment system
- Works on tickets, projects, canvases, etc.
- Mentions and notifications

#### Files (`Domain/Files`)
- File upload and storage
- S3 or local filesystem
- File attachment to entities

### Administration Domains

#### Dashboard (`Domain/Dashboard`)
- Project dashboards
- User "My Work" dashboard
- Metrics and reports
- Widget system

#### Reports (`Domain/Reports`)
- Project reports
- Time reports
- Status reports

#### Notifications (`Domain/Notifications`)
- In-app notifications
- Email notifications
- Notification preferences

#### Audit (`Domain/Audit`)
- Activity logging
- Audit trails

### Integration Domains

#### Api (`Domain/Api`)
- JSON-RPC API implementation
- API key management
- Legacy API controllers

#### Connector (`Domain/Connector`)
- External integrations
- Webhooks
- Slack/Discord/Mattermost

#### Ldap (`Domain/Ldap`)
- LDAP authentication
- Directory synchronization

#### Oidc (`Domain/Oidc`)
- OpenID Connect integration
- OAuth authentication

#### Queue (`Domain/Queue`)
- Background job processing
- Queue management

### System Domains

#### Install (`Domain/Install`)
- Installation wizard
- Database setup
- Initial user creation

#### Cron (`Domain/Cron`)
- Scheduled tasks
- Recurring jobs

#### Errors (`Domain/Errors`)
- Error handling
- Error pages

#### Help (`Domain/Help`)
- User onboarding
- Help documentation
- Guided tours (Shepherd.js)

## Development Practices

### Code Style
- **Standard:** Laravel Pint
- **Commands:**
  - Check: `make test-code-style`
  - Fix: `make fix-code-style`

### Testing Strategy

#### Test Types
1. **Static Analysis:** PHPStan (`make phpstan`)
2. **Code Style:** PHP_CodeSniffer/Pint
3. **Unit Tests:** PHPUnit (`make unit-test`)
4. **Acceptance Tests:** Codeception (`make acceptance-test`)

#### Test Organization
```
tests/
├── Acceptance/     # E2E tests with groups (api, timesheet, etc.)
└── Unit/          # Unit tests by domain
```

#### Running Specific Tests
```bash
# API tests
docker compose --file .dev/docker-compose.yaml \
  --file .dev/docker-compose.tests.yaml \
  exec leantime-dev php vendor/bin/codecept run -g api --steps

# Timesheet tests
docker compose --file .dev/docker-compose.yaml \
  --file .dev/docker-compose.tests.yaml \
  exec leantime-dev php vendor/bin/codecept run -g timesheet --steps
```

### Coding Conventions

#### Strict Types
- Use strict types for parameters and return values
- Evaluate whether arrays should be models/objects

#### PHPDoc
- All methods and classes must have valid PHPDoc
- Methods in services available via JSON-RPC include `@api` annotation

#### Error Logging
- **ALWAYS** use `Log` facade (not `error_log()`)
- Example: `Log::error($exception)`

#### DateTime Handling
- Use `CarbonImmutable` or `dtHelper()` function
- Database dates: UTC in `YYYY-MM-DD HH:MM:SS` format
- User input dates: User's timezone and format
- Parse with `DateTimeHelper` class

#### Layer Enforcement
- Controllers only call services (NOT repositories)
- Services call repositories
- Services validate input and throw exceptions
- Avoid circular service dependencies

#### Comments
- Valid PHPDoc required
- Inline comments only when necessary for complex logic
- No backwards compatibility code unless specified

### Build Commands

```bash
# Development
make clean build                # Clean and build dev environment
make run-dev                    # Start dev server (port 8090)
make install-deps-dev           # Install dev dependencies
make build-dev                  # Build for development (with source maps)

# Production
make install-deps               # Install production dependencies
make build                      # Build for production
make package                    # Package for release

# Assets
npx mix                         # Build JS/CSS (root or plugin directory)

# Maintenance
make clear-cache                # Clear application cache
```

### Development Environment

#### Docker (Recommended)
```bash
make clean build
make run-dev
```

**Services:**
- Leantime: http://localhost:8090
- MailDev: http://localhost:8081
- phpMyAdmin: http://localhost:8082 (leantime/leantime)
- S3Ninja: http://localhost:8083

**Features:**
- Xdebug enabled (port 9003)
- MySQL server
- Mail testing
- S3 testing

#### Manual Setup
1. Install PHP 8.2+ with required extensions
2. Install MySQL 8.0+
3. Clone repository
4. Copy `config/sample.env` to `config/.env`
5. Configure database credentials
6. Run `make install-deps-dev && make build-dev`
7. Point web server to `public/` directory
8. Navigate to `/install`

## Security Considerations

### Data Protection
- Never log sensitive data (passwords, API keys, personal info)
- Validate and sanitize all user inputs
- Use parameterized queries (via Repository pattern)
- Follow existing auth/authz patterns

### Input Validation
- All service methods validate inputs
- Throw exceptions for invalid data
- Sanitize before database storage

### File Security
- Validate file uploads
- Check file types and sizes
- Store outside web root when possible
- Use S3 with proper permissions

### Session Security
- Proper session management via Laravel
- CSRF protection
- Secure cookie flags
- Session timeout configuration

### API Security
- API key authentication
- Rate limiting (configurable)
- Token expiration for Personal Access Tokens
- Validate all API inputs

### Plugin Security
- Plugins follow same security standards
- Validate plugin inputs/outputs
- Principle of least privilege
- No exposure of internal system information

## Performance Optimization

### Database
- Use Repository pattern (no direct queries)
- Avoid N+1 query problems
- Consider indexes for new query patterns
- Use pagination for large result sets

### Caching
- Laravel cache (file/redis)
- Cache expensive operations
- Clear cache appropriately on updates
- Use cache tags for selective clearing

### Frontend
- Minimize JavaScript bundle size
- Use HTMX for efficient partial updates
- Optimize images and assets
- Lazy loading where appropriate
- Use existing patterns for asset loading

### File Operations
- Batch file reads when possible
- Avoid reading large files unnecessarily
- Use S3 for scalable file storage
- Stream large files

## Internationalization

### Language Support
- 20+ languages supported
- Language files: `app/Language/*/`
- Translation via Crowdin
- Format: Language-specific arrays

### Usage
```php
// In templates
$tpl->__('label.key')

// In PHP
__('label.key')
```

## Deployment

### Production Installation

#### Manual
1. Download release package from GitHub
2. Create MySQL database
3. Upload to server (point to `public/`)
4. Configure `.env` with DB credentials
5. Navigate to `/install`
6. Follow installation wizard

#### Docker
```bash
docker run -d --restart unless-stopped -p 8080:8080 \
  -e LEAN_DB_HOST=mysql \
  -e LEAN_DB_USER=admin \
  -e LEAN_DB_PASSWORD=password \
  -e LEAN_DB_DATABASE=leantime \
  -e LEAN_EMAIL_RETURN=changeme@local.local \
  --name leantime leantime/leantime:latest
```

**Important:** Mount plugin folder for plugin support:
```
-v ./plugins:/var/www/html/app/Plugins
```

#### Reverse Proxy
When behind nginx/Apache for SSL:
```
-e LEAN_APP_URL=https://yourdomain.com
```

### Updates

#### Manual
1. Backup database and files
2. Replace files with new version
3. Navigate to `/update` if DB changes

#### CLI
```bash
php bin/leantime system:update
```

#### Docker
1. Ensure MySQL volume is mounted
2. Stop/delete container
3. Pull latest image
4. Rebuild with compose file

## Extensibility

### Plugin Development
- Follow domain structure pattern
- Use event system for integration
- Provide `composer.json` metadata
- Create `register.php` for bootstrapping
- Documentation: https://docs.leantime.io/development/plugin-development

### JSON-RPC API
- Expose service methods with `@api` annotation
- Automatically available via JSON-RPC
- Documentation: https://docs.leantime.io/api/usage

### Event System
- Listen to existing events
- Create new events for your features
- Use filters to modify data flow

### Theme Development
- CSS variables for customization
- Dark/light theme support
- Located in `public/theme/`

## Known Technical Debt

### Refactoring Targets

1. **Event System**
   - Move from string-based to class-based events
   - Event constants/registry for type safety
   - Better IDE tooling support

2. **Data Layer**
   - Migrate from custom PDO to Doctrine ORM
   - Convert repositories to use EntityManager
   - Replace raw SQL with DQL/QueryBuilder
   - Proper entity relationships

3. **Frontend**
   - Phase out jQuery
   - Complete HTMX migration
   - Consolidate JavaScript patterns
   - Complete Tailwind CSS migration

4. **API**
   - Deprecate legacy API controllers
   - Migrate all to JSON-RPC
   - Standardize response formats

5. **Plugin System**
   - Automatic plugin updates
   - Better dependency management
   - Versioning and compatibility checking
   - Standardized activation/deactivation hooks

## Support and Community

### Documentation
- Official Docs: https://docs.leantime.io
- API Docs: https://docs.leantime.io/api/usage
- Plugin Development: https://docs.leantime.io/development/plugin-development

### Community
- Discord: https://discord.gg/4zMzJtAq9z
- GitHub Issues: https://github.com/Leantime/leantime/issues
- Crowdin (Translations): https://crowdin.com/project/leantime

### Commercial
- Managed Hosting: https://leantime.io/managed-hosting/
- SaaS Product: https://leantime.io/pricing/
- Marketplace: https://marketplace.leantime.io
- Priority Support: https://leantime.io/priority-support/

### Contributing
- Bugs: Create GitHub issue or submit PR
- Features: Discuss on Discord first
- Translations: Submit via Crowdin or PR
- Code Style: Follow Laravel Pint standards

## Quick Reference

### File Locations
- **Controllers:** `app/Domain/{Domain}/Controllers/`
- **Services:** `app/Domain/{Domain}/Services/`
- **Repositories:** `app/Domain/{Domain}/Repositories/`
- **Models:** `app/Domain/{Domain}/Models/`
- **Templates:** `app/Domain/{Domain}/Templates/`
- **Views:** `app/Views/` (shared) or domain-specific
- **Config:** `config/.env` and `app/Core/Configuration/laravelConfig.php`
- **Migrations:** `database/migrations/`
- **Routes:** Custom frontcontroller in `app/Core/Routing/`
- **Public Assets:** `public/assets/`
- **Logs:** `storage/logs/`

### Common Patterns

#### Creating a New Domain Feature
1. Create domain directory: `app/Domain/YourFeature/`
2. Add Controller, Service, Repository, Model
3. Add Templates directory
4. Register in Modulemanager (if new module)
5. Add database migrations
6. Add language strings
7. Add menu items

#### Adding an Event Listener
```php
// In domain's register.php or EventListener class
EventHandler::listen('event.name', function($params) {
    // Handle event
});
```

#### Creating an API Endpoint
```php
// In Service class
/**
 * @api
 */
public function publicMethod($params): array
{
    // Method automatically exposed via JSON-RPC
}
```

#### HTMX Endpoint
```php
// HxController method
public function partialContent($params): Response
{
    return view('domain::partials.content', $data);
}
```

### Environment Variables (Selected)

```env
LEAN_DB_HOST=localhost
LEAN_DB_USER=root
LEAN_DB_PASSWORD=
LEAN_DB_DATABASE=leantime
LEAN_DB_PORT=3306

LEAN_APP_URL=https://yourdomain.com
LEAN_SESSION_PASSWORD=changeme
LEAN_SESSION_SALT=changeme

# Email
LEAN_EMAIL_RETURN=changeme@yourdomain.com
LEAN_EMAIL_USE_SMTP=true

# File Storage
LEAN_USE_S3=false
LEAN_S3_KEY=
LEAN_S3_SECRET=
LEAN_S3_BUCKET=

# LDAP
LEAN_LDAP_USE_LDAP=false

# OIDC
LEAN_OIDC_ENABLE=false
```

## Conclusion

Leantime is a comprehensive, well-architected project management system built with PHP/Laravel and following domain-driven design principles. It offers extensive functionality out of the box while remaining extensible through a robust plugin system and API. The codebase is actively maintained with ongoing refactoring efforts to modernize the data layer (Doctrine), frontend (HTMX/Tailwind), and event system (class-based events).

Key strengths:
- Clear domain-driven architecture
- Extensive feature set
- Strong plugin ecosystem
- Active community and commercial support
- Accessibility and neurodiversity focus

The system is suitable for teams seeking an open-source project management solution that balances ease of use with powerful features, and can be customized or extended to meet specific organizational needs.
