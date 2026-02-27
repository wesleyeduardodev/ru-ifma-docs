# Plano: Aplicação Web RU-IFMA

## Contexto
Criar do zero uma aplicação web para o Restaurante Universitário do IFMA. Página pública para alunos verem o cardápio do dia e área admin para gerenciar cardápios. Backend primeiro, frontend depois.

---

## PARTE 1 — BACKEND (Spring Boot 4.0.3 + Java 21)

### 1.1 Inicializar projeto Spring Boot
- Gerar projeto com Spring Initializr (Maven, Java 21, Spring Boot 4.0.3)
- Dependências: Spring Web, Spring Data JPA, PostgreSQL Driver, Spring Security, Validation
- Group: `br.edu.ifma.ru` / Artifact: `ru-ifma-backend`
- Criar em `ru-ifma-backend/`

### 1.2 Docker Compose (apenas PostgreSQL)
- **Arquivo:** `ru-ifma-backend/docker-compose.yml`
- PostgreSQL 17, porta 5432, banco `ru_ifma`, user/pass `postgres/postgres`
- Volume persistente para dados

### 1.3 Dockerfile multi-stage
- **Arquivo:** `ru-ifma-backend/Dockerfile`
- Stage 1 (build): `maven:3.9-eclipse-temurin-21` → build do .jar
- Stage 2 (runtime): `eclipse-temurin:21-jre-alpine` → copiar .jar, expor 8080

### 1.4 application.properties
- **Arquivo:** `ru-ifma-backend/src/main/resources/application.properties`
- Variáveis de ambiente com defaults locais:
  - `DB_URL` → `jdbc:postgresql://localhost:5432/ru_ifma`
  - `DB_USER` → `postgres`
  - `DB_PASSWORD` → `postgres`
- `spring.jpa.hibernate.ddl-auto=update`
- CORS liberado para `localhost:5173`

### 1.5 Entidades JPA
- **Administrador** (`administradores`): id, nome, email (unique), senha, criadoEm
- **Cardapio** (`cardapios`): id, data, tipoRefeicao (enum: ALMOCO/JANTAR), pratoPrincipal, acompanhamento, salada, sobremesa, suco, criadoEm, atualizadoEm
  - Constraint unique: (data + tipoRefeicao)

### 1.6 Enums
- `TipoRefeicao`: ALMOCO, JANTAR

### 1.7 Records (Request/Response)
- `CardapioRequest`: data, tipoRefeicao, pratoPrincipal, acompanhamento, salada, sobremesa, suco
- `CardapioResponse`: todos os campos da entidade
- `AdminRequest`: nome, email, senha
- `AdminResponse`: id, nome, email, criadoEm (sem senha)
- `LoginRequest`: email, senha
- `LoginResponse`: mensagem, adminResponse

### 1.8 Repositories
- `CardapioRepository extends JpaRepository<Cardapio, Long>`
  - `List<Cardapio> findByData(LocalDate data)`
  - `Optional<Cardapio> findByDataAndTipoRefeicao(LocalDate data, TipoRefeicao tipo)`
- `AdministradorRepository extends JpaRepository<Administrador, Long>`
  - `Optional<Administrador> findByEmail(String email)`

### 1.9 Services
- **CardapioService**: listarPorData, buscarPorId, criar, atualizar, deletar
- **AdministradorService**: listar, buscarPorId, criar, atualizar, deletar

### 1.10 Controllers
- **CardapioController** (`/api/cardapios`):
  - `GET /api/cardapios?data=2024-01-15` → lista cardápios da data (público)
  - `GET /api/cardapios/{id}` → detalhe (público)
  - `POST /api/cardapios` → criar (admin)
  - `PUT /api/cardapios/{id}` → atualizar (admin)
  - `DELETE /api/cardapios/{id}` → deletar (admin)
- **AdminController** (`/api/admin`):
  - `GET /api/admin` → listar admins (admin)
  - `GET /api/admin/{id}` → detalhe (admin)
  - `POST /api/admin` → criar admin (admin)
  - `PUT /api/admin/{id}` → atualizar (admin)
  - `DELETE /api/admin/{id}` → deletar (admin)
- **AuthController** (`/api/auth`):
  - `POST /api/auth/login` → login (público)
  - `GET /api/auth/me` → retorna admin logado (admin)

### 1.11 Segurança (Basic Auth simples)
- Spring Security com HTTP Basic Auth
- Rotas públicas: `GET /api/cardapios/**`, `POST /api/auth/login`
- Rotas protegidas: todo o resto (`/api/admin/**`, POST/PUT/DELETE em `/api/cardapios/**`)
- UserDetailsService customizado que busca no banco de Administradores
- Passwords com BCrypt

### 1.12 CORS
- Configuração global liberando `http://localhost:5173`
- Métodos: GET, POST, PUT, DELETE, OPTIONS
- Headers: Authorization, Content-Type

### 1.13 Seed de dados
- Classe com `CommandLineRunner` que roda no startup
- Cria admin padrão se não existir: `admin@ifma.edu.br` / `admin123`
- Senha salva com BCrypt

### 1.14 Tratamento de exceções
- `@RestControllerAdvice` global com handlers para:
  - `EntityNotFoundException` → 404
  - `DataIntegrityViolationException` → 409 (duplicata data+tipo)
  - `MethodArgumentNotValidException` → 400 (validação)

---

## PARTE 2 — FRONTEND (React + Vite + TailwindCSS)

### 2.1 Inicializar projeto
- `npm create vite@latest` com React template
- Instalar: TailwindCSS, axios, react-router-dom, react-icons, date-fns
- Criar em `ru-ifma-frontend/`

### 2.2 Estrutura de pastas
```
src/
  components/     → componentes reutilizáveis
  pages/          → páginas (Home, Admin, Login)
  services/       → api.js (axios config)
  contexts/       → AuthContext
  hooks/          → hooks customizados
```

### 2.3 Configuração Axios
- **Arquivo:** `src/services/api.js`
- `const API_URL = "http://localhost:8080"`
- Instância axios com baseURL e interceptor para Basic Auth

### 2.4 Autenticação (Context)
- AuthContext com login/logout
- Armazena credenciais (email:senha em base64) no localStorage
- PrivateRoute que redireciona para /login se não autenticado

### 2.5 Páginas Públicas
- **Home** (`/`): Cardápio do dia com navegação de datas (anterior/próximo) e seletor de data
  - Exibe cards para Almoço e Jantar
  - Cores do IFMA (verde #00843D e branco)

### 2.6 Páginas Admin
- **Login** (`/login`): Formulário simples email/senha
- **Dashboard** (`/admin`): Visão geral com contadores
- **Cardápios** (`/admin/cardapios`): Listagem + CRUD (criar/editar/excluir)
- **Administradores** (`/admin/administradores`): Listagem + CRUD

### 2.7 Design
- Usar plugin `frontend-design` para visual polido
- Responsivo (mobile-first)
- Cores IFMA: verde #00843D, branco, cinza claro para backgrounds
- Header com logo/nome do IFMA

---

## Estrutura final de arquivos

### Backend
```
ru-ifma-backend/
  docker-compose.yml
  Dockerfile
  pom.xml
  src/main/java/br/edu/ifma/ru/
    RuIfmaBackendApplication.java
    config/
      SecurityConfig.java
      CorsConfig.java
    entity/
      Administrador.java
      Cardapio.java
      TipoRefeicao.java
    repository/
      AdministradorRepository.java
      CardapioRepository.java
    service/
      AdministradorService.java
      CardapioService.java
    controller/
      AdminController.java
      CardapioController.java
      AuthController.java
    dto/
      request/
        CardapioRequest.java
        AdminRequest.java
        LoginRequest.java
      response/
        CardapioResponse.java
        AdminResponse.java
        LoginResponse.java
    exception/
      GlobalExceptionHandler.java
      ResourceNotFoundException.java
    seed/
      DataSeeder.java
```

### Frontend
```
ru-ifma-frontend/
  package.json
  vite.config.js
  tailwind.config.js
  src/
    main.jsx
    App.jsx
    index.css
    services/
      api.js
    contexts/
      AuthContext.jsx
    components/
      Header.jsx
      PrivateRoute.jsx
      CardapioCard.jsx
      AdminLayout.jsx
    pages/
      Home.jsx
      Login.jsx
      Dashboard.jsx
      CardapioList.jsx
      CardapioForm.jsx
      AdminList.jsx
      AdminForm.jsx
```

---

## Ordem de execução

1. Backend: Docker Compose + Dockerfile + projeto Spring Boot
2. Backend: Entidades + Enums + Repositories
3. Backend: DTOs (Request/Response records)
4. Backend: Services
5. Backend: Controllers
6. Backend: Security + CORS + Exception Handler
7. Backend: Seed de dados
8. **Verificação**: Subir PostgreSQL com docker-compose, rodar backend, testar endpoints
9. Frontend: Inicializar projeto + TailwindCSS + estrutura
10. Frontend: Configuração Axios + AuthContext
11. Frontend: Páginas públicas (Home com cardápio)
12. Frontend: Páginas admin (Login, Dashboard, CRUDs)
13. Frontend: Design final com plugin frontend-design
14. **Verificação**: Testar fluxo completo (público + admin)
