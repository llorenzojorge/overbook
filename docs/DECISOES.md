# Decisões

Resumo rápido. Decisões com impacto real na arquitetura ganham um ADR em [adr/](adr/).

## Tomadas

| # | Decisão | Motivo |
|---|---|---|
| 1 | Projeto novo, juntando **assinaturas/pagamentos** e **atendimento com IA** | Os dois domínios se conectam (as reclamações são sobre a assinatura) e cobrem os conceitos citados pelo CTO |
| 2 | **Stripe** em modo teste | Modo teste completo, Stripe CLI para webhooks locais, documentação forte, conhecido no Brasil e em Portugal |
| 3 | **Stripe Billing** controla a recorrência | Construir o motor de cobrança à mão seria muito trabalho; o aprendizado que interessa está em sincronizar o estado via webhooks |
| 4 | **Claude API** como IA | Isolada atrás da interface `TicketClassifier`, para poder comparar modelos na avaliação |
| 5 | **Front-end feito por IA**, só para apresentação | Não é foco de aprendizado agora |
| 6 | Canal: **WhatsApp via Evolution API**, com ports and adapters | Gratuito para testar; trocar pela API oficial depois exige só um novo adapter. Também existe um adapter HTTP para testes |
| 7 | **Express + Prisma** | Já conhecidos: energia vai para os conceitos novos, não para a ferramenta |
| 8 | Tudo em **Docker** + deploy público | Qualquer pessoa consegue rodar com um comando; link funcionando para demonstração |
| 9 | Desenvolvedores decidem a arquitetura; Claude faz o repetitivo e revisa | Ver [CLAUDE.md](../CLAUDE.md) |
| 10 | Monólito modular + worker (sem microsserviços) | Adequado ao tamanho do projeto |
| 11 | Produto: **estúdio de aulas em turma** (substitui a ideia inicial de plataforma de cursos) | Tem algo escasso, as vagas de cada aula, o que gera concorrência de verdade. Planos ilimitados e por pacote de créditos. Detalhes em [BRIEFING.md](BRIEFING.md) |
| 12 | Código e commits em **inglês**, documentação em português | Padrão de mercado no código |
| 13 | Estrutura **por módulo** (`src/modules/<módulo>/`), cada um com suas rotas, serviços e repositórios | Os módulos ficam visíveis e um módulo conversa com o outro por uma interface pública, sem ler as tabelas do outro. Os módulos em si serão definidos no ADR 0001 |
| 14 | Testes de integração com **Testcontainers** | Postgres isolado e igual no CI. A opção de transação com rollback foi descartada porque esconde bugs de concorrência |
| 15 | Autenticação por **sessão com cookie, guardada no Redis** | Logout e revogação imediatos; conceito novo para aprender (cookies, CSRF). O status da assinatura nunca vai na sessão: é consultado no banco a cada reserva |
| 16 | Nome do projeto: **Overbook** | Overbooking é confirmar mais reservas do que vagas: o nome aponta direto para o problema central de concorrência. A descrição do repositório explica que é um sistema de reservas para estúdios |
| 17 | Foco do projeto: **aprendizado e trabalho em equipe** | Projeto de estudo, não comercial |
| 18 | IA com **autonomia gradual** | Responde sozinha só em categorias de baixo risco e com confiança acima de um limite escolhido pela avaliação; o resto vai para revisão humana. Começa com revisão humana em tudo. Detalhes na seção 6.2 do [BRIEFING.md](BRIEFING.md) |
| 19 | Licença **MIT** | A mais comum e simples para projetos abertos: qualquer pessoa pode ler, usar e adaptar o código, mantendo o aviso de autoria |

## Em aberto

| Decisão | Quando decidir | Observações |
|---|---|---|
| Trabalho em equipe (2 devs + 1 tech lead) | Antes de começar o código | Proposta na seção 5 do BRIEFING.md; falta combinar com o tech lead |
| Módulos, modelagem e ordem das fases | Depois do levantamento de requisitos | ADR 0001. Ver seções 8.2 e 9 do BRIEFING.md |
| Modelo do Claude para classificar | Fase de integração com IA | Um modelo pequeno (família Haiku) costuma bastar para classificação; a avaliação confirma |
| Número de WhatsApp para testes | Fase de mensagens | A Evolution não é a API oficial: há risco de bloqueio do número. Preferir um chip secundário |
| Onde fazer o deploy | Quando a parte de cobrança estiver completa | |
