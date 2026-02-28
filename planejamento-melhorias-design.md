# Redesign Frontend RU-IFMA

## Contexto

O frontend atual e funcional mas visualmente basico: cards planos, navegacao de data redundante (setas + input separado), sidebar admin sem personalidade, dashboard vazio com so 3 numeros, tabelas sem polish, e nenhuma micro-interacao. Estudantes acessam o cardapio diariamente pelo celular -- a experiencia precisa ser rapida, bonita e intuitiva. O admin precisa de um painel util com acoes rapidas.

## Escopo

Redesign puramente visual/UX -- sem mudancas em API, rotas, autenticacao ou logica de negocio. Mesma stack (React 19, TailwindCSS 4, react-icons, date-fns).

---

## Fase 1 -- Fundacao

### 1.1 `src/index.css` -- Design tokens e base
- Adicionar bloco `@theme` com cores IFMA como tokens (`--color-ifma`, `ifma-dark`, `ifma-light`, `ifma-50`)
- Adicionar fonte Inter via Google Fonts no `index.html`
- Base layer: `font-sans antialiased text-gray-800`, transicao suave global
- Keyframe `loading` para animacao do spinner

### 1.2 `src/App.jsx` -- Ajuste semantico
- Envolver Routes em `<main className="flex-1">` com flex column no container

---

## Fase 2 -- Componentes utilitarios (novos)

### 2.1 `src/components/PaginaHeader.jsx` (novo, ~20 linhas)
- Props: `titulo`, `subtitulo`, `acao`
- Layout flex responsivo com titulo/subtitulo a esquerda e botao de acao a direita

### 2.2 `src/components/EstadoVazio.jsx` (novo, ~20 linhas)
- Props: `icone`, `titulo`, `descricao`
- Card branco centralizado com icone em container arredondado + textos hierarquicos

### 2.3 `src/components/CampoFormulario.jsx` (novo, ~25 linhas)
- Props: `label`, `destaque`, `children`
- Substitui as funcoes `Field` inline dos formularios
- Modo `destaque` para prato principal (background verde claro)

---

## Fase 3 -- Componentes estruturais

### 3.1 `src/components/Header.jsx`
- **Sticky** com `backdrop-blur-sm` e sombra
- Logo com ring sutil, subtitulo com melhor contraste (`text-white/70`)
- Botoes nav com `bg-white/15 hover:bg-white/25`, transicoes suaves
- Separador gradiente fino na base

### 3.2 `src/components/AdminLayout.jsx`
- Saudacao do admin no topo da sidebar (nome do usuario via AuthContext)
- Links nav com active state moderno: `bg-ifma-50 text-ifma` com icone em badge verde
- **Mobile**: sidebar horizontal scrollavel (pills) ao inves de empilhada verticalmente
- Link "Voltar ao site" com animacao direcional no hover

### 3.3 `src/components/PrivateRoute.jsx`
- Spinner branded: logo "RU" com pulse + barra de progresso animada

---

## Fase 4 -- Pagina publica (maior impacto)

### 4.1 `src/components/CardapioCard.jsx`
- Container com `hover:-translate-y-0.5` (lift sutil no hover)
- Header com gradiente: `from-ifma to-emerald-600` (almoco) / `from-emerald-800 to-emerald-700` (jantar)
- **Prato principal em destaque**: bloco separado com `bg-ifma-50`, texto `text-xl font-bold`
- Itens secundarios (acompanhamento, salada, sobremesa, suco) em **grid 2 colunas** compacto
- Divider entre prato principal e grid

### 4.2 `src/pages/Home.jsx`
- **Navegacao de data unificada**: remover input date separado; data formatada e clicavel e abre o date picker nativo (usando ref + `showPicker()`)
- Botoes prev/next com `active:scale-95` para feedback tatil
- Botao **"Voltar para hoje"** aparece quando data selecionada nao e hoje (usando `isToday` do date-fns)
- **Skeleton loading**: 2 cards placeholder com `animate-pulse` que espelham o layout real
- **Empty state melhorado** via componente EstadoVazio com mensagem contextual (data formatada)
- Titulo com `tracking-tight`, subtitulo contextual (muda se nao e hoje)

---

## Fase 5 -- Login

### 5.1 `src/pages/Login.jsx`
- **Layout split em desktop (lg+)**: painel esquerdo verde com branding RU + formulario a direita
- Mobile: so formulario (painel verde hidden)
- Alerta de erro com icone `FiAlertCircle` e borda lateral vermelha
- Botao submit com `active:scale-[0.98]`
- Link "Voltar ao cardapio" abaixo do formulario

---

## Fase 6 -- Paginas admin

### 6.1 `src/pages/Dashboard.jsx`
- Saudacao: "Bem-vindo de volta, {primeiroNome}"
- Stat cards com barra gradiente no topo e hover lift
- **Secao "Acoes rapidas"** (novo): 2 cards-link para "Novo Cardapio" e "Ver Cardapios" com icones animados no hover
- **Preview do cardapio de hoje** (novo): cards compactos mostrando prato principal de cada refeicao; alerta amarelo se nao ha cardapio cadastrado com link para criar

### 6.2 `src/pages/CardapioList.jsx`
- PaginaHeader com subtitulo "Gerencie as refeicoes do restaurante"
- Filtro de data estilizado: card com icone calendario + label + input
- Tabela: row hover verde sutil, badges de tipo com icones (FiSun/FiMoon)
- Botoes de acao: icon-only no mobile, com texto no desktop (`hidden sm:inline`)
- `overflow-x-auto` no wrapper da tabela; coluna "Acompanhamento" hidden no mobile
- Empty state via EstadoVazio

### 6.3 `src/pages/CardapioForm.jsx`
- Formulario organizado em secoes: "Informacoes gerais" (data + tipo) e "Itens do cardapio"
- Separador `<hr>` entre secoes
- Prato principal com CampoFormulario em modo `destaque`
- Inputs melhorados: `rounded-xl`, `focus:ring-ifma/20`, `focus:border-ifma`
- Botao "Cancelar" ao lado de "Salvar"
- Erro com icone e borda lateral

### 6.4 `src/pages/AdminList.jsx`
- PaginaHeader com subtitulo "Gerencie os usuarios administradores"
- **Avatar com iniciais** antes do nome de cada admin (circulo com primeira letra)
- Email em `text-gray-500`, data em formato curto (`dd MMM yyyy`)
- Mesmos padroes de tabela e acoes do CardapioList
- Empty state via EstadoVazio

### 6.5 `src/pages/AdminForm.jsx`
- Secoes: "Dados pessoais" (nome + email) e "Seguranca" (senha)
- Hint de senha minima ao criar: "Minimo de 6 caracteres"
- Mesmos padroes de campo, erro e botoes do CardapioForm

---

## Padroes de design recorrentes

| Padrao | Classes TailwindCSS |
|--------|---------------------|
| Card | `bg-white rounded-2xl shadow-sm border border-gray-100` |
| Card hover | `hover:shadow-md hover:-translate-y-0.5 transition-all duration-300` |
| Botao primario | `bg-ifma text-white px-5 py-2.5 rounded-xl font-medium hover:bg-ifma-dark shadow-sm active:scale-[0.98] transition-all duration-200` |
| Botao secundario | `text-gray-600 hover:text-gray-800 hover:bg-gray-100 px-4 py-2.5 rounded-xl font-medium transition-all duration-200` |
| Input | `w-full px-4 py-2.5 border border-gray-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-ifma/20 focus:border-ifma text-sm transition-all duration-200` |
| Label secao | `text-sm font-bold text-gray-900 uppercase tracking-wider` |
| Label item | `text-[11px] font-semibold text-gray-400 uppercase tracking-wider` |
| Badge | `inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-semibold` |
| Skeleton | `bg-gray-100 rounded animate-pulse` |

---

## Arquivos modificados (17 total)

**Novos (3):**
- `src/components/PaginaHeader.jsx`
- `src/components/EstadoVazio.jsx`
- `src/components/CampoFormulario.jsx`

**Editados (13):**
- `index.html` (link fonte Inter)
- `src/index.css`
- `src/App.jsx`
- `src/components/Header.jsx`
- `src/components/AdminLayout.jsx`
- `src/components/PrivateRoute.jsx`
- `src/components/CardapioCard.jsx`
- `src/pages/Home.jsx`
- `src/pages/Login.jsx`
- `src/pages/Dashboard.jsx`
- `src/pages/CardapioList.jsx`
- `src/pages/CardapioForm.jsx`
- `src/pages/AdminList.jsx`
- `src/pages/AdminForm.jsx`

**Inalterados:**
- `src/services/api.js`
- `src/contexts/AuthContext.jsx`

---

## Verificacao

1. `npm run dev` -- verificar que compila sem erros
2. Pagina Home: navegar datas, verificar skeleton loading, empty state, cards com hierarquia visual, botao "Voltar para hoje"
3. Login: verificar split layout em desktop, formulario limpo no mobile
4. Dashboard: saudacao, stat cards, acoes rapidas, preview do cardapio de hoje
5. CardapioList/AdminList: tabelas responsivas, filtros, acoes, empty states
6. CardapioForm/AdminForm: secoes organizadas, destaque prato principal, cancelar
7. Mobile: header sticky, sidebar horizontal, tabelas com scroll, botoes icon-only
8. Verificar que todas as funcionalidades existentes continuam funcionando (CRUD completo, login/logout, navegacao)
