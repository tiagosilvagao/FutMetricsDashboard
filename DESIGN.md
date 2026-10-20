# DESIGN.md — CraqueView

> Este documento é vivo: ele descreve só a identidade e a linguagem visual do projeto, não a
> lista de funcionalidades. Componentes, interações e o estado de construção são documentados
> à parte, progressivamente, à medida que cada um é efetivamente construído — não antecipar aqui
> nada que ainda não exista no dashboard.

## 1. Project Identity
- **Product:** CraqueView
- **Tagline:** Dashboard imersivo de estatísticas de jogadores do Brasileirão
- **Stage:** MVP (dados embutidos no próprio arquivo, sem backend/API ainda)
- **Primary User:** Torcedores e analistas que querem explorar estatísticas de jogadores de forma visual e imersiva, não só ler uma tabela
- **Design Feeling:** imersivo, premium, "game-HUD" (holofote, vidro fosco, dourado de troféu)

---

## 2. Visual Language

### Inspiration
- Primary: overlays de transmissão esportiva (HUD sobre o campo), painéis de estatísticas de apps de scouting e games esportivos
- Secondary: dashboards em glassmorphism (painéis translúcidos com blur sobre uma cena de fundo)
- Avoid: dashboards SaaS "flat" tradicionais, grids brancos, Bootstrap/Material padrão — nada aqui deveria parecer uma planilha

### Color System
> O projeto usa tokens em `:root` — não há paleta clara/light mode, é dark-only por natureza.

- Background base: `#08130d` (verde-preto do gramado à noite)
- Surface (painel): `--glass: rgba(14,32,22,.42)` com `backdrop-filter: blur(20px) saturate(1.25)`
- Surface secundária (inputs/chips): `--glass-2: rgba(9,24,16,.55)`
- Border: `--stroke: rgba(255,255,255,.14)` / Border sutil (divisores): `--stroke-2: rgba(255,255,255,.08)`
- Text Primary: `--ink: #ffffff`
- Text Secondary: `--ink-60: rgba(255,255,255,.62)`
- Text Terciário/Muted: `--ink-40: rgba(255,255,255,.40)`
- Accent / Brand (dourado): `--gold: #F5CE3B` · `--gold-soft: #FFDE6B` (destaques, valores de topo, botão principal)
- Success / Positivo: `--green: #38D98A` (valorização, indicadores de destaque positivo)
- Destructive / Alerta: `--red: #FF6B6B` (indicadores de risco, erro ou estado negativo)
- Cor auxiliar secundária: `#4FA3FF` → gradiente `#6ab8ff → #2f7fd6 → #1a5ba4` (uso alternativo ao dourado, quando um segundo destaque for necessário)
- Cor auxiliar intermediária: `#ffa24d` (laranja, estado de alerta intermediário entre `--gold` e `--red`)
- Cores por clube: dicionário `CLUB_COLORS` (2 tons por clube, usados em gradiente 135deg nos avatares/iniciais quando não há brasão)

### Typography
- Display Font: **Sora** (600/700) — carregada via Google Fonts, usada em rótulos de eixos de gráficos e dados técnicos
- Body Font: **Inter** — stack: `'Inter', 'Segoe UI', system-ui, -apple-system, Roboto, Helvetica, Arial, sans-serif`
- Code/Numeric Font: `ui-monospace, 'SF Mono', Menlo, Consolas, monospace` — usada em valores numéricos de destaque (`font-variant-numeric: tabular-nums`), para os números não "dançarem" ao trocar de jogador selecionado
- Scale: escala **fixa**, com 4 níveis de hierarquia — usar sempre um destes tamanhos, nunca um valor solto fora da lista:
  - Display (`--fs-display: 38px`) — maior valor numérico de destaque da tela (ex.: número de maior relevância num painel)
  - Título (`--fs-title: 24px`) — títulos de card/painel e valores de destaque secundários
  - Corpo (`--fs-body: 13px`) — texto padrão da interface, usado na maior parte da UI
  - Rótulo (`--fs-label: 10px`) — uppercase + letter-spacing, kickers e labels auxiliares

### Spacing System
- Espaçamento **fixo**: `--space: 15px` — usar esse valor como gap padrão entre elementos (itens de lista, blocos internos de um painel, margens entre seções). Não criar espaçamentos ad-hoc fora desse valor; exceções só para padding interno de componentes muito pequenos (badges, chips), cobertos pelos radius presets abaixo.
- Radius presets observados:
  - `20px` (`--radius`) — painéis e cards padrão
  - `13–14px` — botões, inputs, chips de filtro
  - `9–11px` — badges pequenos, avatares quadrados
  - `999px` — pílulas (chips/tags arredondados)
- Sombra padrão: `--shadow: 0 20px 50px -22px rgba(0,0,0,.75)` (soft drop, nunca hard box-shadow)

---

## 3. Disciplina de Uso (regras de engenharia, não de feature)
- Sempre usar os tokens definidos na Seção 2 — nunca cor solta em hex fora da paleta
- Sem estilos inline — usar classes ou as custom properties de `:root`
- Dark mode é o único modo — não desenhar variantes claras; todo o sistema visual depende do fundo escuro
- Preferir composição sobre complexidade: um componente reaproveitado em múltiplos lugares em vez de várias versões parecidas do mesmo elemento
- Componentes devem ser copy-pasteáveis e autocontidos
