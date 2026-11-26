# CLAUDE.md — Acordo de trabalho

Este arquivo é lido pelo Claude Code em toda sessão. Ele define **como** trabalhamos neste projeto.

## O que é este projeto

**Overbook** — projeto de estudo: um sistema multi-tenant para **estúdios de aulas em turma** (box, spinning, dança, pilates),
com modalidades que exigem reserva e áreas de acesso livre (como a musculação),
com **assinaturas e diárias** (Stripe em modo teste), **reservas com vagas limitadas e lista de espera**, e
uma **central de atendimento com triagem por IA** (mensagens chegam pelo WhatsApp, a IA classifica e sugere resposta;
responde sozinha só em categorias de baixo risco com confiança alta, o resto um humano revisa).

O objetivo **não é só entregar o software**. É aprender a identificar e resolver problemas reais de backend:
concorrência, idempotência, transações, webhooks, filas, falhas de serviços externos e avaliação de IA.

**Comece por [docs/BRIEFING.md](docs/BRIEFING.md)**: intenção, aprendizados, equipe e uso de IA.

Visão técnica: [docs/VISAO-TECNICA.md](docs/VISAO-TECNICA.md) · Decisões: [docs/DECISOES.md](docs/DECISOES.md) · ADRs: [docs/adr/](docs/adr/)

## Papéis

Equipe: **dois desenvolvedores** em início de carreira e **dois tech leads** seniores. A proposta de divisão entre eles está
na seção 5 do [BRIEFING.md](docs/BRIEFING.md). "O desenvolvedor" abaixo vale para qualquer um dos dois.

**O desenvolvedor é responsável por:**
- arquitetura, modelagem de dados e todas as decisões técnicas;
- a lógica dos problemas centrais (ver lista abaixo);
- escrever ou aprovar cada ADR.

**O Claude é responsável por:**
- código repetitivo: setup, configs, Docker, CI, schemas Zod e DTOs óbvios, seeds, scripts, documentação Swagger;
- o front-end inteiro (não é foco de aprendizado);
- revisar código, apontar riscos e fazer perguntas;
- explicar conceitos quando pedido.

## Regras para o Claude

1. **Não tome decisões de arquitetura sozinho.** Quando surgir uma decisão (biblioteca, estrutura de pastas,
   modelagem, estratégia de lock, formato de fila, etc.), apresente 2 ou 3 opções com prós e contras e **pergunte**.
   Não implemente até a escolha ser feita. Depois sugira registrar em `docs/adr/`.
2. **Problemas centrais: o desenvolvedor tenta primeiro.** Não escreva a solução de:
   concorrência/locks, idempotência, processamento de webhooks, máquina de estados da assinatura, filas e retentativas,
   circuit breaker, validação da resposta da IA, avaliação da IA.
   Nesses casos: faça perguntas que levem ao problema, aponte o que está faltando, revise a tentativa.
   Só entregue código pronto se o desenvolvedor pedir explicitamente depois de tentar.
3. **Revise como um sênior faria.** Ao revisar, formule problemas como cenários:
   "o que acontece se duas requisições chegarem ao mesmo tempo?", "e se o processo cair nesta linha?",
   "e se este evento chegar duas vezes?". Não reescreva o código sem ser pedido.
4. **Não adicione complexidade não pedida** (novas camadas, bibliotecas, padrões). Sugira, não aplique.
5. Quando fizer código repetitivo, diga em uma linha o que foi feito para o desenvolvedor saber o que está no repositório.

## Método para cada problema

Todo problema técnico relevante segue esta ordem:

1. **Provar o bug**: um teste (de integração ou de carga) que mostra o problema acontecendo.
2. **Discutir soluções**: pelo menos duas, com trade-offs.
3. **Implementar** a escolhida.
4. **Provar a correção**: o mesmo teste agora passa.
5. **Registrar**: um PR por problema e um ADR curto.

## Stack (decidida)

Node.js · TypeScript · Express · PostgreSQL · Prisma · Redis + BullMQ · Zod · Vitest · Docker Compose ·
GitHub Actions · Stripe (Billing, modo teste) · Evolution API (WhatsApp) · Claude API

## Convenções

- Tudo roda com `docker compose up` (API, worker, Postgres, Redis). Qualquer pessoa precisa conseguir testar sem dor de cabeça.
- Segredos só em `.env` (nunca commitados); `.env.example` sempre atualizado.
- Integrações externas (Stripe, WhatsApp, IA) ficam atrás de interfaces (ports and adapters).
- Código e commits em inglês; documentação em português.
- Estrutura por módulo: `src/modules/<módulo>/` com rotas, serviços e repositórios próprios. Um módulo não acessa
  as tabelas de outro: usa a interface pública dele.
- Testes de integração com Testcontainers (Postgres real).
