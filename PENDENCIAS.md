# LinkONE — pendências (lista oficial e viva)

Qualquer conversa nova lê este arquivo primeiro. A cada entrega a situação é atualizada aqui e o resumo diz o que
fechou. Item novo entra no fim, com o próximo número (números não mudam nem são reaproveitados). Criada em
14/Nov/2026, junto com o repositório.

**Situações:** 🟡 aberta · 🔵 em andamento · ⏸ aguardando decisão do dono · ✅ feita (com a versão)

| # | O quê | Origem | Situação |
|---|---|---|---|
| 1 | Revisar o desenho (`docs/specs/2026-11-14-linkone-design.md`) e dar o "pode" para a fase 1 | conversa de 14/Nov/2026 | ⏸ aguardando o dono |
| 2 | Repositório público ou privado (não muda nada técnico) | conversa de 14/Nov/2026 | ⏸ aguardando o dono |
| 3 | Fase 1: motor no navegador (texto e foto), registro de provedores, roteador, selos de privacidade, cota local, testes | desenho, seção 12 | 🟡 aberta (depois do #1) |
| 4 | Fase 2: `DGO.ia` delega ao LinkONE; piloto no RootifyONE | desenho, seção 12 | 🟡 aberta |
| 5 | Fase 3: porteiro no Supabase, créditos, Cloudflare e SambaNova, geração de imagem | desenho, seção 12 | 🟡 aberta |
| 6 | Fase 4: painel no RootifyONE (provedores, regra por app, créditos por plano, teto do mês, custo) | desenho, seção 12 | 🟡 aberta |
| 7 | Fase 5: empresas clientes com chaves próprias no porteiro | desenho, seção 12 | 🟡 aberta |
| 8 | Fase 6: IA com os dados da família (busca no aparelho), começando pelo OmniLifeONE; revisão LGPD com advogado | desenho, seções 11 e 12 | 🟡 aberta |
| 9 | Créditos por plano e preço; em quais apps a plataforma paga a IA | desenho, seção 16 | ⏸ aguardando o dono (fase 3/4) |
| 10 | No `diretrizes.js`, o modelo padrão do Claude é antigo (corrige sozinho no erro, mas convém atualizar) e faltam Groq, Z.ai e outros no registro: resolver na fase 2, no master do rootify-one | revisão de 02/Nov/2026 | 🟡 aberta |
| 11 | Porteiro antecipado: `solverone-ia` 1.1.0 no ar com o OpenRouter (`openrouter/free`) para as histórias do Contador; chave `OPENROUTER_API_KEY` com teto de US$ 1/mês; serviço configurado direto no banco | conversa de 25/Nov/2026 (solverone-app) | ✅ solverone-ia 1.1.0 |
| 12 | Um porteiro só: a `solverone-ia` é o porteiro (desenho, seção 9); mover o código-mestre para este repositório no começo da fase 3 | dono delegou, 25/Nov/2026 | 🟡 aberta |
| 13 | Migrar a função `ia` do GuiaXpert (Gemini, nasceu no FeatureTesting, fora do padrão) para o porteiro, com a busca na web | dono, 25/Nov/2026 | 🟡 aberta (depois do #12) |
| 14 | Contador de Histórias com regra `qualquer` (só criação criativa; nomes por variáveis preenchidas pelo app) | dono, 25/Nov/2026 | ✅ decidido (desenho, seção 7) |
| 15 | "Quem paga" confirmado: IA da plataforma no plano pago **e** chave própria do cliente, convivendo | dono, 25/Nov/2026 | ✅ decidido (desenho, seção 8) |
| 16 | Hub de IA e Agentes: VPS Hostinger KVM 2 (WhatsApp por API não oficial com número dedicado, MCP de Gmail e Agenda só leitura, Hermes Agent); modelos chamados pela internet, nunca na VPS; API oficial do WhatsApp descartada (custo) | dono, 25/Nov/2026; pesquisa na conversa de 26/Nov | ⏸ pesquisa de quinta 26/Nov |
| 17 | AssistONE com IA simples: dúvidas sobre o app respondidas pelo porteiro com modelo grátis e rápido e o manual como contexto; trava contra abuso para visitante sem login | conversa de 25/Nov/2026 | 🟡 aberta |
| 18 | RootifyONE: provedor `openrouter` (e campo de opções) no cartão de serviço; hoje o `historia-ia` foi configurado direto no banco e salvar o cartão pode desfazer | conversa de 25/Nov/2026 | 🟡 aberta (repositório rootify-one) |

## Não aplicável hoje (Definição de pronto)

- AssistONE: o motor não tem tela própria.
- Termos: a fase 1 não coleta nada; volta na fase 3 (porteiro).
- Comunicações e e-mails: nenhum e-mail.
- EaglePlanONE: o hub está na caixa de entrada do EaglePlanONE (`roadmap-one`, `docs/ENTRADA-DO-DONO.md`, E23, 25/Nov/2026); vira item do roadmap quando ele tiver backlog no banco.
