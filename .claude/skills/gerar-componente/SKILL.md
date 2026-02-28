---
name: gerar-componente
description: Gera um componente React seguindo os padroes do projeto RU-IFMA
disable-model-invocation: true
argument-hint: [NomeDoComponente]
---

Gere o componente React chamado **$ARGUMENTS** seguindo os padroes do projeto RU-IFMA.

As convencoes gerais e de frontend estao no CLAUDE.md. Siga-as rigorosamente.

## Padroes de UI do projeto

- Cards: `rounded-2xl shadow-sm border border-gray-100`
- Botoes primarios: `bg-ifma text-white hover:bg-ifma-dark rounded-xl px-5 py-2.5 font-medium shadow-sm active:scale-[0.98] transition-all duration-200`
- Tabelas: `divide-y divide-gray-100`, header `bg-gray-50/80 border-b border-gray-200`
- Loading: `animate-spin` com cor verde ifma
- Inputs: `border border-gray-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-ifma/20 focus:border-ifma text-sm transition-all duration-200`
- Cores: verde ifma `bg-ifma` / `text-ifma`, fundo `bg-gray-50`, textos `text-gray-900` / `text-gray-500`
- Icones: react-icons pacote Fi (Feather Icons)

## Estrutura de pastas

- Componente reutilizavel: `ru-ifma-frontend/src/components/`
- Pagina: `ru-ifma-frontend/src/pages/`
- Se precisar de dados da API, usar `src/services/api.js`
- Se precisar de autenticacao, usar `useAuth()` de `src/contexts/AuthContext.jsx`

## Componentes reutilizaveis existentes

- `AdminLayout` - layout do painel admin com sidebar
- `CampoFormulario` - campo de formulario com label
- `CardapioCard` - card de cardapio (almoco/jantar)
- `EstadoVazio` - estado vazio com icone e mensagem
- `Footer` - rodape
- `Header` - cabecalho com navegacao
- `LogoRU` - logo SVG do RU
- `ModalConfirmacao` - modal de confirmacao com acoes
- `PaginaHeader` - header de pagina com titulo, subtitulo e botao de acao
- `PrivateRoute` - protege rotas admin

## Passos

1. Pergunte se e um componente reutilizavel ou uma pagina
2. Crie o arquivo no diretorio correto
3. Implemente seguindo os padroes acima e as convencoes do CLAUDE.md
4. Todos os textos visiveis com acentuacao correta em portugues
5. Sem comentarios, sem emojis
6. Imports organizados no topo
7. Mostre o resultado final
