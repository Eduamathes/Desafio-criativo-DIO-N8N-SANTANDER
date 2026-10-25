# Desafio-criativo-DIO-N8N-SANTANDER
Resposta ao desafio criativo: prompt para geração de automação com IA


# Automação de Encomendas de Salgados no N8N

Automação que recebe, valida e responde automaticamente pedidos de grandes encomendas de salgados para festas, feita como resposta ao Desafio Criativo de Prompt da DIO.

## Contexto do desafio

O desafio pede a construção de um prompt completo, em três etapas (processo, contexto e regras, prompt final), para uma automação no N8N. O cenário escolhido foi uma pastelaria que recebe encomendas grandes para festas, com regras de preço, prazo e quantidade mínima.

## O prompt

```text
Atue como um especialista em N8N.

Crie uma automação para receber, validar e responder automaticamente pedidos de grandes encomendas de salgados para festas de uma pastelaria.

Público:
- Cliente final, que faz a encomenda.
- Dono ou atendente da pastelaria, que recebe a notificação dos pedidos confirmados.

Ferramentas envolvidas:
- Formulário nativo do N8N (nó Form Trigger).
- Gmail, para responder ao cliente.
- Google Sheets, para registrar os pedidos.
- Telegram, para notificar a pastelaria sobre pedidos confirmados.

Fluxo:
1. O cliente preenche o formulário com os itens, as quantidades, a data de entrega e os dados de contato (nome, e-mail e WhatsApp com DDD).
2. O workflow valida os dados recebidos.
3. Se os dados forem inválidos, envia um e-mail ao cliente pedindo a correção e encerra o fluxo.
4. Se forem válidos, calcula o valor total com base na tabela de preços e na taxa de entrega.
5. Envia a cotação ao cliente por e-mail automaticamente.
6. Registra o pedido na planilha.
7. Notifica a pastelaria no Telegram com o resumo do pedido confirmado.

Regras:
- A data de entrega deve ter no mínimo 48 horas de antecedência e não pode estar no passado.
- Quantidade mínima de 30 unidades por item.
- E-mail válido e WhatsApp com DDD.
- Preço total = soma de (quantidade x preço unitário de cada produto) + taxa de entrega fixa.
- Pedidos acima de 300 unidades são aprovados automaticamente, mas marcados como alta prioridade na notificação da pastelaria.
- Pedidos com campos obrigatórios vazios ou com itens fora do catálogo são recusados na validação, com e-mail pedindo correção.

Explique quais nós do N8N devem ser utilizados e a lógica de funcionamento do workflow.
```

## Decisões de arquitetura

### Formulário em vez de chatbot

O caso não tem conversa de ida e volta. O cliente preenche tudo de uma vez e recebe uma resposta. Isso é formulário com envio automático de mensagem, não chatbot. O N8N já tem o nó Form Trigger, que cria a página do formulário dentro do próprio fluxo, sem depender de ferramenta externa.

### Cálculo determinístico em vez de agente de IA

Preço precisa ser exato, repetível e auditável. Um agente de IA pode errar a conta, inventar valor ou responder diferente para o mesmo pedido. Validação e cálculo são lógica determinística: nó IF para validar e nó de código para calcular. IA só entraria se o pedido viesse em texto livre, o que não é o caso.

### Aprovação automática com marcação de prioridade

A aprovação é automática, então a validação é a única porteira do fluxo. Como isso ignora a capacidade real de produção da pastelaria, pedidos acima de 300 unidades são sinalizados como alta prioridade na notificação, para o dono conseguir se organizar antes de a demanda virar problema.

### Gmail e Telegram

Gmail responde o cliente, porque é gratuito e simples. Telegram avisa a pastelaria, porque é gratuito, instantâneo e não exige aprovação de API. WhatsApp exigiria a API Business da Meta, com processo de aprovação, então ficou como evolução futura.

## Fluxo do workflow

```text
Formulário (Form Trigger)
          |
    Validação (IF)
      |         |
   inválido   válido
      |         |
  E-mail de   Cálculo do valor (Code)
  correção        |
      |       E-mail de cotação (Gmail)
     Fim          |
              Registrar (Google Sheets)
                  |
              Notificar pastelaria (Telegram)
```

## Nós do N8N utilizados

- Form Trigger: recebe os dados do formulário.
- IF: valida data, quantidade, e-mail, WhatsApp e itens do catálogo.
- Code: calcula o valor total com base na tabela de preços.
- Gmail: envia o e-mail de correção ou a cotação.
- Google Sheets: registra os pedidos confirmados.
- Telegram: notifica a pastelaria.

## Evoluções possíveis

- Regra de capacidade por data, limitando o total de unidades por dia, o que resolve o furo da aprovação automática sem precisar de aprovação humana.
- Migração do canal de aviso para WhatsApp Business API.
- Painel de acompanhamento dos pedidos em uma planilha com abas e indicadores.
```

