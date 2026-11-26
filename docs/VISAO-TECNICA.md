# Overbook — Visão técnica

> Resumo de **como o sistema funciona**, para apoiar a arquitetura e a modelagem de dados.
> A intenção do projeto, a equipe e o uso de IA estão no [BRIEFING.md](BRIEFING.md); as decisões, no
> [DECISOES.md](DECISOES.md).
>
> **Estado:** requisitos em levantamento. Itens marcados com **(em aberto)** ainda não foram decididos.

---

## 1. O sistema em um parágrafo

O sistema atende **vários estúdios** (multi-tenant). Cada estúdio tem suas modalidades, salas e instrutores.
Uma modalidade **exige reserva** (aulas em turma com **vagas limitadas** pela sala, como box, spinning ou dança) ou é
**de acesso livre** (como a área de musculação, usada sem reservar). O aluno paga um **plano ilimitado** (assinatura recorrente)
ou uma **diária**, e com isso usa a área livre e **reserva** vagas nas aulas. Turma cheia → o aluno entra na **lista de
espera**. O aluno fala com o estúdio pelo **WhatsApp**; cada mensagem é **classificada por IA**, que sugere uma resposta.
Um atendente revisa as sugestões; com o tempo, a IA passa a responder sozinha em casos de baixo risco.

## 2. Atores

| Ator | O que faz no sistema |
|---|---|
| **Aluno** | Cadastra-se, faz login, assina um plano ou paga diária, reserva e cancela aulas, entra na lista de espera, manda mensagens |
| **Atendente / recepção** | Vê as mensagens classificadas, aprova, edita ou rejeita a resposta sugerida, assume conversas escaladas |
| **Proprietário do estúdio** | Administra o próprio estúdio: modalidades, salas, turmas, horários e instrutores (pode ser via seed no início). Define os papéis e edita os dados de alunos e funcionários |
| **Instrutor** | Dá as aulas das modalidades a que está vinculado |
| **Equipe do sistema (admin)** | Os desenvolvedores e os tech leads. Cria e desativa estúdios |
| **Stripe** | Cobra o aluno e avisa o sistema por webhook |
| **Evolution API** | Entrega ao sistema as mensagens do WhatsApp e envia as respostas |
| **Claude API** | Classifica a mensagem e sugere a resposta |

## 3. Fluxos principais

### 3.1 Assinar o plano ilimitado

1. O aluno escolhe o período (mensal, trimestral ou anual) e clica em "Assinar".
2. O sistema cria um checkout no Stripe e redireciona o aluno.
3. O aluno paga no Stripe. Ele pode fechar o navegador antes de voltar ao site.
4. O Stripe envia um **webhook** dizendo que o pagamento foi confirmado.
5. O sistema verifica a assinatura do webhook, registra o evento e ativa o plano.
6. A partir daí, renovações, falhas de cartão e cancelamentos também chegam por webhook.

### 3.2 Pagar uma diária

1. O aluno sem plano paga a diária por um checkout avulso do Stripe.
2. O webhook de pagamento confirmado libera a reserva. Registrar o pagamento e garantir a vaga: **tudo ou nada**.
3. A diária vale por um dia, para a área livre e para as aulas com reserva.
4. O aluno paga antes ou depois de garantir a vaga? E se o pagamento for confirmado e a aula já estiver cheia?
   **(em aberto)**

### 3.3 Reservar uma aula

1. O aluno escolhe uma aula com vaga.
2. O sistema confere se o plano permite reservar (plano ativo? atrasado? diária paga?).
3. Ocupa a vaga. Para quem paga diária, a reserva depende do pagamento (3.2).
4. Se não há vaga, o aluno pode entrar na lista de espera.

### 3.4 Cancelar uma reserva

1. O aluno cancela até um prazo antes da aula **(prazo em aberto)**.
2. Diária cancelada: tem estorno? Dentro e fora do prazo **(em aberto)**.
3. A vaga liberada vai para o próximo da lista de espera (3.5).

### 3.5 Lista de espera

1. Turma cheia → o aluno entra na fila daquela aula.
2. Quando uma vaga abre, o próximo da fila é promovido e avisado.
3. A promoção é automática ou o aluno recebe uma oferta e precisa aceitar? Se for oferta, por quanto tempo?
   **(em aberto)**

### 3.6 Ciclo de vida da assinatura

- Estados previstos: `pending → active → past_due → canceled`. Nem toda transição é válida.
- Renovação paga → continua `active`. Falha no cartão → `past_due`. Cancelamento → no fim do período.
- Com o plano `past_due`, o aluno pode reservar? E as reservas futuras dele? **(em aberto)**
- Troca de plano (ex.: mensal → anual). **(em aberto)**

### 3.7 Atendimento pelo WhatsApp

1. O aluno manda uma mensagem. A Evolution API entrega ao sistema por webhook.
2. O sistema **salva a mensagem antes de qualquer coisa** e coloca na fila.
3. Um worker chama a IA, que devolve `{ category, priority, confidence, suggestedReply }`.
4. A resposta da IA é validada. Inválida → nova tentativa com limite; depois disso, vai para revisão sem sugestão.
5. Decisão:
   - categoria de baixo risco **e** confiança acima do limite → resposta automática (só depois da avaliação);
   - qualquer outro caso → o atendente revisa;
   - pedido de falar com uma pessoa ou insatisfação → a conversa passa para o atendente.
6. A resposta é enviada pelo WhatsApp **uma única vez**, mesmo que o envio seja tentado de novo.
7. Também existe um **canal HTTP** com o mesmo fluxo, para testes e avaliação.

Categorias: cobrança, reserva/cancelamento de aula, cancelamento ou congelamento do plano, acesso/login,
dúvida geral. A IA **nunca executa ações** (reservar, cancelar, reembolsar).

### 3.8 Reconciliação

De tempos em tempos, o sistema compara o que tem salvo com o que o Stripe diz, para corrigir webhooks perdidos
(ex.: o aluno pagou e o plano não foi liberado).

## 4. Regras de negócio

**Definidas**
- O sistema atende vários estúdios; modalidades, salas, instrutores e turmas pertencem a um estúdio.
- Cada modalidade exige reserva ou é de acesso livre. Só as modalidades com reserva têm turmas e vagas.
- Cada estúdio é uma empresa independente. O aluno pertence a um único estúdio e o plano vale só nele.
- O administrador de um estúdio é o proprietário e não acessa dados de outro estúdio.
- Novos estúdios são criados pela equipe do sistema.
- Perfis em dois níveis: a equipe do sistema (admin), separada dos usuários dos estúdios, e os papéis dentro de cada
  estúdio (proprietário, recepção, instrutor, aluno). Uma pessoa pode ter mais de um papel no mesmo estúdio.
- Um cadastro por estúdio: quem treina em dois estúdios tem duas contas. O e-mail é único por estúdio, então login
  e recuperação de senha identificam o estúdio.
- A área de acesso livre exige plano ativo ou diária paga.
- A diária é um pagamento avulso e vale por um dia, para a área livre e para as aulas com reserva.
- Uma aula tem modalidade, horário, instrutor, sala e número máximo de vagas.
- Nunca confirmar mais reservas do que vagas.
- Cancelar duas vezes tem efeito uma vez só: a vaga é liberada e a lista de espera avança uma vez.
- O status da assinatura é consultado no banco a cada reserva (não fica guardado na sessão).
- Categorias de alto risco (cobrança, reembolso, cancelamento do plano) vão sempre para o atendente.

**Em aberto** (afetam a modelagem)
- Prazo de cancelamento e o que acontece fora dele (a diária é estornada? conta como falta?).
- Lista de espera automática ou por oferta.
- O plano ilimitado tem limite de aulas por dia?
- Com diária, o aluno paga antes ou depois de garantir a vaga?
- O que acontece com reservas futuras quando o plano é cancelado ou fica atrasado.
- Falta sem aviso tem consequência?
- Turmas cadastradas por seed ou por painel de administração.
- Um instrutor que também é aluno pode reservar vaga numa aula que ele mesmo dá?
- Como os dados de um estúdio ficam separados dos outros (decisão de arquitetura, ADR).

## 5. Integrações externas

| Serviço | O que o sistema envia | O que o sistema recebe | Interface |
|---|---|---|---|
| Stripe (modo teste) | Criação de checkout e assinatura | Webhooks de pagamento, renovação, falha, cancelamento | gateway de pagamento |
| Evolution API | Respostas ao aluno | Webhooks com as mensagens recebidas | `MessageChannel` |
| Claude API | Mensagem do aluno + contexto | Classificação e resposta sugerida | `TicketClassifier` |

Todas ficam atrás de interfaces (ports and adapters) e podem falhar, demorar ou responder duas vezes.

## 6. Arquitetura

Em definição (ADR 0001). Decisões registradas até agora no [DECISOES.md](DECISOES.md):
Node.js, TypeScript, Express, PostgreSQL, Prisma, Redis + BullMQ, monólito modular + worker, sessão com cookie no Redis.

O que a arquitetura precisa mostrar:
- quem recebe cada entrada (navegador, webhook do Stripe, webhook da Evolution, canal HTTP, tarefas agendadas);
- o que roda na requisição e o que roda no worker;
- qual módulo é dono de cada dado;
- o que acontece quando cada serviço externo falha.

## 7. Onde estão os problemas técnicos

| Fluxo | Problema |
|---|---|
| Reservar (3.3) | Concorrência pela última vaga |
| Pagar diária (3.2) | Vaga disputada enquanto o pagamento é confirmado; transação pagamento + vaga; clique duplo |
| Cancelar (3.4) | Idempotência (cancelar duas vezes) |
| Lista de espera (3.5) | Concorrência na promoção; fila para avisar o aluno |
| Assinar (3.1) | Idempotência no clique duplo; webhooks duplicados, fora de ordem, perdidos |
| Ciclo da assinatura (3.6) | Máquina de estados |
| Reconciliação (3.8) | Webhook perdido |
| Atendimento (3.7) | Fila, retentativas, DLQ, circuit breaker, validação da IA, prompt injection, envio idempotente, avaliação |
