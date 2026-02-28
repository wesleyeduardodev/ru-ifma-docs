---
name: gerar-endpoint
description: Gera um endpoint CRUD completo com Controller, Service, Repository e DTOs seguindo a arquitetura do RU-IFMA
disable-model-invocation: true
argument-hint: [NomeDaEntidade]
---

Gere um endpoint CRUD completo para a entidade **$ARGUMENTS** seguindo a arquitetura do projeto RU-IFMA.

As convencoes de backend (camadas, DTOs, validacao) estao no CLAUDE.md. Siga-as rigorosamente.

## Pacote base

`br.edu.ifma.ru`

## Arquivos a criar (nesta ordem)

1. `entity/$ARGUMENTS.java` - Entity JPA com getters/setters (sem Lombok)
2. `repository/$ARGUMENTSRepository.java` - Interface JpaRepository
3. `dto/request/$ARGUMENTSRequest.java` - Record com Bean Validation (@NotBlank, @NotNull, @Email, @Size)
4. `dto/response/$ARGUMENTSResponse.java` - Record com metodo static fromEntity() (sem dados sensiveis)
5. `service/$ARGUMENTSService.java` - Logica de negocio (mensagens de erro genericas, sem expor IDs)
6. `controller/$ARGUMENTSController.java` - Endpoints REST com @Validated na classe e @Positive nos @PathVariable

## Diretorio base

`ru-ifma-backend/src/main/java/br/edu/ifma/ru/`

## Padroes obrigatorios

- Controller NUNCA acessa Repository diretamente (sempre via Service)
- DTOs de API: sufixo Request/Response, sempre Java Records
- Validacao: @NotBlank, @NotNull, @Email, @Size(min/max) em todos os campos
- @Validated na classe do Controller, @Positive nos @PathVariable Long id
- ResourceNotFoundException com mensagem generica (sem expor IDs)
- Nomes em portugues (entidade, campos, metodos)
- Sem comentarios, sem emojis

## Endpoints gerados

```
GET    /api/{entidade}       - Lista todos (200)
GET    /api/{entidade}/{id}  - Busca por ID (200)
POST   /api/{entidade}       - Cria (201)
PUT    /api/{entidade}/{id}  - Atualiza (200)
DELETE /api/{entidade}/{id}  - Remove (204)
```

## Passos

1. Pergunte quais campos a entidade precisa ter
2. Crie os arquivos na ordem acima
3. Atualize o SecurityConfig se necessario (adicionar o endpoint como protegido ou publico)
4. Mostre um resumo dos endpoints criados
