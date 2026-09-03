# Visão de Produto — Restaurante Universitário (D6)

## Problema

Nosso cenário problema parte de um restaurante universitário onde o fluxo de refeições depende de processos manuais e desconectados: o cardápio do dia não tem um lugar único e confiável de consulta, o pagamento é feito à parte (dinheiro, cartão, ficha física) sem relação direta com o pedido, e a cozinha não tem visibilidade estruturada da fila de preparo. Isso gera filas, erros de cobrança e falta de rastreabilidade sobre o que foi pedido, pago e preparado.

## Público-alvo

- **Alunos, docentes e servidores** que fazem refeições no restaurante universitário e precisam consultar o cardápio, fazer pedidos e controlar seu saldo de forma simples.
- **Equipe do restaurante (cozinha/balcão)** que precisa saber, em tempo real, o que foi pedido e pago, para organizar o preparo.
- **Gestão do RU** que precisa cadastrar o cardápio diário e acompanhar a operação.

## Proposta de valor

Uma plataforma que integra três frentes do dia a dia do restaurante — **cardápio/pedido**, **pagamento (carteira digital)** e **preparo** — permitindo que o usuário monte sua comanda a partir do cardápio fixo do dia, pague com saldo pré-carregado na carteira, e acompanhe o status do preparo até a retirada. A equipe da cozinha, por sua vez, enxerga a fila de pedidos já pagos, sem depender de conferência manual.

## Contextos delimitados (Bounded Contexts)

| Contexto | Responsabilidade |
|---|---|
| **Pedido** | Cardápio fixo diário (publicado no site), cadastro de refeições, montagem da comanda |
| **Pagamento** | Carteira digital do usuário: recarga de saldo e cobrança pelo pedido |
| **Preparo** | Fila de produção do restaurante, atualização de status (recebido → em preparo → pronto → retirado) |

Cada contexto será dono do seu próprio banco de dados (Database per Service), conforme exigido pelo checklist arquitetural da disciplina.

## A transação que atravessa contextos (candidata a SAGA)

### Fluxo: Fazer Pedido

1. **Pedido** — usuário monta a comanda a partir do cardápio do dia e solicita o pedido.
2. **Pagamento** — o serviço de Pagamento tenta **cobrar** o valor da comanda na carteira do usuário.
   - Se o saldo for **insuficiente**, o pagamento é recusado.
3. **Preparo** — com o pagamento confirmado, o pedido é enviado para a fila da cozinha (status inicial: *recebido*).

**Porque essa transação pode falhar no meio:**

Se a cobrança na carteira falhar (saldo insuficiente, ou a carteira estar temporariamente indisponível), a comanda registrada no contexto de Pedido precisa ser **cancelada/compensada** — ela não pode ficar "pendurada" como se tivesse sido feita, já que o serviço de Pedido não tem como fazer `ROLLBACK` direto no banco do serviço de Pagamento (e vice-versa).

Esse é o ponto que vai virar a **SAGA** do grupo (Encontro 5): o passo de compensação é **cancelar o pedido / liberar a comanda** caso a cobrança falhe, análogo ao exemplo de "liberar a vaga" quando o cartão é recusado.

## Fora de escopo (por enquanto)

- Múltiplos restaurantes / unidades (foco em um único RU).
- Pagamento externo (cartão de crédito real, PIX) — a carteira interna com **recarga** é a única forma de pagamento na v1.
- Reservas antecipadas de refeição (pedido é feito no momento).

## Métricas de sucesso do projeto (didáticas)

- Fluxo ponta a ponta funcionando: cardápio → pedido → cobrança → preparo.
- SAGA com compensação testável (cobrança falha → pedido cancelado automaticamente).
- Sistema **degrada, não morre** quando o serviço de Pagamento ou Preparo cai (teste do caos do Encontro 9).