# RU Digital — Restaurante Universitário

Projeto da disciplina **GCC267 — Projeto Integrador I** (DCC/UFLA, 2026/2, Turma 14A).
Domínio: **D6 — Restaurante Universitário**.

## O que é

Sistema que integra três frentes da operação de um restaurante universitário:

- **Pedido** — cardápio fixo diário publicado no site, cadastro de refeições e montagem da comanda.
- **Pagamento** — carteira digital do usuário, com recarga de saldo e cobrança pelo pedido.
- **Preparo** — acompanhamento do status do pedido na cozinha (recebido → em preparo → pronto → retirado).

Veja a visão completa do produto em [`docs/visao-produto.md`](docs/visao-produto.md), incluindo a transação que atravessa contextos (o fluxo de *Fazer Pedido*, que será implementado como uma SAGA no decorrer do semestre).

## Arquitetura

O projeto evolui em três sprints, saindo de um monólito modular até um sistema com serviços independentes se comunicando por eventos:

1. **Sprint 1 — Fundação:** monólito modular com os contextos Pedido, Pagamento e Preparo bem separados, fluxo ponta a ponta funcionando, testes e CI verde.
2. **Sprint 2 — Distribuição:** extração de pelo menos 2 serviços, broker de eventos, padrão Outbox, consumidor idempotente, SAGA com compensação e um read model (CQRS).
3. **Sprint 3 — Robustez:** circuit breaker, rastreio ponta a ponta (correlation-id), deploy e roteiro de teste do caos.

## Contextos e responsabilidades

| Serviço | Responsabilidade | Banco |
|---|---|---|
| `pedido` | Cardápio do dia, cadastro de refeições, comanda | próprio |
| `pagamento` | Carteira do usuário (recarga e cobrança) | próprio |
| `preparo` | Fila de produção e status do pedido na cozinha | próprio |

## Como rodar

```bash
docker compose up
```

> Instruções detalhadas de subida (variáveis de ambiente, portas, seed de dados) serão adicionadas conforme os serviços forem implementados nas próximas quinzenas.

## Documentação

- [`docs/visao-produto.md`](docs/visao-produto.md) — problema, público-alvo e transação distribuída
- [`docs/equipe.md`](docs/equipe.md) — integrantes e organização do time
- [`docs/adr/`](docs/adr/) — registro das decisões arquiteturais (ADRs)

## Equipe

_A preencher em `docs/equipe.md`._

## Disciplina

- **Professor:** Dr. Rafael S. Durelli
