# LinkONE — desenho do motor que liga os apps às IAs

Versão do desenho: 1 · 14/Nov/2026 · situação: **aguardando a revisão do dono**

Origem: conversa dedicada de 02 a 14/Nov/2026 (pesquisa de mercado, escolha do caminho B, resposta sobre IA da
plataforma de 06/Nov/2026). Nenhum código foi escrito antes deste documento.

---

## 1. O que o dono pediu (palavras dele) e o que este desenho assume

**Pedido:** "um módulo que eu consiga encaixar em qualquer um dos aplicativos que eu esteja fazendo (corporativos,
pessoais, de bolinho de chuva, jurídico), que pode ter as chaves de API cadastradas pagas que eu tenho (Claude,
Gemini), mas que, quando eu for para um terceiro ou um cliente, ele tem esse motor para cadastrar as próprias chaves
de modelos que oferecem chave grátis. Para não criar dependência de uma ou outra tecnologia e otimizar custos."
Depois: "embarcar um serviço de IA meu na plataforma, sem a pessoa ter chave própria", com custo por usuário,
cobrança por plano, limites por pessoa, IA com dados da família sem vazar, LGPD, e convivência com a chave própria.

**Decisões do dono:**
- 02/Nov: dedicar a conversa a isso.
- 14/Nov: caminho **B** (motor no navegador + porteiro opcional na nuvem); repositório próprio; nome **LinkONE**
  ("pode ser conector de outras coisas"); decisões técnicas delegadas ("a melhor escolha, performática, manutenível
  e escalável").

**Suposições deste desenho (o dono pode corrigir):**
- S1. O porteiro é uma Edge Function no Supabase da plataforma (`solverone-app`), porque ali já existem a conta
  SolverONE, os planos e a contagem de uso (função `solverone-ia`, set/2026). A Cloudflare entra como provedor, não
  como servidor.
- S2. A primeira versão cuida de **texto e foto** (visão). Voz continua no `DGO.ia` como está; **imagem** (gerar
  figura) entra na fase 3, junto com o porteiro, porque a única fonte grátis boa (Cloudflare FLUX) exige servidor.
- S3. "Conector de outras coisas" fica preparado no nome e na organização do código (`conectores/ia`), sem nenhum
  outro tipo construído agora.
- S4. Os apps que já usam `DGO.ia` não mudam: o módulo comum passa a delegar ao LinkONE quando ele estiver na página.

## 2. O que já existe (não reescrever, aproveitar)

| Onde | O quê | Situação |
|---|---|---|
| `diretrizes.js` 1.13.0 (`DGO.ia`, master no rootify-one) | 7 provedores, adaptador único para o formato OpenAI, adaptadores do Gemini e da Anthropic, troca de modelo aposentado, insistência 6/15/30 s e troca para outro provedor, gancho `cfg.ia.proxy` | funciona; padrão do Claude aponta para modelo antigo (corrige sozinho no 404) |
| `cofre.js` 1.5.0 | chaves `ia:<provedor>` cifradas com PIN/digital | funciona; o LinkONE usa os mesmos nomes |
| `solverone-app/supabase/functions/solverone-ia` | chave da plataforma em Secrets; `reservar_uso_servico` / `estornar_uso_servico`; limite por plano e por mês; freio de 40/min | pronto para histórias e voz premium do Contador, **desligado** |

## 3. Escopo

**Fase 1 (esta entrega de código, depois da aprovação):** o motor no navegador, texto e foto, com roteamento,
selos de privacidade, contagem de cota, "quem paga" e troca automática; testes automáticos; documentação.

**Fora da fase 1 (fica no plano, seção 12):** porteiro, imagem, painel no RootifyONE, créditos por plano, chaves de
empresa, IA com os dados da família, voz.

**Nunca:** revenda de chave, proxy anônimo, várias contas no mesmo provedor para multiplicar cota grátis (quebra os
termos), chave de provedor no código ou em arquivo público.

## 4. As três peças

```
 app (qualquer)                       nuvem da plataforma                 provedores
 ┌───────────────────────┐           ┌──────────────────────────┐       ┌──────────────┐
 │ DGO.ia / LinkONE.ia   │──chave da─►─────────────────────────────────►│ Groq, Gemini,│
 │  registro + roteador  │  pessoa   │                          │       │ OpenRouter…  │
 │  selos + cota local   │ (direto)  │                          │       └──────────────┘
 │  cofre.js (chaves)    │           │ porteiro (Edge Function) │       ┌──────────────┐
 │                       │──login da─►  confere plano/créditos  ├──────►│ Cloudflare,  │
 └───────────────────────┘  pessoa   │  chaves em Secrets/Vault │       │ Claude pago… │
            ▲                        └────────────┬─────────────┘       └──────────────┘
            │ comportamentos publicados           │ uso e custo (sem conteúdo)
 ┌──────────┴──────────────────────────────────────┴──────────────┐
 │ RootifyONE: provedores ligados, regra de privacidade por app,   │
 │ créditos por plano, teto do mês, painel de custo                │
 └─────────────────────────────────────────────────────────────────┘
```

Por que assim:
- **Rápido e barato:** pergunta com a chave da própria pessoa vai do aparelho direto ao provedor; nenhum servidor no
  meio, nenhum custo para a plataforma.
- **Fácil de manter:** um arquivo de motor igual em todos os apps e uma função no servidor.
- **Escala:** o Supabase cresce sozinho; empresa nova entra cadastrando chaves, sem código.
- **Sem amarra:** tudo atrás de um registro de provedores; trocar de provedor é trocar uma entrada.

### 4.1 Como o motor chega aos apps

`linkone.js` é **copiado igual** em cada app (como `cofre.js`), não carregado de outro endereço: funciona sem
internet, cada app fica com a versão que testou, e o repositório não precisa de site (GitHub Pages). Ordem na página:
`janela.js` → `cofre.js` → `linkone.js` → `diretrizes.js` → `diretrizes-config.js`. App sem o módulo comum usa
`LinkONE.configurar({ app, ... })` direto. App de fora da SolverONE (cliente) recebe o arquivo e um `cofre.js`.

### 4.2 Organização dos arquivos (fase 1)

```
linkone.js                 o motor inteiro, sem dependência (o arquivo que os apps copiam)
fontes/                    o código-fonte em partes, para ler e testar (juntado em linkone.js por um script)
  nucleo.js                configurar, eventos, versão
  conectores/ia/registro.js     provedores: capacidades, formato, selo, limites, preço, guia
  conectores/ia/adaptadores.js  formato OpenAI, Gemini, Anthropic (e Cloudflare, na fase 3)
  conectores/ia/roteador.js     escolha do provedor, tentativas, troca
  conectores/ia/cota.js         contagem local por provedor (minuto e dia), "esfriando" depois de 429
  conectores/ia/pagador.js      "quem paga": chave própria × plataforma
ferramentas/juntar.js      gera linkone.js a partir de fontes/ (node, sem dependência)
teste/rodar.js             testes com fetch falso
docs/specs/                este desenho e os próximos
ARQUITETURA.md · PENDENCIAS.md · CREDITOS.md · versoes.json · arquitetura.json
```

Juntar em um arquivo evita cinco `<script>` em cada página; o teste confere que `linkone.js` é igual ao que o
`juntar.js` produz (ninguém edita o arquivo juntado à mão).

## 5. A API do motor (o que um app chama)

```js
LinkONE.ia.perguntar({
  mensagens: [{ papel: 'pessoa', texto: '…', imagens: [blob] }],   // ou 'pergunta': '…'
  sistema: '…',                  // contexto do app (opcional)
  tarefa: 'rapida' | 'normal' | 'caprichada',   // padrão 'normal'
  capacidade: 'texto' | 'visao',                // padrão: 'visao' se houver imagem
  privacidade: 'so-verde' | 'sem-vermelho' | 'qualquer',  // padrão: a regra do app
  limite: 900,                   // tamanho máximo da resposta (tokens)
  sinal: AbortSignal,            // o ✕ da pílula do Fundo
  aoTrocar(info), aoEsperar(seg) // avisos para a tela ("usando o Groq…")
}) → Promise<{ texto, provedor, modelo, pagou: 'pessoa'|'plataforma'|'empresa', tokens: {entrada, saida}, custoUS }>

LinkONE.ia.provedores({ capacidade, prontos })   // lista para a tela de chaves
LinkONE.ia.situacao()        // por provedor: chave? selo? cota usada hoje? esfriando até?
LinkONE.ia.aoMudar(fn)       // chaves, cota ou regra mudaram
```

`DGO.ia.perguntar(p, opcoes)` continua existindo e, com o LinkONE na página, vira uma chamada a
`LinkONE.ia.perguntar` (mesmos nomes de chave no cofre, mesmo modelo escolhido por provedor, mesma preferência
`ia-provedor`). Sem o LinkONE, tudo como hoje.

Erros sempre com `codigo` e texto para leigo (PT e EN; ES quando a plataforma tiver o espanhol completo): `sem-provedor` (nenhum com chave para essa capacidade e
regra), `cota` (todos esgotados agora), `sem-internet`, `so-wifi`, `privacidade` (só há provedor que a regra do app
proíbe), `cancelado`, `recusada` (chave inválida: abre o guia daquele provedor).

## 6. O registro de provedores

Cada provedor é **uma entrada**, sem código de tela:

```js
{ id: 'groq', nome: 'Groq', formato: 'openai', base: 'https://api.groq.com/openai/v1',
  cap: ['texto'], navegador: true,             // aceita chamada direto do navegador (CORS)
  selo: 'verde',                               // verde | amarelo | vermelho | teste
  gratis: { porMinuto: 30, porDia: 1000 },     // conferido em 02/Nov/2026; a tela diz "conferir na conta"
  preco: null,                                 // US$ por milhão de tokens {entrada, saida} no modelo padrão, se pago
  modelos: { rapida: 'openai/gpt-oss-20b', normal: 'openai/gpt-oss-120b', caprichada: 'qwen/qwen3.8-27b' },
  prefixo: 'gsk_', onde: 'https://console.groq.com/keys', guia: [...passos...], fonte: 'https://console.groq.com/docs/rate-limits' }
```

Os nomes de modelo são **só o ponto de partida**: o motor lista os modelos que a chave aceita (`/models`) e troca
sozinho quando um sai do ar (como o `DGO.ia` já faz).

Registro inicial (pesquisa de 02/Nov/2026; selos e limites conferidos nas páginas oficiais quando possível):

| Provedor | Capacidades | Grátis | Selo | Direto do navegador | Fase |
|---|---|---|---|---|---|
| Groq | texto | sim (≈30/min, ≈1.000/dia) | 🟢 não guarda por padrão | sim | 1 |
| Google Gemini | texto, visão | sim (Flash/Flash-Lite) | 🔴 no grátis treina e tem revisão humana; 🟢 no pago | sim | 1 |
| OpenRouter | texto, visão | 50/dia (`:free`) | 🔴 nos Nemotron grátis; depende do modelo | sim | 1 |
| Z.ai (GLM Flash) | texto, visão | sim (limite não publicado) | 🟡 não informado | sim | 1 |
| Mistral | texto, visão | sim, com telefone | 🔴 o grátis exige aceitar treino | sim | 1 |
| OpenAI | texto, visão | não | 🟢 (API paga não treina por padrão) | sim | 1 |
| Anthropic (Claude) | texto, visão | não | 🟢 | sim (cabeçalho próprio) | 1 |
| DeepSeek | texto | não (barato) | 🟡 não informado | sim | 1 |
| xAI (Grok) | texto, visão | não confirmado | 🟡 | sim | 1 |
| Personalizado (formato OpenAI) | texto | depende | 🟡 até a pessoa marcar | depende | 1 |
| Cloudflare Workers AI | texto, imagem | sim (10 mil neurônios/dia) | 🟢 não treina nem guarda | **não** → porteiro | 3 |
| SambaNova (DeepSeek) | texto | 20/dia | 🟡 | **não** → porteiro | 3 |
| NVIDIA build | — | só teste | ⛔ termos proíbem produção | não | nunca em produção |

Fora do registro, de propósito: GitHub Models (fechou em 30/Jul/2026), Cerebras grátis (virou teste pago com
cartão), Hugging Face grátis (sem crédito), Pollinations (sem garantia, origem dos modelos não clara), qualquer
"relay" ou chave compartilhada.

Selo 🔴 e 🟢 dependem às vezes do **plano da conta** (Gemini grátis × pago). A pessoa marca na tela de chaves "esta
chave é de conta paga"; sem marcar, vale o selo do grátis.

## 7. Como o motor escolhe o provedor de cada pergunta

Para cada pedido, nesta ordem:

1. **Capacidade:** fica só quem faz o que foi pedido (texto ou visão).
2. **Privacidade:** a regra do pedido, senão a do app, senão `sem-vermelho`. `so-verde` aceita só 🟢;
   `sem-vermelho` aceita 🟢 e 🟡; `qualquer` aceita todos menos ⛔. A regra do app vem do comportamento
   `linkone.privacidade` publicado pelo RootifyONE (vence) ou do config do app (padrão): **`so-verde`** nos apps com
   dado pessoal ou de saúde (OmniLifeONE, MoneyTRIO, RiseONE, HyperNutry, Contador, Leitor OCR, apps jurídicos e
   corporativos), **`qualquer`** nos públicos (CifrasONE, RadarONE, Bolinho de Chuva).
3. **Quem paga** (seção 8): separa os candidatos da pessoa dos da plataforma.
4. **Disponível agora:** tem chave (ou porteiro), está com internet, a rede deixa (`DGO.rede.podeUsarIA`), não está
   "esfriando" e a cota local do minuto e do dia não acabou.
5. **Ordem:** pela tarefa —
   - `rapida`: grátis mais rápido primeiro (Groq, Gemini Flash-Lite, Z.ai);
   - `normal`: grátis primeiro, na ordem de preferência da pessoa (se escolheu uma), depois pagos do mais barato ao
     mais caro;
   - `caprichada`: o melhor modelo **pago** que a pessoa tiver (Claude, GPT, Gemini Pro), depois os grátis.
6. **Tentar em sequência**, com prazo (25 s texto, 60 s visão):
   - 429/503/529 → marca "esfriando" pelo `Retry-After` (ou 60 s) e passa ao próximo, avisando por `aoTrocar`;
     se for o último, espera 6/15/30 s como o `DGO.ia` faz;
   - 401/403 → chave recusada: marca, avisa a pessoa uma vez ("a chave do Groq foi recusada — copie de novo"), passa
     ao próximo;
   - 404 / modelo aposentado → troca o modelo sozinho (lista da chave) e tenta uma vez;
   - 402 / sem crédito → marca sem crédito até amanhã, passa ao próximo;
   - erro de rede → tenta o próximo; sem internet → erro `sem-internet` na hora.
7. Esgotou tudo: erro `cota` (ou `privacidade`, se havia provedor bloqueado só pela regra), com o que fazer.

A **cota local** é um contador por provedor (minuto e dia, dia pelo fuso do provedor quando conhecido, senão UTC),
guardado no `localStorage` com o prefixo do motor e **sem a chave nem o conteúdo**. É uma estimativa: serve para
desviar antes de bater no limite, não para garantir; o 429 de verdade continua sendo tratado.

## 8. Quem paga

Preferência da pessoa em ⚙ → IA (vale para todos os apps da mesma origem): **Automático** (padrão) · **Sempre a
minha chave** · **Sempre a da SolverONE**.

Automático:
1. a pessoa tem chave própria que serve (capacidade + regra) → usa a dela, sem gastar crédito;
2. senão, se o app tem porteiro ligado e a pessoa tem crédito no plano → plataforma;
3. senão → erro `sem-provedor` com dois caminhos: "Usar a IA grátis com a sua chave (2 minutos)" (abre o guia) e,
   se houver, "Conhecer o plano com IA".

`caprichada` inverte 1 e 2 **só** se a pessoa não tiver chave paga própria e o plano incluir o modelo bom.
Em empresa cliente, "plataforma" vira "empresa": as chaves dela no porteiro, os funcionários não as veem.

## 9. O porteiro (fase 3, desenhado agora para a fase 1 não fechar portas)

- Edge Function `linkone` no `solverone-app` (a `solverone-ia` continua para voz e histórias até migrar).
- Pedido: `{ app, tarefa, capacidade, privacidade, mensagens, sistema, limite }` com o token da conta SolverONE;
  resposta no mesmo formato do motor.
- Confere no banco, com o token da pessoa: plano, créditos do mês e do dia, teto do mês da plataforma (reaproveita
  `reservar_uso_servico` / `estornar_uso_servico`, trocando "unidades" por **créditos**: pergunta simples 1, foto 3,
  texto longo 10).
- Chaves da plataforma em **Secrets**; chaves de empresa no **Vault** do Supabase, por empresa, nunca devolvidas.
- Roda o **mesmo roteador** do motor (o arquivo é JavaScript puro e roda no Deno), com os provedores que só
  funcionam no servidor (Cloudflare, SambaNova) e os pagos da plataforma.
- Grava só **metadados**: quando, app, provedor, modelo, tokens, créditos, custo. **Nunca** o texto.
- Todo SQL novo segue a regra de GRANT explícito e RLS no mesmo arquivo (`teste/sql-grants.js`).

## 10. Segurança e privacidade

- Chave da pessoa: só no `cofre.js`; o motor pede a chave na hora do pedido e não guarda cópia.
- CSP de cada app: o `connect-src` passa a listar só os endereços do registro em uso (o `arquitetura.json` também).
- Nenhum conteúdo de pergunta em log, telemetria, feedback ou erro guardado.
- Gancho `antesDeEnviar(mensagens)` no ponto único: troca nomes por `[NOME1]` (regra de criança, LGPD art. 14) e o
  app pode acrescentar outros; `depoisDeReceber(texto)` restaura no aparelho.
- "O que a IA recebe junto" visível antes de mandar dado do app (widget já previsto na diretriz).
- Provedor "Personalizado": selo 🟡 até a pessoa marcar, e aviso de que o endereço recebe tudo o que for perguntado.
- LGPD: cada provedor usado pela plataforma entra no registro de tratamento (ROPA) e na política de privacidade, com
  país (transferência internacional, art. 33); dado de saúde só com consentimento específico (art. 11).

## 11. IA com os dados da família (fase 6, só o princípio agora)

Os dados da família são cifrados no aparelho; um índice no servidor quebraria essa proteção. Por isso a busca é
**no aparelho**: o app procura os trechos ligados à pergunta, troca nomes por códigos, mostra o que vai junto e manda
só isso, com regra `so-verde`. Nenhum índice ou embedding de dado pessoal no servidor.

## 12. Fases

| Fase | O quê | Depende de |
|---|---|---|
| 1 | motor no navegador (texto e foto), registro, roteador, selos, cota local, quem paga (só "pessoa"), testes | aprovação deste desenho |
| 2 | `DGO.ia` delega ao LinkONE; piloto num app (sugestão: RootifyONE, que já usa IA nos termos) | fase 1 |
| 3 | porteiro (plataforma), créditos, Cloudflare e SambaNova, **imagem** (Cloudflare FLUX; pagos: Gemini, GPT Image, Grok) | fase 2; "pode aplicar" do dono para SQL e Secrets |
| 4 | painel no RootifyONE: provedores, regra por app, créditos por plano, teto do mês, custo | fase 3 |
| 5 | empresas clientes com chaves próprias no porteiro | fase 4; contrato/termos B2B |
| 6 | IA com os dados da família (busca no aparelho), começando pelo OmniLifeONE | fase 3; revisão LGPD com advogado |

## 13. Testes (fase 1)

`node teste/rodar.js`, sem dependência, com `fetch` falso que imita cada provedor:
- escolha por capacidade, por regra de privacidade (nenhum 🔴 com `so-verde`), por tarefa;
- 429 → esfria e troca; 401 → marca recusada; 404 → troca de modelo; 402 → sem crédito; rede caída;
- cota local: para de usar ao chegar no limite do minuto e do dia, volta no dia seguinte;
- quem paga: Automático, Sempre a minha, sem chave nenhuma;
- cancelar pelo `sinal` no meio da espera;
- nenhum valor de chave aparece em `localStorage`, log ou erro;
- `linkone.js` igual ao produzido por `ferramentas/juntar.js`;
- adaptadores: corpo e cabeçalhos certos para OpenAI, Gemini e Anthropic (incluindo imagem em base64).

Revisão independente (outra conversa ou subagente) antes do PR.

## 14. Definição de pronto: os 14 pontos neste repositório

Estágio do LinkONE hoje: **biblioteca sem tela própria** (parecido com "R Robô", mas roda dentro dos apps).

| Ponto | Situação |
|---|---|
| 1 Manual e orientações | vale: `README.md`, `ARQUITETURA.md`, mensagens de erro para leigo |
| 2 AssistONE | não aplicável hoje: o motor não tem tela; a ajuda das telas de IA é do app |
| 3 Novidades da versão | vale no `versoes.json` deste repositório; nos apps, só quando a pessoa notar diferença |
| 4 Termos | não aplicável hoje: a fase 1 não coleta nada; volta na fase 3 (porteiro) |
| 5 Comunicações e e-mails | não aplicável hoje: nenhum e-mail |
| 6 RootifyONE | vale: comportamento `linkone.privacidade` e o interruptor `ia` já existente |
| 7 ArchONE | vale: `arquitetura.json` deste repositório desde a fase 1 |
| 8 EaglePlanONE | vale quando houver item do roadmap; até lá, `PENDENCIAS.md` |
| 9 Retrocompatibilidade | vale: mesmas chaves do cofre e mesmas preferências do `DGO.ia`; sem o LinkONE, tudo como antes |
| 10 Segurança e LGPD | vale: seção 10 |
| 11 Qualidade verificável | vale: seção 13 |
| 12 Registro | vale: `PENDENCIAS.md` e resumo de cada entrega |
| 13 Entrega técnica | vale: `versoes.json`, versão no código, zip `LINK-ONE vX.Y.Z …`, `ARQUITETURA.md` |
| 14 Capacidade e custo | vale: limites de cada provedor no registro e em `arquitetura.json` (`capacidades`); teto de gasto no painel de cada fornecedor pago |

## 15. Riscos

- **Limites grátis mudam sem aviso** (em 2026 o GitHub Models fechou e a Cerebras tirou o grátis): o registro tem
  data de conferência e fonte por provedor, e o roteador sobrevive a qualquer um sumir.
- **Selo errado** é o risco mais grave para cliente jurídico: na dúvida, 🟡; só vira 🟢 com página oficial citada.
- **Estimativa de cota diverge da real:** o 429 de verdade continua mandando.
- **Mesma origem:** um script ruim na página, com o cofre aberto, poderia usar as chaves (limite já registrado na
  diretriz); por isso CSP sem script de terceiros.

## 16. O que o dono ainda decide (negócio, não técnico)

1. Repositório público ou privado (não muda nada técnico: não há site; a cópia do motor dentro dos apps é pública de
   qualquer jeito).
2. Créditos por plano e preço (fase 3/4).
3. Em quais apps a plataforma paga a IA (fase 3).
