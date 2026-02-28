# Correcoes de Seguranca - RU-IFMA

Auditoria realizada em 28/02/2026. Foram identificadas e corrigidas **37 vulnerabilidades** (2 criticas, 11 altas, 14 medias, 10 baixas).

---

## FASE 1 - Criticas

### 1.1 URL hardcoded removida do frontend
- **Arquivo:** `ru-ifma-frontend/src/services/api.js`
- **Antes:** URL do Railway hardcoded no codigo-fonte
- **Depois:** `import.meta.env.VITE_API_URL` com fallback para `http://localhost:8080`
- **Impacto:** URL de producao nao fica exposta no repositorio publico

### 1.2 localStorage substituido por sessionStorage
- **Arquivos:** `ru-ifma-frontend/src/contexts/AuthContext.jsx`, `ru-ifma-frontend/src/services/api.js`
- **Antes:** credenciais base64 persistiam em `localStorage` (sobrevivem ao fechamento do navegador)
- **Depois:** `sessionStorage` - credenciais sao apagadas ao fechar o navegador
- **Impacto:** reduz janela de exposicao de credenciais em computadores compartilhados

### Arquivos de ambiente criados
- `ru-ifma-frontend/.env.development` - variavel `VITE_API_URL` para desenvolvimento local
- `ru-ifma-frontend/.env.example` - template para documentar variaveis necessarias

---

## FASE 2 - Altas

### 2.1 Headers de seguranca no Vercel
- **Arquivo:** `ru-ifma-frontend/vercel.json`
- Headers adicionados:
  - `Content-Security-Policy` - restringe origens de scripts, estilos, conexoes
  - `X-Content-Type-Options: nosniff` - previne MIME type sniffing
  - `X-Frame-Options: DENY` - previne clickjacking
  - `Referrer-Policy: strict-origin-when-cross-origin` - controla vazamento de URLs
  - `Permissions-Policy` - desabilita camera, microfone, geolocalizacao

### 2.2 Protecao de arquivos .env no .gitignore
- **Arquivo:** `ru-ifma-frontend/.gitignore`
- Adicionado `.env`, `.env.*` e excecao para `!.env.example`
- **Impacto:** previne commit acidental de variaveis de ambiente com segredos

### 2.3 Interceptor de response 401
- **Arquivo:** `ru-ifma-frontend/src/services/api.js`
- Interceptor que ao receber HTTP 401:
  - Remove credenciais do `sessionStorage`
  - Redireciona para `/login` (exceto se ja estiver na pagina de login)
- **Impacto:** logout automatico quando sessao expira ou credenciais sao invalidadas

### 2.4 Profiles de aplicacao (dev vs prod)
- **Arquivos:**
  - `ru-ifma-backend/src/main/resources/application.properties` - configs base + `spring.profiles.active=${SPRING_PROFILES_ACTIVE:dev}`
  - `ru-ifma-backend/src/main/resources/application-dev.properties` - `show-sql=true`, `ddl-auto=update`
  - `ru-ifma-backend/src/main/resources/application-prod.properties` - `show-sql=false`, `ddl-auto=validate`, `include-stacktrace=never`, `include-message=never`
- **Impacto:** producao nao exibe SQLs no log nem stacktraces nas respostas de erro

### 2.5 GlobalExceptionHandler - handler de fallback
- **Arquivo:** `ru-ifma-backend/src/main/java/br/edu/ifma/ru/exception/GlobalExceptionHandler.java`
- Adicionado `@ExceptionHandler(Exception.class)` retornando mensagem generica "Erro interno do servidor"
- Adicionado `@ExceptionHandler(ConstraintViolationException.class)` para validacoes de path variables
- **Impacto:** excecoes nao tratadas nunca vazam stacktrace ou detalhes internos

### 2.6 AuthService criado - AuthController refatorado
- **Arquivos:**
  - `ru-ifma-backend/src/main/java/br/edu/ifma/ru/service/AuthService.java` (novo)
  - `ru-ifma-backend/src/main/java/br/edu/ifma/ru/controller/AuthController.java` (refatorado)
- **Mudancas:**
  - Controller nao acessa mais o repository diretamente (respeita a camada Controller -> Service -> Repository)
  - `RuntimeException` substituido por `ResourceNotFoundException` no endpoint `/me`
  - Timing attack corrigido: `passwordEncoder.matches()` sempre executa, mesmo quando usuario nao existe
- **Impacto:** atacante nao consegue distinguir "usuario nao existe" de "senha incorreta" pelo tempo de resposta

### 2.7 Headers de seguranca no Spring Security
- **Arquivo:** `ru-ifma-backend/src/main/java/br/edu/ifma/ru/config/SecurityConfig.java`
- Headers adicionados:
  - HSTS com `includeSubDomains` e `maxAge` de 1 ano
  - CSP: `default-src 'self'; frame-ancestors 'none'`
  - Referrer-Policy: `strict-origin-when-cross-origin`
- **Impacto:** protecao contra downgrade HTTPS, clickjacking e vazamento de referrer

### 2.8 CORS via variavel de ambiente
- **Arquivo:** `ru-ifma-backend/src/main/java/br/edu/ifma/ru/config/CorsConfig.java`
- **Antes:** origens hardcoded no codigo (`localhost:5173`, `ru-ifma-frontend.vercel.app`)
- **Depois:** variavel de ambiente `CORS_ORIGINS` (separada por virgula), fallback para `http://localhost:5173`
- **Impacto:** origens permitidas podem ser configuradas sem alterar codigo

---

## FASE 3 - Medias

### 3.1 Rate limiting
- **Arquivo:** `ru-ifma-backend/src/main/java/br/edu/ifma/ru/config/RateLimitFilter.java` (novo)
- Limites:
  - `/api/auth/login` POST: 5 requisicoes por minuto por IP
  - Demais endpoints `/api/*`: 60 requisicoes por minuto por IP
- Implementacao com `ConcurrentHashMap` e janela deslizante de 1 minuto
- Suporte a `X-Forwarded-For` para identificar IP real atras de proxy/load balancer
- **Impacto:** previne brute force de senha e abuso de endpoints

### 3.2 Validacao de tamanho nos DTOs
- **Arquivos:**
  - `AdminRequest.java` - `@Size(min=2, max=100)` nome, `@Size(max=150)` email, `@Size(min=8, max=72)` senha
  - `CardapioRequest.java` - `@Size(max=200)` em todos os campos de texto
  - `LoginRequest.java` - `@Size(max=150)` email, `@Size(max=72)` senha
- **Impacto:** previne payloads excessivamente grandes e garante senha minima de 8 caracteres no backend

### 3.3 Sourcemap desabilitado
- **Arquivo:** `ru-ifma-frontend/vite.config.js`
- Adicionado `build: { sourcemap: false }`
- **Impacto:** codigo-fonte nao fica acessivel via DevTools em producao

### 3.4 Validacao de senha minima no frontend
- **Arquivo:** `ru-ifma-frontend/src/pages/AdminForm.jsx`
- Adicionado `minLength={8}` no input e validacao no submit (tanto para criacao quanto edicao)
- Texto de ajuda atualizado para "Minimo de 8 caracteres"
- **Impacto:** feedback imediato ao usuario antes de enviar ao backend

### 3.5 Mensagens de erro genericas nos services
- **Arquivos:** `AdministradorService.java`, `CardapioService.java`
- **Antes:** `"Administrador nao encontrado com id: " + id` (expoe ID)
- **Depois:** `"Administrador nao encontrado"` / `"Cardapio nao encontrado"` (sem ID)
- **Impacto:** nao vaza informacao sobre quais IDs existem no banco

### 3.6 @Positive nos PathVariables
- **Arquivos:** `AdminController.java`, `CardapioController.java`
- Adicionado `@Validated` na classe e `@Positive` nos parametros `@PathVariable Long id`
- **Impacto:** rejeita IDs negativos ou zero antes de chegarem ao service/repository

### 3.7 Placeholder generico no login
- **Arquivo:** `ru-ifma-frontend/src/pages/Login.jsx`
- **Antes:** `placeholder="admin@ifma.edu.br"` (expoe email do admin padrao)
- **Depois:** `placeholder="seu@email.com"`
- **Impacto:** nao sugere emails validos para atacantes

---

## FASE 4 - Baixas (hardening)

### 4.1 BCrypt strength 12
- **Arquivo:** `SecurityConfig.java`
- **Antes:** `new BCryptPasswordEncoder()` (strength padrao 10)
- **Depois:** `new BCryptPasswordEncoder(12)` (~4x mais lento para brute force)
- **Nota:** senhas existentes continuam funcionando - BCrypt detecta o strength pelo hash armazenado

### 4.2 Dockerfile com usuario nao-root
- **Arquivo:** `ru-ifma-backend/Dockerfile`
- Adicionado `addgroup`/`adduser` e `USER appuser`
- **Impacto:** se o container for comprometido, o atacante nao tem acesso root

### 4.3 Healthcheck no docker-compose
- **Arquivo:** `ru-ifma-backend/docker-compose.yml`
- Adicionado `healthcheck` com `pg_isready` (intervalo 10s, timeout 5s, 5 retries)
- Adicionado `restart: unless-stopped`
- **Impacto:** Docker monitora saude do banco e reinicia automaticamente se necessario

### 4.4 Porta do banco restrita a localhost
- **Arquivo:** `ru-ifma-backend/docker-compose.yml`
- **Antes:** `"5432:5432"` (acessivel de qualquer interface de rede)
- **Depois:** `"127.0.0.1:5432:5432"` (so acessivel localmente)
- **Impacto:** banco nao fica exposto na rede

### 4.5 ESLint com regras de seguranca
- **Arquivo:** `ru-ifma-frontend/eslint.config.js`
- Adicionada regra `no-script-url` (previne `javascript:` URLs)
- **Impacto:** previne XSS via URLs maliciosas no codigo

---

## Configuracao necessaria no deploy

### Vercel (frontend)
- Configurar variavel de ambiente `VITE_API_URL` com a URL do backend no Railway
- Os headers de seguranca sao aplicados automaticamente pelo `vercel.json`

### Railway (backend)
- Configurar `SPRING_PROFILES_ACTIVE=prod`
- Configurar `CORS_ORIGINS=https://ru-ifma-frontend.vercel.app`
- Variaveis de banco (`DB_URL`, `DB_USER`, `DB_PASSWORD`) ja devem estar configuradas

---

## Arquivos modificados

### Frontend (8 arquivos)
1. `src/services/api.js` - variavel de ambiente + interceptor 401 + sessionStorage
2. `src/contexts/AuthContext.jsx` - sessionStorage
3. `src/pages/Login.jsx` - placeholder generico
4. `src/pages/AdminForm.jsx` - validacao senha minima 8 chars
5. `vercel.json` - headers de seguranca
6. `vite.config.js` - sourcemap false
7. `.gitignore` - proteger .env
8. `.env.example` (novo) - documentar variaveis
9. `.env.development` (novo) - ambiente local
10. `eslint.config.js` - regra no-script-url

### Backend (14 arquivos)
1. `src/.../config/SecurityConfig.java` - headers HSTS/CSP/Referrer + BCrypt 12 + mensagem generica
2. `src/.../config/CorsConfig.java` - origens via env var CORS_ORIGINS
3. `src/.../config/RateLimitFilter.java` (novo) - rate limiting por IP
4. `src/.../controller/AuthController.java` - refatorado para usar AuthService
5. `src/.../controller/AdminController.java` - @Validated + @Positive
6. `src/.../controller/CardapioController.java` - @Validated + @Positive
7. `src/.../service/AuthService.java` (novo) - logica de autenticacao + protecao timing attack
8. `src/.../service/AdministradorService.java` - mensagens de erro genericas
9. `src/.../service/CardapioService.java` - mensagens de erro genericas
10. `src/.../dto/request/AdminRequest.java` - @Size nos campos
11. `src/.../dto/request/CardapioRequest.java` - @Size nos campos
12. `src/.../dto/request/LoginRequest.java` - @Size nos campos
13. `src/.../exception/GlobalExceptionHandler.java` - handler fallback + ConstraintViolation
14. `application.properties` - profile ativo via env var
15. `application-dev.properties` (novo) - perfil desenvolvimento
16. `application-prod.properties` (novo) - perfil producao
17. `Dockerfile` - usuario nao-root
18. `docker-compose.yml` - healthcheck + porta restrita + restart policy
19. `RuIfmaBackendApplication.java` - @EnableScheduling para limpeza do rate limit

---

## FASE 5 - Correcoes da segunda auditoria (28/02/2026)

Apos a primeira rodada, uma segunda auditoria automatizada identificou mais 11 pontos. Todos foram corrigidos:

### 5.1 CSP do Vercel incompativel com Google Fonts
- **Arquivo:** `ru-ifma-frontend/vercel.json`
- Adicionado `https://fonts.googleapis.com` ao `style-src` e `https://fonts.gstatic.com` ao `font-src`
- **Impacto:** fonte Inter carrega corretamente em producao sem violar o CSP

### 5.2 Header HSTS ausente no Vercel
- **Arquivo:** `ru-ifma-frontend/vercel.json`
- Adicionado `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- **Impacto:** browser nunca aceita HTTP apos o primeiro acesso HTTPS

### 5.3 RateLimitFilter: X-Forwarded-For removido
- **Arquivo:** `ru-ifma-backend/.../config/RateLimitFilter.java`
- Removida leitura do cabecalho `X-Forwarded-For` (spoofavel pelo cliente)
- Agora usa exclusivamente `request.getRemoteAddr()`
- **Impacto:** atacante nao consegue burlar o rate limit enviando cabecalhos falsos

### 5.4 RateLimitFilter: limpeza periodica de memoria
- **Arquivo:** `ru-ifma-backend/.../config/RateLimitFilter.java`
- Adicionado metodo `@Scheduled(fixedRate = 120_000)` que remove entradas expiradas
- Habilitado `@EnableScheduling` na classe principal da aplicacao
- **Impacto:** previne crescimento ilimitado de memoria (vetor de DoS)

### 5.5 SecurityConfig: headers defaults restaurados
- **Arquivo:** `ru-ifma-backend/.../config/SecurityConfig.java`
- Adicionados explicitamente `contentTypeOptions(Customizer.withDefaults())` e `frameOptions(frame -> frame.deny())`
- CSP da API restringido para `default-src 'none'; frame-ancestors 'none'; base-uri 'self'`
- **Impacto:** X-Content-Type-Options: nosniff e X-Frame-Options: DENY presentes nas respostas

### 5.6 Credenciais do banco removidas do application.properties
- **Arquivo:** `ru-ifma-backend/.../application.properties`
- Removidos fallbacks hardcoded (`postgres`/`postgres`) das propriedades do banco
- Fallbacks movidos apenas para o perfil `application-dev.properties`
- **Impacto:** em producao, a aplicacao falha ao iniciar se as variaveis nao estiverem definidas

### 5.7 application-prod.properties: binding errors suprimidos
- **Arquivo:** `ru-ifma-backend/.../application-prod.properties`
- Adicionado `server.error.include-binding-errors=never`
- **Impacto:** erros de binding nao vazam detalhes tecnicos em producao

### 5.8 CLAUDE.md do frontend atualizado
- **Arquivo:** `ru-ifma-frontend/CLAUDE.md`
- Corrigida documentacao: `localStorage` -> `sessionStorage`

### 5.9 .env.example com placeholder explicito
- **Arquivo:** `ru-ifma-frontend/.env.example`
- Valor trocado de `http://localhost:8080` para `https://sua-api.up.railway.app`
- **Impacto:** reduz risco de configuracao incorreta em deploy
