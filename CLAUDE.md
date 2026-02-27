# RU-IFMA - Restaurante Universitario do IFMA

## Stack

- **Backend:** Java 21 + Spring Boot 3.4.3 + Spring Security + Spring Data JPA
- **Frontend:** React 19 + Vite 7 + TailwindCSS 4 + React Router DOM
- **Banco:** PostgreSQL 17 (via Docker Compose)
- **Autenticacao:** HTTP Basic Auth com BCrypt

## Estrutura do workspace

```
ru-ifma/
  ru-ifma-backend/    → API REST Spring Boot (porta 8080)
  ru-ifma-frontend/   → SPA React (porta 5173)
```

## Como rodar

```bash
# 1. Subir PostgreSQL
cd ru-ifma-backend && docker compose up -d

# 2. Backend (pelo IntelliJ ou terminal)
cd ru-ifma-backend && mvn spring-boot:run

# 3. Frontend
cd ru-ifma-frontend && npm run dev
```

Admin padrao: `admin@ifma.edu.br` / `admin123`

## Convencoes obrigatorias

### Linguagem
- Todo codigo em portugues: variaveis, metodos, classes, componentes, nomes de arquivos de pagina
- Nomes de entidades, DTOs e endpoints em portugues
- NAO adicionar comentarios em lugar nenhum — codigo autoexplicativo (Clean Code)
- NAO usar emojis em nenhum lugar do projeto (codigo, mensagens, UI)

### Organizacao
- Imports sempre no topo do arquivo, organizados (nunca inline)
- SOLID: responsabilidade unica, classes e funcoes pequenas e coesas
- Evitar over-engineering — so implementar o que foi pedido

### Backend
- Camadas: Controller -> Service -> Repository (controller NUNCA acessa repository direto)
- DTOs de API usam sufixo Request/Response (Java Records)
- Sufixo DTO so para uso interno entre camadas, nunca exposto na API
- Entidades JPA com getters/setters, sem Lombok
- Senhas sempre com BCrypt, nunca em texto puro

### Frontend
- Componentes funcionais com hooks (nunca class components)
- Logica de API separada em `src/services/api.js`
- Estado de autenticacao via Context API (`AuthContext`)
- Rotas protegidas via componente `PrivateRoute`
- Cores IFMA: verde `#00843D`, branco, cinza claro para backgrounds
- Todos os textos visiveis ao usuario com acentuacao correta em portugues

### Infraestrutura
- Docker so para o banco de dados (PostgreSQL)
- Backend roda pelo IntelliJ ou `mvn spring-boot:run` (NAO em container)
- Frontend roda com `npm run dev`
