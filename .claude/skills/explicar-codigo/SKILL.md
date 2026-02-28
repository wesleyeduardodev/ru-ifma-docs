---
name: explicar-codigo
description: Explica partes do codigo do projeto RU-IFMA de forma didatica, ideal para alunos e iniciantes. Usar quando pedirem para explicar como algo funciona no projeto.
argument-hint: [arquivo ou conceito]
---

Explique **$ARGUMENTS** do projeto RU-IFMA de forma didatica para alunos.

## Como explicar

1. **Analogia simples**: comece comparando com algo do dia a dia
2. **Diagrama visual**: use ASCII art para mostrar o fluxo ou estrutura
3. **Passo a passo**: explique o que acontece em cada etapa
4. **Exemplo pratico**: mostre como o codigo e usado na pratica
5. **Armadilha comum**: destaque um erro que iniciantes costumam cometer

## Contexto do projeto

- Backend: Spring Boot 3.4.3 + Java 21 (arquitetura em camadas: Controller -> Service -> Repository)
- Frontend: React 19 + Vite 7 + TailwindCSS 4
- Banco: PostgreSQL 17
- Auth: HTTP Basic Auth com Spring Security + BCrypt strength 12
- Credenciais no frontend via sessionStorage (interceptor Axios)
- API URL via variavel de ambiente (VITE_API_URL)
- Deploy: Frontend na Vercel, Backend no Railway

## Tom da explicacao

- Conversacional e acessivel
- Sem jargao desnecessario (ou explique o jargao quando usar)
- Use exemplos concretos do proprio projeto RU-IFMA
- SEM emojis

## Conceitos que os alunos costumam perguntar

- Como funciona a autenticacao (Basic Auth + Spring Security + sessionStorage)
- Fluxo de uma requisicao (Controller -> Service -> Repository)
- Como o React se comunica com o backend (Axios + interceptors)
- Como o CORS funciona (CorsConfig com variavel de ambiente)
- O que sao DTOs e por que usar Records
- Como o TailwindCSS v4 funciona sem tailwind.config.js
- Como funcionam os profiles do Spring (dev vs prod)
- O que e rate limiting e por que usar
- Como variaveis de ambiente funcionam (VITE_API_URL, CORS_ORIGINS, etc)
