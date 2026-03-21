# RotaLog - Fleet and Delivery Management System

**Legacy codebase for Alura course: "Desenvolvendo com IA: complexidade, legado e escala"**

RotaLog is a fleet and delivery management system with intentional technical debt, inconsistent patterns, and missing documentation. This is a didactic project designed to teach how to use AI to navigate, understand, and refactor complex legacy systems.

## Project Structure

```
rotalog-workspace/
├── rotalog-frontend/          # NX 17 monorepo (Angular 14 + React 17)
│   ├── apps/
│   │   ├── painel-admin/      # Angular 14 admin panel
│   │   └── rastreamento/      # React 17 tracking portal
│   └── libs/
│       ├── shared-types/      # TypeScript interfaces (mismatched with backends)
│       ├── ui-components/     # Angular components (only 2, poorly abstracted)
│       └── api-contracts/     # OpenAPI specs (outdated)
├── api-frotas/                # Java/Spring Boot 2.7 - vehicles & drivers
├── api-entregas/              # Node/Express 4.x - orders & tracking
├── api-notificacoes/          # .NET Core 6 - emails & SMS
├── docker-compose.yml         # PostgreSQL + schemas
└── tools/scripts/             # Database initialization
```

## Services Overview

### api-frotas (Java/Spring Boot 2.7)
Fleet management microservice with intentional technical debt:
- **Debt**: Spring Boot 2.7, JPA with TABLE_PER_CLASS inheritance, 400+ line services, mixed pt/en naming, zero unit tests, Flyway migrations that skip versions
- **Database**: PostgreSQL (schema: `frotas`)
- **Port**: 8080
- **Start**: `./mvnw spring-boot:run`

### api-entregas (Node/Express 4.x)
Delivery management microservice with legacy patterns:
- **Debt**: 60% callbacks, auth middleware copied from StackOverflow (2020), raw SQL queries, hardcoded env vars, outdated Swagger
- **Database**: PostgreSQL (schema: `entregas`) via Sequelize
- **Port**: 3000
- **Start**: `npm start`

### api-notificacoes (.NET Core 6)
Notification microservice with abandoned architecture:
- **Debt**: Clean Architecture abandoned mid-project, MediatrR with fat handlers, credentials in appsettings.json, 30% test coverage
- **Database**: PostgreSQL (schema: `notificacoes`)
- **Port**: 5000
- **Start**: `dotnet run`

### rotalog-frontend (NX 17 Monorepo)
Frontend applications with mixed concerns:
- **painel-admin** (Angular 14): 500+ line components, CSS global leaking, business logic in services
- **rastreamento** (React 17): 70% class components, Redux without Toolkit, API calls scattered, PropTypes instead of TypeScript
- **Ports**: 4200 (Angular), 3001 (React)

## Getting Started

### Prerequisites
- Docker & Docker Compose
- Java 11+ (for api-frotas)
- Node.js 18+ (for api-entregas and rotalog-frontend)
- .NET SDK 6.0 (for api-notificacoes)
- Maven (for api-frotas)

### Setup Database

```bash
cd rotalog-workspace
docker-compose up -d postgres
```

This creates PostgreSQL with three schemas and automatically populates them with seed data:
- `frotas` - for api-frotas (10 veículos, 8 motoristas, 6 manutenções)
- `entregas` - for api-entregas (20 entregas, 42 rastreamentos)
- `notificacoes` - for api-notificacoes (10 templates, 10 notificações)

The scripts in `tools/scripts/` are executed automatically in order:
1. `01-init.sql` - Create schemas and permissions
2. `02-migration-frotas.sql` - Create frotas tables
3. `03-migration-entregas.sql` - Create entregas tables
4. `04-migration-notificacoes.sql` - Create notificacoes tables
5. `05-seed-frotas.sql` - Populate frotas data
6. `06-seed-entregas.sql` - Populate entregas data
7. `07-seed-notificacoes.sql` - Populate notificacoes data

> **Nota**: Os scripts só são executados na primeira inicialização do container. Para recriar o banco do zero: `docker-compose down -v && docker-compose up -d postgres`

### Start Services

**Terminal 1 - api-frotas**
```bash
cd api-frotas
./mvnw spring-boot:run
```

**Terminal 2 - api-entregas**
```bash
cd api-entregas
npm install
npm start
```

**Terminal 3 - api-notificacoes**
```bash
cd api-notificacoes
dotnet restore
dotnet run
```

**Terminal 4 - rotalog-frontend**
```bash
cd rotalog-frontend

# For painel-admin (Angular)
ng serve painel-admin

# Or for rastreamento (React)
npm start rastreamento
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Users                                 │
│          (Final Clients & Fleet Managers)                    │
└──────────────────┬──────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
    ┌───▼────────┐    ┌──────▼────────┐
    │ Rastreamento│    │ Painel-Admin  │
    │  (React)   │    │   (Angular)   │
    └───┬────────┘    └──────┬────────┘
        │                     │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │   REST API Layer    │
        └──────────┬──────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
┌───▼────┐   ┌────▼────┐   ┌────▼────┐
│ Frotas │   │Entregas │   │Notif.   │
│ (Java) │   │ (Node)  │   │ (.NET)  │
└───┬────┘   └────┬────┘   └────┬────┘
    │             │             │
    └─────────────┼─────────────┘
                  │
           ┌──────▼──────┐
           │ PostgreSQL  │
           │  (3 schemas)│
           └─────────────┘
```

## Known Issues & Technical Debt

### Database
- [ ] No indexes on frequently queried columns
- [ ] No partitioning for large tables
- [ ] api-notificacoes directly accesses `frotas` schema (service boundary violation)
- [ ] No audit trail
- [ ] No event sourcing

### api-frotas (Java)
- [ ] VeiculoService is 400+ lines with mixed concerns
- [ ] JPA inheritance using TABLE_PER_CLASS (performance issue)
- [ ] No unit tests
- [ ] Flyway migrations skip versions
- [ ] Hardcoded URLs to other services

### api-entregas (Node)
- [ ] 60% callback-based code (no async/await)
- [ ] SQL injection vulnerabilities (raw queries)
- [ ] Credentials hardcoded in .env
- [ ] Auth middleware from StackOverflow 2020
- [ ] No error handling
- [ ] Swagger documentation outdated

### api-notificacoes (.NET)
- [ ] Clean Architecture abandoned
- [ ] MediatrR handlers too large
- [ ] Credentials in appsettings.json
- [ ] Only 30% test coverage
- [ ] No proper dependency injection

### rotalog-frontend
- [ ] Shared types don't match backend responses (camelCase vs snake_case)
- [ ] Only 2 UI components in ui-components lib
- [ ] Angular components with 500+ lines of template
- [ ] React class components (70%)
- [ ] Redux without Toolkit
- [ ] CSS global scope leaking
- [ ] API calls scattered throughout components

## Course Learning Path

This project is used in a 6-part course:

1. **Aula 1**: Mapeando o legado - Understanding the codebase
2. **Aula 2**: Context engineering - Teaching AI to navigate the workspace
3. **Aula 3**: Diagnóstico assistido - Using AI for code analysis
4. **Aula 4**: Refatoração segura - Safe refactoring with AI
5. **Aula 5**: Feature cross-service - Implementing features across services
6. **Aula 6**: Debug e observabilidade - Debugging distributed problems

## Important Notes

**All technical debt is intentional.** Do not fix issues during the course - they are the teaching material. The goal is to learn how to use AI to:
- Understand complex legacy code
- Navigate large workspaces
- Decompose large features
- Refactor safely with AI assistance
- Debug cross-service issues
- Implement observability

## TODO (Never Implemented)

- [ ] Add Redis for caching
- [ ] Add Kafka for event streaming
- [ ] Add Elasticsearch for logging
- [ ] Add Prometheus for metrics
- [ ] Add Grafana for visualization
- [ ] Add health checks to services
- [ ] Add proper error handling
- [ ] Add request logging
- [ ] Add authentication/authorization
- [ ] Add rate limiting
- [ ] Add API documentation
- [ ] Add integration tests
- [ ] Add performance tests
- [ ] Add security scanning
- [ ] Add CI/CD pipeline

## License

Educational project for Alura course.

---

**Note**: This is a didactic project. Do not use in production. All technical debt is intentional for learning purposes.
