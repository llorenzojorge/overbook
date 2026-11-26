# Overbook

**Sistema de reservas para estúdios de aulas em turma que nunca faz overbooking.**

Overbooking é confirmar mais reservas do que vagas, como duas pessoas reservando a última vaga de uma aula no mesmo
instante. Evitar esse bug, e outros do mesmo tipo, é o assunto deste projeto.

> 🚧 **Status:** em planejamento. O código ainda não começou.

## O que o sistema faz

- **Planos:** assinatura recorrente (mensal, trimestral, anual) ou diária avulsa, com Stripe em modo teste.
- **Estúdios:** vários estúdios no mesmo sistema, cada um com suas modalidades. Aulas em turma exigem reserva;
  áreas como a musculação são de acesso livre.
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
| **Transações** | Confirmar uma diária registra o pagamento e garante a vaga: tudo ou nada |
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

- [Visão técnica](docs/VISAO-TECNICA.md): como o sistema funciona (atores, fluxos e regras de negócio)
- [Briefing](docs/BRIEFING.md): motivação, objetivos, produto, equipe e uso de IA
- [Decisões](docs/DECISOES.md): o que já foi decidido e o que está em aberto
- [ADRs](docs/adr/): registro detalhado de cada decisão de arquitetura
- [CLAUDE.md](CLAUDE.md): como usamos IA no desenvolvimento

## Equipe

| Papel | Pessoa |
|---|---|
| Desenvolvedor | Lorenzo Jorge ([@llorenzojorge](https://github.com/llorenzojorge)) |
| Desenvolvedor | Matheus ([@matheuscostabeber](https://github.com/matheuscostabeber)) |
| Tech lead | Vinícius ([@eovinicius](https://github.com/eovinicius)) |
| Tech lead | Maurício |

Os requisitos, a arquitetura e a modelagem de dados são feitos pelos desenvolvedores; os tech leads orientam e
revisam. A IA é usada com regras definidas no [CLAUDE.md](CLAUDE.md).

## Licença

[MIT](LICENSE)
