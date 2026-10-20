# CLAUDE.md — boas práticas para dashboards HTML de arquivo único

Instruções genéricas para qualquer projeto de dashboard construído como um único arquivo HTML
(HTML + CSS + JS + dados de exemplo, tudo inline). Objetivo: **minimizar o consumo de tokens em
alterações**, sem abrir mão do arquivo único — a escolha por um só arquivo costuma ser
deliberada nesse tipo de projeto (abrir sem servidor, compartilhar como um único artefato), e
não deve ser desfeita só para "resolver" custo de token. As regras abaixo atacam a causa real.

Aplicável a qualquer dashboard nesse formato — vendas, esportivo, financeiro, operacional etc. —
independente do domínio dos dados.

## Regra 1 — Dados Legíveis

**A regra mais importante deste arquivo.** Qualquer array/objeto JSON extenso embutido num
`<script>` (dataset principal, tabelas auxiliares derivadas, histórico, catálogos) deve sempre
ser formatado com **um registro por linha** — nunca como uma única linha minificada, mesmo que
isso deixe o arquivo mais comprido.

Por quê: ferramentas de busca (`grep`) devolvem a **linha inteira** onde um termo bate, não o
trecho ao redor dele. Um array de algumas dezenas/centenas de registros minificado numa linha só
pode facilmente passar de 10-20KB — qualquer busca por uma palavra que apareça em qualquer lugar
daquele JSON (nomes de campo comuns, que também aparecem no código) faz a ferramenta devolver o
bloco inteiro, mesmo que o objetivo fosse achar uma função em outro lugar do arquivo. Com um
registro por linha, o mesmo grep devolve só 1-2 linhas pequenas.

Isso é puramente cosmético para o navegador (JSON válido com ou sem quebra de linha) — não muda
comportamento nem performance de execução.

```js
// ERRADO — uma linha só, dezenas de KB; qualquer grep por um campo comum traz tudo isso:
const DADOS = [{"id":1,"nome":"Item A", ...},{"id":2,"nome":"Item B", ...}, ...];

// CERTO — um registro por linha:
const DADOS = [
{"id":1,"nome":"Item A", ...},
{"id":2,"nome":"Item B", ...},
...
];
```

Se os dados forem gerados/regenerados por um script externo (Python, Node etc.) e depois colados
no HTML, o script deve produzir a saída já nesse formato — um registro por linha — e não usar
serialização compacta de linha única seguida de colagem direta.

## Regra 2 — Mapa Arquivo

Manter, logo após `<style>` (ou no topo do `<script>` principal), um comentário mapeando as
faixas de linha aproximadas de cada seção grande do arquivo:

```
/* MAPA DO ARQUIVO — atualizar ao adicionar/remover seções grandes
   1-XXX      CSS
   XXX-XXX    HTML (estrutura da página)
   XXX-XXX    dados embutidos — não ler à toa, ver Regra 1
   XXX-fim    JS: configuração, dados derivados, render, eventos
*/
```

Isso permite localizar uma seção por número de linha (leitura com offset/limite) em vez de busca
exploratória ou releitura do arquivo inteiro a cada mudança. Atualizar esse índice sempre que uma
seção grande for adicionada, removida, ou deslocada o suficiente para desatualizar as faixas.

## Regra 3 — Busca Direcionada

- Preferir busca pelo nome exato de uma função/seção, com poucas linhas de contexto, a reler o
  arquivo inteiro.
- Ao ler o arquivo, restringir a faixa de linhas usando o índice da Regra 2; ler o arquivo
  inteiro só se justifica na primeira exploração de uma sessão nova (ou quando o arquivo ainda é
  pequeno o suficiente para isso ser barato).
- Evitar buscas por termos muito genéricos sem contexto adicional — combinar com filtros que
  excluam a linha de dados, se por algum motivo ela ainda não seguir a Regra 1.

## Regra 4 — Dados Primeiro

Se em algum lugar do arquivo existir um cálculo que usa os dados embutidos assim que a página
carrega — fora de uma função, por exemplo somar valores, achar o maior/menor, montar um ranking
direto no `<script>` — esse trecho precisa vir **depois** do bloco de dados no arquivo, nunca
antes. O navegador lê o arquivo de cima pra baixo: se o cálculo aparecer antes dos dados terem
sido declarados, a página quebra ao carregar.

Isso só afeta código que roda direto, fora de função. Um cálculo dentro de uma função pode ficar
em qualquer posição do arquivo, porque só executa quando a função é chamada — e nesse momento o
arquivo inteiro já foi carregado.

Por isso: antes de mover um bloco de dados de lugar (por exemplo, ao aplicar a Regra 1), checar
se existe algum cálculo solto logo abaixo dele que dependa dos dados. Se existir, manter a ordem
atual — não mover só por organização.

## Regra 5 — Verificação Enxuta

Ao validar uma mudança inspecionando o DOM/estado da página, pedir só o dado específico
necessário (um texto, uma contagem, um booleano de uma condição, um objeto pequeno) — nunca o
HTML completo de um elemento grande ou o dump de arrays inteiros, a menos que o objetivo seja
justamente inspecionar aquele markup gerado (ex.: depurar a geometria de um gráfico SVG).

## Regra 6 — Revisão Prévia

Sempre que o pedido do usuário tiver **duas alterações pequenas ou menos**, antes de começar a
executar, perguntar se há mais algum ajuste que ele queira incluir na mesma leva. Cada rodada de
execução força reancorar o estado do arquivo — perguntar antes evita pedidos avulsos em sequência
e concentra o custo numa única passada.

Se o pedido já vier com três ou mais alterações, ou o usuário confirmar que não há mais nada,
seguir direto para a execução sem perguntar de novo.

## Regra 7 — Design Vivo

Quando existir um `DESIGN.md` na raiz do projeto, ele não é só consultado — é **atualizado ao
vivo**, em conversa, conforme decisões de estilo vão sendo validadas pelo usuário.

- Sempre que uma alteração estética for feita no dashboard (cor, espaçamento, tipografia, forma
  de um componente, timing de animação, etc.) e o usuário reagir positivamente a ela (ex.: "ficou
  bom", "gostei", "perfeito", "pode manter assim", ou qualquer confirmação equivalente), **perguntar
  explicitamente se ele quer salvar aquilo como padrão fixo no `DESIGN.md`** para ser reaproveitado
  em elementos futuros — não presumir a resposta nem salvar sem perguntar.
- Se a resposta for sim, **atualizar o `DESIGN.md` na mesma leva**, registrando o novo padrão na
  seção apropriada (token de cor/tipografia/espaçamento em "Visual Language", ou regra de uso em
  "Disciplina de Uso"), na mesma linguagem genérica e reutilizável já usada no resto do documento —
  sem referenciar o componente específico que originou a decisão, só o padrão em si (mantendo o
  princípio de não colocar spoiler de feature no documento).
- A partir do momento em que um padrão for salvo, **usá-lo por padrão em todo elemento novo**
  daquele tipo, sem precisar perguntar de novo — a menos que o usuário peça explicitamente algo
  diferente para aquele caso específico.
- Se a resposta for não (o usuário só queria testar/ajustar pontualmente), não salvar — tratar
  como decisão local daquela sessão, sem propagar para o resto do dashboard.
- Não interromper o fluxo perguntando isso a cada microajuste trivial — reservar a pergunta para
  quando uma escolha estética for genuinamente candidata a virar padrão (é reutilizável, não é
  um caso isolado, e o usuário demonstrou satisfação real com o resultado).

## Checklist rápido (revisar ao fechar uma sessão de edição)

- [ ] Algum array/objeto grande foi colado numa única linha? → reformatar (Regra 1).
- [ ] O índice de seções no topo do arquivo ainda bate com as faixas reais? → atualizar (Regra 2).
- [ ] Alguma leitura de verificação devolveu mais dado do que o necessário? → reduzir escopo (Regra 5).
- [ ] Alguma mudança estética foi aprovada nesta sessão sem perguntar se vira padrão no `DESIGN.md`? → perguntar (Regra 7).
