# SolverONE — regras do projeto para o Claude Code

Antes de qualquer alteração neste repositório, leia e siga **inteira** a diretriz geral da plataforma em
`.claude/skills/diretrizes-gerais-apps/SKILL.md` (ela também é carregada como skill `diretrizes-gerais-apps`).

Resumo do que nunca pode ser esquecido (o detalhe está na diretriz):

- **Pendências do dono**: o que só o dono decide ou faz fica em `docs/PENDENCIAS-DO-DONO.md` (lista viva); toda conversa
  de diretrizes lê no começo e atualiza ao entregar, para nada ficar perdido no contexto.

- **Regra zero**: antes de implementar no app um requisito da diretriz que ainda não existe aqui, liste o que
  falta e confirme com o dono. Não crie repositórios sem permissão explícita.
  **Publicar é sempre autorizado** (dono, 02/Nov/2026): ao terminar e testar, faça o pull request e o merge sem
  perguntar e conte o que mudou.
- **Segurança**: nenhuma senha, chave de API, token ou segredo entra no código, no repositório, no chat ou em
  arquivo. Chaves de IA ficam só no cofre do navegador da pessoa. Se uma chave aparecer no chat, peça para revogar.
- **Idioma** (dono, 14/Nov/2026): textos para o usuário em **PT (Brasil), EN (EUA) e ES (Espanha)**, tradução
  humanizada; **um seletor só em todo lugar**: um botão com bandeira + sigla do idioma atual que, tocado, pergunta se continua ou
  troca (🇧🇷 PT · 🇺🇸 EN · 🇪🇸 ES), componente do módulo comum,
  nunca toggle ou dropdown próprio; ES só aparece num app quando ele está 100 % traduzido (até lá, PT|EN). Detalhe na
  diretriz, "Idioma e datas". Instruções para o dono em português, passo a passo, como para iniciante (ele não é
  desenvolvedor).
- **Entrega**: um `.zip` por app, nome `<NOME APP> <VERSAO> <dd-Mmm-aaaa> <HHhMMm>.zip`, arquivos soltos na
  raiz; atualizar `versoes.json`, a versão no código, o `sw.js` e `ARQUITETURA.md`/`PENDENCIAS.md`.
- **Trabalho demorado** (foto, OCR, IA, importação, áudio) roda no módulo `Fundo`: nunca morre ao sair da tela,
  Wake Lock, aviso antes de fechar, pílula de andamento, resultado entregue onde a pessoa estiver.
- **Interruptores do RootifyONE** (Controle dos apps) valem de verdade em todo app, atual e futuro:
  `data-recurso="id"` + `DGO.recursos` (ou `recursos.js`) e a lista `recursos-do-app.json` do que o
  app obedece. Nada de segredo nesses arquivos; trava de verdade fica no banco.
- **Feedback dos usuários**: pergunta do dia, 💬 em toda tela e no AssistONE, tudo estruturado no RootifyONE.
  **Chamados pelo AssistONE** (dono, 12/Nov/2026): "🎫 Abrir um chamado" no balão, já com tela, último clique e versão,
  categorias prontas e "Outros"; vai para a central de feedback (que vira chamado) com internet, fica guardado sem
  internet; anexo só pelo RootifyONE (URL assinada, limites por dia); nada pessoal sai sem a pessoa ver antes.
- **Campos de CPF e CNPJ** (dono, 12/Nov/2026): aceitam vazio; números digitados ganham máscara
  (`000.000.000-00`, `00.000.000/0000-00`) e dígitos verificadores conferidos; CNPJ alfanumérico (IN RFB 2.229/2024,
  12 caracteres com letras maiúsculas e números + 2 dígitos numéricos, cálculo com ASCII − 48); inválido avisa e não
  salva; nome continua livre.
- **Planilha "0-SolverONE DIRETRIZES E BACKLOG"** (dono, 12/Nov/2026): arquivo único no Google Sheets (versão = Histórico de
  versões, nunca arquivo novo); linha nova no fim da aba; ninguém renumera o "#" nem reaproveita número; não ordenar só um
  pedaço da aba. Vale até existir a coluna "⚙ ID" (criada só pelo roadmap-one). Detalhe na diretriz, "Planilha de controle".
- **Definição de pronto, 14 pontos e regra de aplicabilidade** (dono, 11 a 13/Nov/2026): toda mudança que muda o que a
  pessoa vê ou o que o app faz só está pronta com os pontos **aplicáveis ao estágio do app** conferidos contra o que o
  programa faz, em PT e EN: (1) manual e orientações; (2) AssistONE; (3) Novidades da versão; (4) termos; (5) comunicações
  e e-mails; (6) RootifyONE; (7) ArchONE; (8) EaglePlanONE; (9) retrocompatibilidade e migração; (10) segurança e LGPD;
  (11) qualidade verificável; (12) registro; (13) entrega técnica e dependências; (14) capacidade e custo. O que não se aplica hoje: "não aplicável hoje" e o motivo em uma linha;
  nada de login, banco ou integração só por causa da diretriz. Detalhe e tabela por app na diretriz e em
  `docs/specs/definition-of-done-e-teia.md`. Gestão dos apps e CRM ficam no mesmo RootifyONE, em dois módulos, com um
  CRM só de duas visões (dono, 13/Nov/2026; seção própria na diretriz).
---

# LinkONE — o que é este repositório

**LinkONE** é o motor que liga os apps às IAs (nome do dono, 14/Nov/2026: "pode ser conector de outras coisas e não
só de IA"). A IA é o primeiro tipo de conector; outro tipo só entra quando o dono pedir.

- **Não é site.** Não usa GitHub Pages. O arquivo do motor (`linkone.js`) tem o master aqui e vai **copiado igual**
  para cada app, como o `cofre.js` e o `janela.js`: nunca editar a cópia de um app; mudar aqui e copiar.
- **Três peças** (decisão técnica delegada pelo dono, 14/Nov/2026): (1) o motor no navegador; (2) o **porteiro**, uma
  Edge Function do Supabase da plataforma (`solverone-app`) que guarda as chaves da plataforma e das empresas e confere
  plano e créditos no banco; (3) o painel no RootifyONE. Desenho completo: `docs/specs/2026-11-14-linkone-design.md`.
- **Chaves:** as da pessoa só no `cofre.js` (nomes `ia:<provedor>`, os mesmos do `DGO.ia`); as da plataforma e das
  empresas só nos Secrets/Vault do Supabase. Nunca no código, no chat, em teste ou em arquivo.
- **Privacidade:** cada provedor tem um selo (🟢 não treina, 🟡 não informado, 🔴 treina, ⛔ só teste) e cada app uma
  regra; o motor nunca manda para um provedor que a regra do app proíbe. Revenda de chave ou proxy anônimo nunca entra
  no registro de provedores.
- **Testes** (a partir da fase 1): `node teste/rodar.js` (sem dependência; `fetch` falso, nunca chama provedor de verdade nem usa chave).
- **Aviso do dono de 13/Nov/2026 (Definição de pronto e teia):** cada um dos 14 pontos vale quando for aplicável ao
  estágio deste repositório; o que não se aplica, "não aplicável hoje" e o motivo em uma linha. Não criar login, banco
  ou integração por causa do aviso. Em cada entrega, dizer no resumo o que o RootifyONE, o ArchONE ou o EaglePlanONE
  precisariam saber. Nenhuma integração nova sem a regra zero. Quando sair a diretriz nova, segui-la.
