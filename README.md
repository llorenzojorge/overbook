# Overbook

**Sistema de reservas para estúdios de aulas em turma que nunca faz overbooking.**

Overbooking é confirmar mais reservas do que vagas, como duas pessoas reservando a última vaga de uma aula no mesmo
instante. Evitar esse bug, e outros do mesmo tipo, é o assunto deste projeto.

> 🚧 **Status:** em planejamento. O código ainda não começou.

## O que o sistema faz

- **Planos:** assinatura recorrente (mensal, trimestral, anual) ou pacote de créditos, com Stripe em modo teste.
- **Reservas:** turmas com vagas limitadas, cancelamento com prazo e lista de espera.
- **Atendimento inteligente:** alunos falam com o estúdio pelo WhatsApp; uma IA classifica as mensagens e sugere
  respostas. Ela responde sozinha só em casos de baixo risco; o resto passa por um atendente.

## Os problemas que o projeto resolve

Fazer uma API funcionar é a parte fácil. Este projeto existe para **identificar problemas reais de backend, provar
cada um com um teste e só então corrigir**:

| Problema | Exemplo no estúdio |
|---|---|
| **Concorrência** | 30 alunos tentam reservar as 12 vagas de uma aula no mesmo segundo |
| **Idempotência** | Clique duplo em "Assinar" não pode gerar duas assinaturas |
| **Transações** | Reservar ocupa a vaga e debita o crédito: tudo ou nada |
| **Webhooks** | O aluno paga e fecha o navegador; o sistema descobre pelo webhook, que pode chegar duplicado ou fora de ordem |
| **Máquina de estados** | Assinatura ativa, atrasada ou cancelada, com transições inválidas rejeitadas |
| **Filas e resiliência** | Nenhuma mensagem se perde, mesmo com o worker caindo ou a IA fora do ar |
| **IA em produção** | Resposta da IA validada, proteção contra prompt injection e avaliação com dados rotulados |

Cada problema segue o mesmo método: **teste que prova o bug → opções com trade-offs → correção → o mesmo teste
passando → ADR**.

## Stack

Node.js · TypeScript · Express · PostgreSQL · Prisma · Redis + BullMQ · Zod · Vitest + Testcontainers ·
Docker Compose · GitHub Actions · Stripe Billing · Evolution API (WhatsApp) · Claude API

## Documentação

- [Briefing](docs/BRIEFING.md): origem, objetivos, produto, equipe e uso de IA
- [Decisões](docs/DECISOES.md): o que já foi decidido e o que está em aberto
- [ADRs](docs/adr/): registro detalhado de cada decisão de arquitetura
- [CLAUDE.md](CLAUDE.md): como usamos IA no desenvolvimento

## Equipe

Dois desenvolvedores e um tech lead.
