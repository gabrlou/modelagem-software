# Especificação de Requisitos — Site de Loja de Eletrônicos (E-commerce)

> **Documento:** APS 2 — Modelagem de Software
> **Fonte:** descrição inicial (`instrucao-inicial.md`) + instruções do professor
> **Finalidade:** servir de base para a modelagem UML (Diagrama de Casos de Uso e Diagrama de Componentes)
> **Versão:** 1.0 (rascunho para validação do grupo)

---

## Sumário

1. [Visão Geral do Sistema](#1-visão-geral-do-sistema)
2. [Atores](#2-atores)
3. [Requisitos Funcionais](#3-requisitos-funcionais)
4. [Requisitos Não Funcionais](#4-requisitos-não-funcionais)
5. [Regras de Negócio](#5-regras-de-negócio)
6. [Casos de Uso](#6-casos-de-uso)
7. [Especificação Textual dos Casos de Uso](#7-especificação-textual-dos-casos-de-uso)
8. [Arquitetura e Componentes](#8-arquitetura-e-componentes)
9. [Premissas e Decisões de Projeto](#9-premissas-e-decisões-de-projeto)
10. [Apêndice — Rastreabilidade](#apêndice--rastreabilidade)

---

## 1. Visão Geral do Sistema

### 1.1 Objetivo

Disponibilizar um **site de loja de eletrônicos** em que o cliente navega por um catálogo (com filtros por categoria, marca e preço), monta um carrinho e finaliza o pedido, com **pagamento via gateway** e **cálculo de frete** por serviço externo. Um **módulo administrativo** permite cadastrar produtos, gerenciar o estoque e acompanhar o status dos pedidos.

### 1.2 Problema que resolve

- Venda de eletrônicos limitada ao atendimento presencial ou por canais informais;
- Controle manual de estoque, gerando venda de itens indisponíveis;
- Dificuldade do cliente em comparar e encontrar produtos (muitas marcas, faixas de preço e categorias);
- Falta de acompanhamento do pedido por parte do cliente e da loja;
- Pagamento e frete tratados fora do sistema, sem rastreabilidade.

### 1.3 Benefícios esperados

| Benefício | Descrição |
|---|---|
| Venda 24/7 | O cliente compra a qualquer hora, sem depender de atendente |
| Descoberta de produtos | Filtros e busca reduzem o tempo até o produto desejado |
| Estoque consistente | Reserva e baixa automáticas evitam venda sem estoque |
| Pagamento seguro | Dados de cartão tratados pelo gateway, não pela loja |
| Frete transparente | Valor e prazo exibidos antes da confirmação |
| Rastreabilidade | Status do pedido visível ao cliente e à loja |

### 1.4 Escopo

**Dentro do escopo:**

- Catálogo com navegação, filtros (categoria, marca, faixa de preço), busca e detalhes do produto;
- Cadastro e autenticação de clientes; autenticação do administrador;
- Carrinho de compras;
- Cálculo de frete (serviço externo);
- Finalização do pedido e pagamento (gateway externo), incluindo estorno em cancelamentos;
- Acompanhamento e cancelamento de pedido pelo cliente;
- Módulo administrativo: produtos, categorias e marcas, estoque e status dos pedidos;
- Notificação por e-mail das mudanças de status do pedido.

**Fora do escopo (nesta versão):**

- Cupons, promoções, programa de fidelidade e avaliações de produtos;
- Troca, devolução e pós-venda;
- Emissão de nota fiscal e integração fiscal/contábil;
- Marketplace (múltiplos vendedores) e múltiplas lojas;
- Aplicativo nativo (considera-se **web responsivo**, P-01);
- Recomendação de produtos e *chat* de atendimento.

---

## 2. Atores

### 2.1 Atores principais

#### Visitante
- **Descrição:** qualquer pessoa que acessa o site sem estar autenticada.
- **Pode:** navegar pelo catálogo, filtrar, buscar, ver detalhes de produto, montar carrinho, calcular frete e cadastrar-se.
- **Não pode:** finalizar pedido nem acessar pedidos.

#### Cliente (especialização de Visitante)
- **Descrição:** visitante cadastrado e autenticado. **Herda** tudo o que o Visitante faz (**generalização de ator**).
- **Responsabilidades:** manter seus dados e endereços, finalizar pedidos, pagar, acompanhar e cancelar pedidos.
- **Permissões:** lê e altera **somente** seus dados e seus pedidos.

#### Administrador
- **Descrição:** funcionário da loja que opera o módulo administrativo.
- **Responsabilidades:** cadastrar e manter produtos, categorias e marcas; controlar estoque; acompanhar e atualizar o status dos pedidos.
- **Permissões:** acesso total ao módulo administrativo; **não** compra em nome de clientes nem acessa dados de pagamento sensíveis (que não são armazenados, RNF-05).

### 2.2 Atores secundários (sistemas externos)

| Ator | Papel | Interação |
|---|---|---|
| **Gateway de Pagamento** | Autoriza, captura e estorna pagamentos (cartão, Pix, boleto) | Recebe solicitação de cobrança; devolve resultado (aprovado/recusado/pendente) e confirmações assíncronas |
| **Serviço de Frete** | Calcula valor e prazo de entrega por CEP e dimensões/peso | Recebe CEP de origem/destino e itens; devolve opções de frete |
| **Serviço de E-mail** | Entrega e-mails transacionais | Recebe a mensagem montada pelo sistema; devolve resultado do envio |

> Atores secundários são **sistemas externos**: ficam **fora** da fronteira do sistema no Diagrama de Casos de Uso e aparecem como **componentes externos** no Diagrama de Componentes.

### 2.3 Matriz ator × ação

| Ação | Visitante | Cliente | Administrador |
|---|---|---|---|
| Navegar / filtrar / buscar / ver detalhes | sim | sim | — |
| Montar carrinho e simular frete | sim | sim | — |
| Cadastrar-se | sim | — | — |
| Finalizar pedido e pagar | não | sim | não |
| Acompanhar / cancelar o próprio pedido | não | sim | — |
| Gerenciar produtos, categorias e marcas | não | não | sim |
| Gerenciar estoque | não | não | sim |
| Consultar e atualizar status de pedidos; cancelar pedido | não | não | sim |

---

## 3. Requisitos Funcionais

### 3.1 Catálogo

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-01 | Listar produtos | Exibir produtos ativos de forma paginada, com imagem, nome, marca, preço e disponibilidade. | RN-01, RN-02 |
| RF-02 | Filtrar produtos | Filtrar por **categoria**, **marca** e **faixa de preço**, de forma combinável. | RN-03 |
| RF-03 | Buscar produtos | Buscar por texto (nome, marca, descrição). | RN-01 |
| RF-04 | Ordenar resultados | Ordenar por preço (crescente/decrescente), nome ou relevância. | — |
| RF-05 | Exibir detalhes | Mostrar descrição, especificações técnicas, imagens, preço e disponibilidade. | RN-01, RN-02 |

### 3.2 Conta e autenticação

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-06 | Cadastrar cliente | Cadastro com nome, CPF, e-mail, telefone e senha. | RN-04, RN-05, RN-06 |
| RF-07 | Autenticar usuário | Login por e-mail + senha para Cliente e Administrador, distinguindo perfis. | RN-07, RN-08 |
| RF-08 | Recuperar senha | Redefinição por link enviado ao e-mail cadastrado. | RN-09 |
| RF-09 | Gerenciar endereços | O cliente mantém ao menos um endereço de entrega (CEP, logradouro, número, complemento, cidade, UF). | RN-10 |

### 3.3 Carrinho

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-10 | Adicionar ao carrinho | Incluir produto com quantidade, validando disponibilidade. | RN-11, RN-12 |
| RF-11 | Alterar / remover item | Alterar quantidade ou remover item do carrinho. | RN-11, RN-12 |
| RF-12 | Visualizar carrinho | Listar itens, preços unitários, subtotal e total parcial. | RN-13 |
| RF-13 | Manter carrinho | O carrinho do visitante é mantido na sessão e **incorporado** ao carrinho do cliente ao autenticar. | RN-14 |

### 3.4 Frete

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-14 | Calcular frete | Calcular opções de frete (valor e prazo) por CEP de destino, via Serviço de Frete, no carrinho (simulação) e no checkout. | RN-15, RN-16 |

### 3.5 Pedido e pagamento

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-15 | Finalizar pedido | Fluxo de checkout: autenticar → endereço → frete → forma de pagamento → revisão → confirmação. | RN-10, RN-17, RN-18 |
| RF-16 | Registrar pedido | Criar o pedido com número único, itens (preço no momento da compra), frete e total. | RN-13, RN-17, RN-19 |
| RF-17 | Processar pagamento | Enviar a cobrança ao Gateway de Pagamento e tratar o retorno (aprovado, recusado, pendente). | RN-20, RN-21 |
| RF-18 | Receber confirmação de pagamento | Atualizar o pedido ao receber a confirmação assíncrona do gateway (Pix e boleto). | RN-21, RN-22 |
| RF-19 | Acompanhar pedido | O cliente lista seus pedidos e vê status e histórico de cada um. | RN-23 |
| RF-20 | Cancelar pedido | O cliente cancela pedido ainda não enviado; se pago, dispara **estorno**. | RN-24, RN-25 |
| RF-21 | Estornar pagamento | Solicitar ao gateway o estorno total de pedido pago e cancelado. | RN-25 |
| RF-22 | Notificar por e-mail | Enviar e-mail ao cliente nos eventos: pedido recebido, pagamento confirmado, enviado, entregue, cancelado. | RN-26 |

### 3.6 Estoque

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-23 | Verificar disponibilidade | Consultar a quantidade disponível de um produto (carrinho e checkout). | RN-11, RN-12 |
| RF-24 | Reservar e baixar estoque | Reservar ao criar o pedido; baixar ao confirmar o pagamento; devolver ao cancelar ou expirar. | RN-12, RN-27, RN-28 |
| RF-25 | Gerenciar estoque | O administrador ajusta a quantidade (entrada/saída/ajuste) com motivo registrado. | RN-29, RN-30 |
| RF-26 | Alertar estoque baixo | Sinalizar ao administrador produtos abaixo do estoque mínimo. | RN-31 |

### 3.7 Módulo administrativo

| ID | Requisito | Descrição | RN |
|---|---|---|---|
| RF-27 | Cadastrar produto | Cadastrar produto com SKU, nome, descrição, especificações, imagens, categoria, marca, preço, peso/dimensões e estoque inicial. | RN-01, RN-32, RN-33 |
| RF-28 | Editar / inativar produto | Alterar dados ou inativar o produto (sem exclusão física se houver pedidos). | RN-01, RN-34 |
| RF-29 | Gerenciar categorias e marcas | Criar, editar e inativar categorias e marcas usadas no catálogo e nos filtros. | RN-35 |
| RF-30 | Consultar pedidos | Listar e filtrar pedidos por status, data e cliente; ver detalhes. | RN-23 |
| RF-31 | Atualizar status do pedido | Avançar o status (em separação, enviado com código de rastreio, entregue), respeitando a tabela de transições. | RN-22, RN-36 |
| RF-32 | Cancelar pedido (admin) | O administrador cancela um pedido com motivo obrigatório; se pago, dispara estorno. | RN-24, RN-25, RN-36 |

---

## 4. Requisitos Não Funcionais

### 4.1 Segurança

| ID | Requisito |
|---|---|
| RNF-01 | Senhas armazenadas com hash (bcrypt/argon2). |
| RNF-02 | Tráfego exclusivamente via HTTPS/TLS 1.2+. |
| RNF-03 | Controle de acesso por perfil (RBAC): o módulo administrativo só responde a Administrador. |
| RNF-04 | Bloqueio temporário após 5 falhas de login consecutivas. |
| RNF-05 | A loja **não armazena** número completo de cartão nem CVV; o pagamento usa tokenização/redirecionamento do gateway (escopo PCI-DSS reduzido). |
| RNF-06 | Proteção contra ataques web comuns (OWASP Top 10: injeção, XSS, CSRF). |

### 4.2 Usabilidade

| ID | Requisito |
|---|---|
| RNF-07 | Interface responsiva (mobile-first), de 320 px a 1920 px. |
| RNF-08 | Checkout em no máximo 4 etapas (identificação/endereço → frete → pagamento → revisão). |
| RNF-09 | Mensagens de erro claras e junto ao campo; acessibilidade WCAG AA. |

### 4.3 Desempenho

| ID | Requisito |
|---|---|
| RNF-10 | Listagem e busca com resposta ≤ 500 ms (P95) em até 200 usuários simultâneos. |
| RNF-11 | Carregamento de página ≤ 3 s (P90) em banda larga. |
| RNF-12 | Chamadas a serviços externos (frete, pagamento, e-mail) com *timeout* e tratamento de falha; e-mail enviado de forma **assíncrona**. |

### 4.4 Disponibilidade

| ID | Requisito |
|---|---|
| RNF-13 | Disponibilidade de 99,5% mensal. |
| RNF-14 | Backup diário do banco com retenção de 30 dias. |
| RNF-15 | Indisponibilidade de serviço externo degrada apenas a função correspondente (ex.: frete fora do ar não derruba o catálogo). |

### 4.5 Escalabilidade e manutenção

| ID | Requisito |
|---|---|
| RNF-16 | Backend *stateless*, permitindo réplicas horizontais. |
| RNF-17 | Cache de catálogo e resultados de filtros (TTL curto). |
| RNF-18 | Módulos com baixo acoplamento e comunicação por **interfaces bem definidas**, permitindo trocar o gateway ou o serviço de frete sem alterar o núcleo do sistema. |

### 4.6 Privacidade e auditoria

| ID | Requisito |
|---|---|
| RNF-19 | Dados pessoais tratados conforme a LGPD (minimização; consentimento no cadastro; direito de exclusão respeitando retenção legal de pedidos). |
| RNF-20 | Log de auditoria das ações administrativas (produto, estoque, status/cancelamento de pedido): quem, o quê, quando, de onde. |
| RNF-21 | Dados pessoais e de pagamento não aparecem em URLs nem em logs. |

---

## 5. Regras de Negócio

### 5.1 Catálogo e produtos

| ID | Regra |
|---|---|
| **RN-01** | Apenas produtos `ATIVO` (com categoria e marca ativas) aparecem no catálogo, na busca e nos detalhes. |
| **RN-02** | Produto sem estoque disponível continua visível, marcado como **indisponível**, e não pode ser adicionado ao carrinho. |
| **RN-03** | Filtros combinados usam **E** entre critérios distintos (categoria E marca E faixa de preço) e **OU** entre valores do mesmo critério (marca A OU marca B). |
| **RN-32** | O SKU é **único**; preço > 0; todo produto tem exatamente 1 categoria e 1 marca. |
| **RN-33** | Cadastro exige ao menos 1 imagem, descrição, peso e dimensões (necessários ao frete). |
| **RN-34** | Produto que já consta em pedido **não é excluído**, apenas inativado; o pedido mantém o preço e os dados da época da compra. |
| **RN-35** | Categoria ou marca só pode ser inativada/removida se não houver produto ativo associado. |

### 5.2 Conta e acesso

| ID | Regra |
|---|---|
| **RN-04** | CPF válido (módulo 11) e **único** entre clientes. |
| **RN-05** | E-mail válido e **único** no sistema. |
| **RN-06** | Senha com no mínimo 8 caracteres, ao menos 1 letra e 1 número. |
| **RN-07** | Perfil (CLIENTE ou ADMINISTRADOR) é derivado do cadastro; perfis são exclusivos (P-08). |
| **RN-08** | Após 5 falhas consecutivas de login, a conta fica bloqueada por 15 minutos. |
| **RN-09** | Link de redefinição de senha expira em 30 minutos e é de uso único. |
| **RN-10** | O pedido exige endereço de entrega completo e com CEP válido. |

### 5.3 Carrinho

| ID | Regra |
|---|---|
| **RN-11** | A quantidade de cada item deve ser ≥ 1 e ≤ estoque disponível. |
| **RN-12** | Estoque disponível = quantidade em estoque − quantidade reservada por pedidos em andamento. |
| **RN-13** | Total do pedido = soma(preço unitário × quantidade) + frete. O preço é o vigente no momento da **criação do pedido** e fica congelado nele. |
| **RN-14** | Ao autenticar, itens do carrinho da sessão são somados ao carrinho do cliente; em caso de mesmo produto, soma-se a quantidade limitada ao estoque. |

### 5.4 Frete

| ID | Regra |
|---|---|
| **RN-15** | O frete é calculado a partir do CEP de destino e do peso/dimensões dos itens; o CEP de origem é fixo (loja). |
| **RN-16** | Se o Serviço de Frete estiver indisponível, o checkout **não prossegue** (sem frete não há total), mas o carrinho continua utilizável. |

### 5.5 Pedido e pagamento

| ID | Regra |
|---|---|
| **RN-17** | Somente cliente autenticado finaliza pedido. |
| **RN-18** | O valor do frete escolhido é confirmado no checkout; mudança de itens ou CEP invalida o frete e exige novo cálculo. |
| **RN-19** | O pedido nasce com status `AGUARDANDO_PAGAMENTO` e recebe número único. |
| **RN-20** | Formas de pagamento aceitas: cartão de crédito, Pix e boleto (P-04). Cartão é autorizado de imediato; Pix e boleto ficam pendentes até a confirmação. |
| **RN-21** | Pagamento recusado **não cancela** o pedido de imediato: o cliente pode tentar outra forma de pagamento enquanto a reserva de estoque estiver válida. |
| **RN-22** | Pagamento aprovado muda o pedido para `PAGO`; só pedido `PAGO` pode ser separado e enviado. |
| **RN-23** | O cliente só vê seus próprios pedidos; o administrador vê todos. |
| **RN-24** | O cancelamento é permitido enquanto o pedido estiver `AGUARDANDO_PAGAMENTO`, `PAGO` ou `EM_SEPARACAO`; depois de `ENVIADO`, não. Cancelamento exige motivo. |
| **RN-25** | Pedido pago e cancelado gera **estorno total** pelo gateway; o pedido só vai a `CANCELADO` com o estorno solicitado com sucesso (ou pendente de reprocessamento, P-10). |
| **RN-26** | Falha no envio de e-mail **não** desfaz nenhuma operação; o envio é reprocessado (até 3 tentativas). |

**Tabela de transições do status do pedido** (fonte única):

| De | Evento | Para | Quem |
|---|---|---|---|
| — | finalizar pedido | `AGUARDANDO_PAGAMENTO` | Cliente |
| `AGUARDANDO_PAGAMENTO` | pagamento aprovado | `PAGO` | Gateway (via sistema) |
| `AGUARDANDO_PAGAMENTO` | prazo de pagamento expirou | `CANCELADO` | Sistema (P-09) |
| `PAGO` | iniciar separação | `EM_SEPARACAO` | Administrador |
| `EM_SEPARACAO` | despachar (com rastreio) | `ENVIADO` | Administrador |
| `ENVIADO` | confirmar entrega | `ENTREGUE` | Administrador |
| `AGUARDANDO_PAGAMENTO`, `PAGO`, `EM_SEPARACAO` | cancelar | `CANCELADO` | Cliente, Administrador |

`ENTREGUE` e `CANCELADO` são estados finais; transições inválidas são rejeitadas.

### 5.6 Estoque

| ID | Regra |
|---|---|
| **RN-27** | O estoque é **reservado** ao criar o pedido e **baixado** quando o pagamento é aprovado. |
| **RN-28** | A reserva é devolvida ao estoque se o pedido for cancelado ou expirar (P-09). |
| **RN-29** | O estoque nunca fica negativo; ajuste que resultaria em valor negativo é rejeitado. |
| **RN-30** | Todo ajuste manual registra produto, quantidade, motivo, autor e data. |
| **RN-31** | Cada produto tem estoque mínimo; abaixo dele, gera alerta ao administrador. |

### 5.7 Administração

| ID | Regra |
|---|---|
| **RN-36** | Mudanças de status e cancelamentos feitos pelo administrador registram autor e data em auditoria (RNF-20). Envio exige código de rastreio. |

### 5.8 Exceções de domínio (síntese)

- `ProdutoIndisponivelException` — estoque insuficiente na adição ao carrinho ou no checkout (corrida entre clientes);
- `FreteIndisponivelException` — Serviço de Frete fora do ar ou CEP inválido;
- `PagamentoRecusadoException` — recusa do gateway;
- `TransicaoInvalidaException` — mudança de status não permitida;
- `NaoAutorizadoException` — acesso a recurso de outro perfil/usuário.

---

## 6. Casos de Uso

### 6.1 Catálogo de casos de uso

| ID | Caso de uso | Ator principal | Atores secundários | RF |
|---|---|---|---|---|
| UC-01 | Navegar pelo Catálogo | Visitante | — | RF-01, RF-04 |
| UC-02 | Filtrar Produtos | Visitante | — | RF-02 |
| UC-03 | Buscar Produtos | Visitante | — | RF-03 |
| UC-04 | Visualizar Detalhes do Produto | Visitante | — | RF-05 |
| UC-05 | Cadastrar-se | Visitante | — | RF-06 |
| UC-06 | Autenticar Usuário | Cliente, Administrador | — | RF-07 |
| UC-07 | Recuperar Senha | Cliente, Administrador | Serviço de E-mail | RF-08 |
| UC-08 | Gerenciar Carrinho | Visitante (Cliente) | — | RF-10…RF-13 |
| UC-09 | Verificar Disponibilidade em Estoque | (interno) | — | RF-23 |
| UC-10 | Calcular Frete | Visitante (Cliente) | Serviço de Frete | RF-14 |
| UC-11 | Finalizar Pedido | Cliente | — | RF-09, RF-15, RF-16, RF-24 |
| UC-12 | Processar Pagamento | (interno) | Gateway de Pagamento | RF-17, RF-18 |
| UC-13 | Acompanhar Pedido | Cliente | — | RF-19 |
| UC-14 | Cancelar Pedido | Cliente | — | RF-20 |
| UC-15 | Estornar Pagamento | (interno) | Gateway de Pagamento | RF-21 |
| UC-16 | Notificar Cliente por E-mail | (interno) | Serviço de E-mail | RF-22 |
| UC-17 | Gerenciar Produtos | Administrador | — | RF-27, RF-28 |
| UC-18 | Gerenciar Categorias e Marcas | Administrador | — | RF-29 |
| UC-19 | Gerenciar Estoque | Administrador | — | RF-25 |
| UC-20 | Alertar Estoque Baixo | Administrador | — | RF-26 |
| UC-21 | Consultar Pedidos | Administrador | — | RF-30 |
| UC-22 | Atualizar Status do Pedido | Administrador | — | RF-31 |
| UC-23 | Cancelar Pedido (Administrativo) | Administrador | — | RF-32 |

> *(interno)* = caso de uso incluído por outros; não é acionado diretamente por ator humano, mas é associado ao ator secundário.

### 6.2 Relacionamentos propostos para o diagrama

**Generalização de atores**

- `Cliente` ──▷ `Visitante` (o Cliente herda todos os casos de uso do Visitante).

**Generalização de casos de uso**

- `Pagar com Cartão`, `Pagar com Pix`, `Pagar com Boleto` ──▷ `Processar Pagamento` (UC-12). Evidenciam as variações da mesma funcionalidade (RN-20).

**«include»** (passos obrigatórios/reutilizáveis)

| Base | Incluído | Motivo |
|---|---|---|
| UC-11 Finalizar Pedido | UC-06 Autenticar Usuário | RN-17: exige cliente autenticado |
| UC-11 Finalizar Pedido | UC-10 Calcular Frete | RN-18: não há total sem frete |
| UC-11 Finalizar Pedido | UC-12 Processar Pagamento | O pedido só avança com o pagamento |
| UC-11 Finalizar Pedido | UC-09 Verificar Disponibilidade | RN-27: reserva só com estoque |
| UC-08 Gerenciar Carrinho | UC-09 Verificar Disponibilidade | RN-11: limita a quantidade ao estoque |
| UC-13 Acompanhar Pedido | UC-06 Autenticar Usuário | RN-23: somente o titular |
| UC-14 Cancelar Pedido | UC-06 Autenticar Usuário | — |
| UC-17, UC-18, UC-19, UC-21, UC-22, UC-23 | UC-06 Autenticar Usuário | RNF-03: área restrita |
| UC-22 Atualizar Status / UC-14 / UC-23 / UC-11 | UC-16 Notificar Cliente por E-mail | RN-26 / RF-22 (eventos de pedido) |

> **Nota de modelagem:** o estorno (UC-15) só ocorre quando o pedido já está `PAGO`; por isso **não** é `include` (que é sempre executado), e sim **«extend»** de UC-14 e UC-23 (tabela seguinte).

**«extend»** (fluxos opcionais/alternativos)

| Extensão | Estende | Condição / ponto de extensão |
|---|---|---|
| UC-02 Filtrar Produtos | UC-01 Navegar pelo Catálogo | O visitante aplica filtros (categoria, marca, preço) |
| UC-03 Buscar Produtos | UC-01 Navegar pelo Catálogo | O visitante digita um termo de busca |
| UC-10 Calcular Frete | UC-08 Gerenciar Carrinho | O visitante simula o frete por CEP antes do checkout |
| UC-07 Recuperar Senha | UC-06 Autenticar Usuário | O usuário esqueceu a senha |
| UC-05 Cadastrar-se | UC-06 Autenticar Usuário | Usuário ainda sem cadastro opta por criar conta |
| UC-15 Estornar Pagamento | UC-14 Cancelar Pedido, UC-23 | Pedido já estava `PAGO` |
| UC-14 Cancelar Pedido | UC-13 Acompanhar Pedido | O cliente decide cancelar a partir do detalhe do pedido |
| UC-20 Alertar Estoque Baixo | UC-19 Gerenciar Estoque | Estoque após o ajuste < estoque mínimo |
| UC-22 Atualizar Status do Pedido | UC-21 Consultar Pedidos | O administrador seleciona um pedido para avançar o status |
| UC-23 Cancelar Pedido (Adm.) | UC-21 Consultar Pedidos | O administrador seleciona um pedido para cancelar |
| UC-04 Visualizar Detalhes | UC-01 Navegar pelo Catálogo | O visitante abre um produto (opcional; pode ser associação direta) |

> Para o desenho final, escolher entre simplificar (menos setas, mais legível) ou manter todos os relacionamentos. **Mínimo exigido pelo professor:** associação, generalização, ao menos um `include` e ao menos um `extend` — todos já atendidos.

### 6.3 Fronteira do sistema

Caixa de contorno: **"Sistema de E-commerce de Eletrônicos"**. Dentro: UC-01…UC-23. Fora: Visitante, Cliente, Administrador (à esquerda) e Gateway de Pagamento, Serviço de Frete, Serviço de E-mail (à direita).

---

## 7. Especificação Textual dos Casos de Uso

> Seguem o gabarito do professor: nome, atores, pré-condições, fluxo principal, fluxos alternativos/exceção, e uma descrição resumida.

### UC-01 — Navegar pelo Catálogo
- **Descrição:** permite ao visitante explorar os produtos ativos da loja.
- **Atores:** Visitante.
- **Pré-condições:** nenhuma.
- **Fluxo principal:** 1. Visitante acessa o catálogo; 2. Sistema lista produtos ativos paginados; 3. Visitante ordena e navega entre páginas; 4. Visitante seleciona um produto (UC-04).
- **Alternativos/exceção:** **A1** — catálogo vazio: sistema exibe mensagem "Nenhum produto disponível". **E1** — falha ao carregar: mensagem com opção "Tentar novamente".

### UC-02 — Filtrar Produtos *(extend UC-01)*
- **Descrição:** refina a listagem por categoria, marca e faixa de preço.
- **Atores:** Visitante.
- **Pré-condições:** listagem de produtos exibida.
- **Fluxo principal:** 1. Visitante escolhe categoria(s), marca(s) e/ou faixa de preço; 2. Sistema aplica os filtros (RN-03); 3. Sistema atualiza a listagem e o total de resultados.
- **Alternativos/exceção:** **A1** — nenhum produto atende aos filtros: sistema informa e sugere limpar filtros. **E1** — faixa inválida (mínimo > máximo): validação imediata.

### UC-03 — Buscar Produtos *(extend UC-01)*
- **Descrição:** localiza produtos por texto.
- **Atores:** Visitante.
- **Pré-condições:** nenhuma.
- **Fluxo principal:** 1. Visitante digita o termo; 2. Sistema pesquisa por nome, marca e descrição; 3. Sistema exibe os resultados (podem ser filtrados, UC-02).
- **Alternativos/exceção:** **A1** — sem resultados: mensagem e sugestão de categorias. **E1** — termo vazio: ignora e mantém a listagem.

### UC-04 — Visualizar Detalhes do Produto
- **Descrição:** exibe informações completas de um produto.
- **Atores:** Visitante.
- **Pré-condições:** produto ativo.
- **Fluxo principal:** 1. Visitante seleciona o produto; 2. Sistema exibe imagens, descrição, especificações, preço e disponibilidade; 3. Visitante pode adicionar ao carrinho (UC-08).
- **Alternativos/exceção:** **A1** — produto indisponível: botão de compra desabilitado (RN-02). **E1** — produto inativado/inexistente: mensagem e retorno ao catálogo.

### UC-05 — Cadastrar-se
- **Descrição:** cria a conta do cliente.
- **Atores:** Visitante.
- **Pré-condições:** nenhuma.
- **Fluxo principal:** 1. Visitante informa nome, CPF, e-mail, telefone e senha; 2. Aceita o termo de privacidade; 3. Sistema valida os dados (RN-04 a RN-06); 4. Sistema cria a conta e autentica o usuário.
- **Alternativos/exceção:** **E1** — CPF/e-mail já cadastrado: mensagem e opção de login/recuperação. **E2** — dados inválidos: erros junto aos campos.

### UC-06 — Autenticar Usuário
- **Descrição:** identifica Cliente ou Administrador e estabelece a sessão.
- **Atores:** Cliente, Administrador.
- **Pré-condições:** conta cadastrada e ativa.
- **Fluxo principal:** 1. Usuário informa e-mail e senha; 2. Sistema valida credenciais; 3. Sistema identifica o perfil e inicia a sessão; 4. Sistema devolve o usuário ao ponto em que estava (RF-13).
- **Alternativos/exceção:** **A1** — esqueceu a senha: UC-07. **A2** — sem cadastro: UC-05. **E1** — credenciais inválidas: mensagem genérica. **E2** — 5 falhas: conta bloqueada por 15 min (RN-08).

### UC-07 — Recuperar Senha *(extend UC-06)*
- **Descrição:** redefine a senha por e-mail.
- **Atores:** Cliente, Administrador; Serviço de E-mail.
- **Pré-condições:** e-mail cadastrado.
- **Fluxo principal:** 1. Usuário informa o e-mail; 2. Sistema gera link de uso único (30 min) e envia pelo Serviço de E-mail; 3. Usuário abre o link e define nova senha (RN-06); 4. Sistema atualiza a senha.
- **Alternativos/exceção:** **A1** — e-mail inexistente: resposta neutra (não revela cadastro). **E1** — link expirado/usado: solicita novo link.

### UC-08 — Gerenciar Carrinho
- **Descrição:** adiciona, altera e remove itens e mostra o total parcial.
- **Atores:** Visitante / Cliente.
- **Pré-condições:** produto ativo e disponível.
- **Fluxo principal:** 1. Ator adiciona produto e quantidade; 2. Sistema verifica disponibilidade (UC-09); 3. Sistema atualiza o carrinho e o subtotal; 4. Ator altera quantidade ou remove itens; 5. Ator segue para o checkout (UC-11) ou continua comprando.
- **Alternativos/exceção:** **A1** — simula frete por CEP (UC-10). **E1** — quantidade acima do estoque: limita à quantidade disponível e avisa (RN-11). **E2** — item ficou indisponível: sinaliza no carrinho.

### UC-09 — Verificar Disponibilidade em Estoque *(incluído)*
- **Descrição:** consulta se há quantidade suficiente de um produto.
- **Atores:** — (acionado por UC-08 e UC-11).
- **Pré-condições:** produto existente.
- **Fluxo principal:** 1. Sistema consulta quantidade em estoque e reservas; 2. Calcula o disponível (RN-12); 3. Retorna "disponível" ou "insuficiente".
- **Alternativos/exceção:** **E1** — estoque insuficiente: o caso de uso chamador trata a recusa (`ProdutoIndisponivelException`).

### UC-10 — Calcular Frete
- **Descrição:** obtém opções de valor e prazo de entrega para um CEP.
- **Atores:** Visitante / Cliente; **Serviço de Frete**.
- **Pré-condições:** carrinho com ao menos um item.
- **Fluxo principal:** 1. Ator informa o CEP (ou seleciona endereço salvo); 2. Sistema envia ao Serviço de Frete CEP e peso/dimensões dos itens; 3. Serviço retorna as opções; 4. Sistema exibe valor e prazo de cada opção; 5. Ator escolhe uma opção (no checkout).
- **Alternativos/exceção:** **E1** — CEP inválido: pede correção. **E2** — Serviço de Frete indisponível: mensagem; no checkout impede avançar (RN-16).

### UC-11 — Finalizar Pedido
- **Descrição:** converte o carrinho em pedido, com endereço, frete e pagamento.
- **Atores:** Cliente.
- **Pré-condições:** carrinho com itens; cliente autenticado (UC-06).
- **Fluxo principal:** 1. Cliente inicia o checkout (autentica-se se necessário); 2. Seleciona/cadastra endereço de entrega (RN-10); 3. Sistema calcula frete (UC-10) e cliente escolhe; 4. Cliente escolhe a forma de pagamento; 5. Sistema exibe revisão (itens, frete, total); 6. Cliente confirma; 7. Sistema verifica disponibilidade e **reserva estoque** (UC-09); 8. Sistema cria o pedido `AGUARDANDO_PAGAMENTO`; 9. Sistema processa o pagamento (UC-12); 10. Aprovado → pedido `PAGO`, estoque baixado, e-mail de confirmação (UC-16).
- **Alternativos/exceção:** **A1** (passo 10) — Pix/boleto: pedido segue `AGUARDANDO_PAGAMENTO` até a confirmação. **A2** — pagamento recusado: cliente tenta outra forma (RN-21). **E1** — item sem estoque no passo 7: aviso, ajuste do carrinho, nenhum pedido criado. **E2** — sessão expirada: login e retomada. **E3** — preço mudou: exibe novo total e pede nova confirmação.

### UC-12 — Processar Pagamento *(incluído)*
- **Descrição:** realiza a cobrança do pedido no Gateway de Pagamento. Especializado em **Pagar com Cartão**, **Pagar com Pix** e **Pagar com Boleto**.
- **Atores:** **Gateway de Pagamento**.
- **Pré-condições:** pedido criado com valor total e forma de pagamento escolhida.
- **Fluxo principal:** 1. Sistema envia a cobrança ao gateway (dados tokenizados, RNF-05); 2. Gateway responde aprovado/recusado/pendente; 3. Sistema registra o resultado; 4. Em aprovado, atualiza o pedido para `PAGO` (RN-22); 5. Em pendente, aguarda a confirmação assíncrona (RF-18).
- **Alternativos/exceção:** **A1** — recusado: `PagamentoRecusadoException`, pedido permanece aguardando. **E1** — timeout/indisponibilidade do gateway: estado "pendente de verificação" e nova consulta posterior; sem cobrança duplicada. **E2** — prazo de pagamento expirou: pedido cancelado e reserva devolvida (P-09).

### UC-13 — Acompanhar Pedido
- **Descrição:** o cliente consulta seus pedidos, status e histórico.
- **Atores:** Cliente.
- **Pré-condições:** cliente autenticado com ao menos um pedido.
- **Fluxo principal:** 1. Cliente abre "Meus Pedidos"; 2. Sistema lista seus pedidos com status; 3. Cliente seleciona um pedido; 4. Sistema exibe itens, valores, endereço, status, histórico e código de rastreio (se enviado).
- **Alternativos/exceção:** **A1** — sem pedidos: mensagem. **A2** — cliente decide cancelar: UC-14. **E1** — pedido de outro cliente: `NaoAutorizadoException`.

### UC-14 — Cancelar Pedido *(extend UC-13)*
- **Descrição:** o cliente cancela um pedido ainda não enviado.
- **Atores:** Cliente.
- **Pré-condições:** pedido do cliente em `AGUARDANDO_PAGAMENTO`, `PAGO` ou `EM_SEPARACAO` (RN-24).
- **Fluxo principal:** 1. Cliente seleciona "Cancelar" e informa o motivo; 2. Sistema pede confirmação; 3. Cliente confirma; 4. Sistema muda o pedido para `CANCELADO` e devolve a reserva/estoque; 5. Sistema notifica por e-mail (UC-16).
- **Alternativos/exceção:** **A1** — pedido `PAGO`: executa **UC-15 Estornar Pagamento** antes do passo 4. **A2** — cliente desiste: nada muda. **E1** — pedido já `ENVIADO`: cancelamento indisponível, orientação a seguir o fluxo de devolução (fora de escopo).

### UC-15 — Estornar Pagamento *(extend UC-14 e UC-23)*
- **Descrição:** devolve ao cliente o valor de um pedido pago e cancelado.
- **Atores:** **Gateway de Pagamento**.
- **Pré-condições:** pedido `PAGO` em cancelamento.
- **Fluxo principal:** 1. Sistema solicita o estorno total ao gateway; 2. Gateway confirma; 3. Sistema registra o estorno no pedido.
- **Alternativos/exceção:** **E1** — gateway indisponível/recusa: estorno marcado como pendente e reprocessado; alerta ao administrador (P-10).

### UC-16 — Notificar Cliente por E-mail *(incluído)*
- **Descrição:** informa o cliente sobre eventos do pedido.
- **Atores:** **Serviço de E-mail**.
- **Pré-condições:** evento de pedido ocorrido (recebido, pago, enviado, entregue, cancelado).
- **Fluxo principal:** 1. Sistema monta a mensagem com dados mínimos; 2. Envia ao Serviço de E-mail de forma assíncrona; 3. Registra o resultado.
- **Alternativos/exceção:** **E1** — falha: até 3 novas tentativas; persistindo, registra falha sem afetar o pedido (RN-26).

### UC-17 — Gerenciar Produtos
- **Descrição:** o administrador cadastra, edita e inativa produtos.
- **Atores:** Administrador.
- **Pré-condições:** administrador autenticado; categorias e marcas cadastradas.
- **Fluxo principal:** 1. Administrador escolhe "Novo produto"; 2. Informa SKU, nome, descrição, especificações, imagens, categoria, marca, preço, peso/dimensões e estoque inicial; 3. Sistema valida (RN-32, RN-33); 4. Sistema salva e publica no catálogo; 5. Auditoria registrada.
- **Alternativos/exceção:** **A1** — edição: altera campos e salva. **A2** — inativação: produto some do catálogo (RN-34). **E1** — SKU duplicado ou dado inválido: erro junto ao campo. **E2** — tentativa de excluir produto com pedidos: só inativação permitida.

### UC-18 — Gerenciar Categorias e Marcas
- **Descrição:** mantém as categorias e marcas usadas na organização e nos filtros.
- **Atores:** Administrador.
- **Pré-condições:** administrador autenticado.
- **Fluxo principal:** 1. Administrador cria ou edita categoria/marca; 2. Sistema valida nome único; 3. Sistema salva e atualiza os filtros.
- **Alternativos/exceção:** **E1** — inativar categoria/marca com produtos ativos: bloqueado (RN-35). **E2** — nome duplicado: erro.

### UC-19 — Gerenciar Estoque
- **Descrição:** consulta e ajusta a quantidade em estoque dos produtos.
- **Atores:** Administrador.
- **Pré-condições:** administrador autenticado; produto cadastrado.
- **Fluxo principal:** 1. Administrador busca o produto; 2. Sistema mostra estoque, reservado e disponível; 3. Administrador registra entrada, saída ou ajuste com motivo (RN-30); 4. Sistema atualiza e registra a movimentação.
- **Alternativos/exceção:** **A1** — estoque abaixo do mínimo após o ajuste: UC-20. **E1** — ajuste negativo além do disponível: rejeitado (RN-29).

### UC-20 — Alertar Estoque Baixo *(extend UC-19)*
- **Descrição:** sinaliza produtos com estoque abaixo do mínimo.
- **Atores:** Administrador.
- **Pré-condições:** estoque mínimo configurado.
- **Fluxo principal:** 1. Sistema detecta estoque < mínimo; 2. Exibe o alerta no painel; 3. Administrador consulta a lista e decide a reposição.
- **Alternativos/exceção:** **A1** — estoque reposto: alerta removido.

### UC-21 — Consultar Pedidos
- **Descrição:** lista e filtra os pedidos da loja.
- **Atores:** Administrador.
- **Pré-condições:** administrador autenticado.
- **Fluxo principal:** 1. Administrador filtra por status, data ou cliente; 2. Sistema lista os pedidos; 3. Administrador abre o detalhe de um pedido.
- **Alternativos/exceção:** **A1** — avança o status: UC-22. **A2** — cancela: UC-23. **A3** — sem resultados: mensagem.

### UC-22 — Atualizar Status do Pedido *(extend UC-21)*
- **Descrição:** faz o pedido avançar na operação logística.
- **Atores:** Administrador.
- **Pré-condições:** pedido em status que permita a transição (tabela 5.5).
- **Fluxo principal:** 1. Administrador seleciona o novo status; 2. Sistema valida a transição (RN-22, RN-36); 3. Em "enviado", informa o código de rastreio; 4. Sistema atualiza, registra auditoria e notifica o cliente (UC-16).
- **Alternativos/exceção:** **E1** — transição inválida: `TransicaoInvalidaException`. **E2** — envio sem rastreio: bloqueado. **E3** — pedido alterado por outro usuário: aviso e recarga.

### UC-23 — Cancelar Pedido (Administrativo) *(extend UC-21)*
- **Descrição:** o administrador cancela um pedido (ex.: ruptura de estoque, suspeita de fraude).
- **Atores:** Administrador.
- **Pré-condições:** pedido em `AGUARDANDO_PAGAMENTO`, `PAGO` ou `EM_SEPARACAO`.
- **Fluxo principal:** 1. Administrador seleciona "Cancelar" e informa o motivo; 2. Confirma; 3. Sistema muda para `CANCELADO`, devolve o estoque e registra auditoria; 4. Notifica o cliente (UC-16).
- **Alternativos/exceção:** **A1** — pedido `PAGO`: executa UC-15. **E1** — pedido já `ENVIADO`: cancelamento bloqueado (RN-24).

---

## 8. Arquitetura e Componentes

### 8.1 Estilo arquitetural (premissa)

**Aplicação web em camadas, modular (monólito modular)**, com front-ends separados do backend e serviços externos acessados por **interfaces/adaptadores** (P-05). Cada módulo de negócio expõe uma interface e consome outras apenas por interface, o que permite desenhar *lollipop/socket* e trocar fornecedores externos (RNF-18).

### 8.2 Pacotes (subsistemas)

| Pacote | Conteúdo |
|---|---|
| **Frontend** | Interface Web da Loja; Painel Administrativo |
| **Backend — Núcleo** | API de Aplicação (fachada/roteamento); Módulo de Usuários e Autenticação; Módulo de Catálogo; Módulo de Busca e Filtros; Módulo de Carrinho; Módulo de Pedidos; Módulo de Estoque; Módulo de Notificações |
| **Backend — Integrações** | Adaptador de Pagamento; Adaptador de Frete; Adaptador de E-mail |
| **Dados** | Banco de Dados (relacional) |
| **Serviços Externos** | Gateway de Pagamento; Serviço de Frete; Serviço de E-mail |

### 8.3 Dicionário de componentes

| Componente | Responsabilidade | Interfaces **fornecidas** (lollipop) | Interfaces **requeridas** (socket) |
|---|---|---|---|
| **Interface Web da Loja** | Telas do cliente/visitante: catálogo, carrinho, checkout, conta e pedidos | — (consumida pelo navegador) | `IApiLoja` (API de Aplicação) |
| **Painel Administrativo** | Telas do administrador: produtos, estoque, pedidos | — | `IApiAdmin` (API de Aplicação) |
| **API de Aplicação** | Ponto único de entrada HTTPS; roteia, valida sessão/perfil (RBAC) e orquestra os módulos | `IApiLoja`, `IApiAdmin` | `IAutenticacao`, `ICatalogo`, `IBusca`, `ICarrinho`, `IPedidos`, `IEstoque` |
| **Módulo de Usuários e Autenticação** | Cadastro, login, recuperação de senha, sessões, perfis e endereços | `IAutenticacao`, `IUsuarios` | `IRepositorio`, `INotificacao` (e-mail de recuperação) |
| **Módulo de Catálogo** | Produtos, categorias e marcas; detalhes e regras de publicação | `ICatalogo` | `IRepositorio`, `IEstoque` (disponibilidade exibida) |
| **Módulo de Busca e Filtros** | Busca textual, filtros e ordenação sobre o catálogo | `IBusca` | `ICatalogo` (ou `IRepositorio`) |
| **Módulo de Carrinho** | Itens, quantidades, subtotal; mescla visitante→cliente | `ICarrinho` | `ICatalogo` (preços), `IEstoque` (disponibilidade), `IFrete` (simulação), `IRepositorio` |
| **Módulo de Pedidos** | Checkout, criação do pedido, status, cancelamento e orquestração de pagamento/estoque | `IPedidos` | `ICarrinho`, `IEstoque`, `IFrete`, `IPagamento`, `INotificacao`, `IRepositorio`, `IUsuarios` |
| **Módulo de Estoque** | Disponibilidade, reserva, baixa, devolução, ajustes e alertas | `IEstoque` | `IRepositorio` |
| **Módulo de Notificações** | Monta e agenda e-mails de eventos do pedido; reprocessa falhas | `INotificacao` | `IEmail` (adaptador), `IRepositorio` |
| **Adaptador de Pagamento** | Traduz o domínio para o protocolo do gateway; tokenização; webhook de confirmação | `IPagamento` | `IGatewayPagamento` (externo) |
| **Adaptador de Frete** | Traduz itens/CEP para o serviço de frete e normaliza as opções | `IFrete` | `IServicoFrete` (externo) |
| **Adaptador de E-mail** | Envia mensagens ao provedor de e-mail | `IEmail` | `IServicoEmail` (externo) |
| **Banco de Dados** | Persistência de usuários, produtos, estoque, pedidos, auditoria | `IRepositorio` (SQL/ORM) | — |
| **Gateway de Pagamento** *(externo)* | Autoriza, captura e estorna pagamentos | `IGatewayPagamento` | — (chama o webhook do Adaptador) |
| **Serviço de Frete** *(externo)* | Cotação de frete | `IServicoFrete` | — |
| **Serviço de E-mail** *(externo)* | Entrega de e-mails | `IServicoEmail` | — |

### 8.4 Dependências principais (setas tracejadas)

- Interface Web da Loja → API de Aplicação (`IApiLoja`); Painel Administrativo → API de Aplicação (`IApiAdmin`);
- API de Aplicação → todos os módulos de negócio pelas interfaces;
- Pedidos → Carrinho, Estoque, Pagamento, Frete, Notificações;
- Carrinho → Catálogo, Estoque, Frete;
- Busca → Catálogo; Catálogo → Estoque;
- Notificações → Adaptador de E-mail; Pagamento/Frete/E-mail → serviços externos;
- Módulos de negócio → Banco de Dados (`IRepositorio`);
- **Não** há dependência direta de módulos de negócio com serviços externos (somente via adaptador).

### 8.5 Portas (opcional/recomendado)

| Componente | Porta | Função |
|---|---|---|
| API de Aplicação | `porta HTTPS` | Entrada pública (REST/JSON) para os dois front-ends |
| Adaptador de Pagamento | `porta Webhook` | Recebe a confirmação assíncrona do gateway (RF-18) |
| Adaptador de Pagamento, Frete e E-mail | `porta de saída` | Chamadas ao serviço externo respectivo |
| Banco de Dados | `porta SQL` | Acesso pelos módulos |

---

## 9. Premissas e Decisões de Projeto

| ID | Premissa / Decisão | Justificativa |
|---|---|---|
| **P-01** | O sistema é um **web app responsivo**. | O tema fala em "site"; evita escopo de app nativo. |
| **P-02** | Cadastro/autenticação de cliente e administrador foram **acrescentados** (RF-06…RF-08). | A descrição não os cita, mas "finalizar pedido" e "módulo administrativo" pressupõem identificação. |
| **P-03** | Há um **único perfil administrativo** (Administrador) que gerencia produtos, estoque e pedidos. | A descrição fala em "módulo administrativo" sem diferenciar papéis; evita complexidade. Pode ser dividido em Gerente de Estoque e Atendente em evolução futura. |
| **P-04** | Formas de pagamento: **cartão, Pix e boleto**, todas via gateway. | Padrão do mercado brasileiro; a descrição apenas cita "gateways de pagamento". |
| **P-05** | Arquitetura em **monólito modular** com **adaptadores** para sistemas externos. | Adequada ao escopo; facilita representar interfaces fornecidas/requeridas e trocar fornecedores. |
| **P-06** | **Visitante** pode navegar e montar carrinho; autenticação só é exigida no checkout. | Prática de e-commerce que reduz atrito; justifica a generalização `Cliente ▷ Visitante`. |
| **P-07** | **Serviço de E-mail** é ator secundário e as notificações são só por e-mail. | Complemento mínimo para "acompanhar status do pedido"; sem SMS/WhatsApp. |
| **P-08** | Perfis **exclusivos** (cliente ou administrador). | Simplifica o RBAC. |
| **P-09** | Pedido `AGUARDANDO_PAGAMENTO` **expira** (ex.: 24 h Pix/boleto, valor parametrizável) e devolve a reserva. | Evita estoque preso indefinidamente. Executado por agendador interno (Sistema). |
| **P-10** | Falha no estorno **não bloqueia** o cancelamento: o estorno fica pendente e é reprocessado, com alerta ao administrador. | Evita pedido preso; garante rastreabilidade. |
| **P-11** | Dimensões/peso são atributos do produto e usados no frete. | Necessário para o cálculo; sem isso o frete não é viável. |
| **P-12** | Fora de escopo: cupons, avaliações, devolução, nota fiscal, marketplace. | Não citados; mantêm o trabalho dentro do escopo sugerido. |
| **P-13** | Cancelamento é permitido até `EM_SEPARACAO`; depois de enviado, é devolução (fora de escopo). | Regra comum; simplifica o fluxo. |
| **P-14** | **Alerta de estoque baixo** é um complemento leve para dar sentido ao módulo de estoque e ilustrar um `extend`. | Justificado como boa prática; pode ser removido sem impacto no restante (RF-26, RN-31, UC-20). |
| **P-15** | **IDs** (RF, RNF, RN, UC) devem ser mantidos nos diagramas e no PDF. | Rastreabilidade entre artefatos. |
| **P-16** | A ferramenta de modelagem será **PlantUML** (como na APS 1), com fontes `.puml` entregues junto. | Reuso do fluxo `scripts/` e dos padrões já usados. *(Confirmar com o grupo.)* |

---

## Apêndice — Rastreabilidade

### A. Casos de uso × requisitos × componentes

| UC | RF | Componentes principais |
|---|---|---|
| UC-01…UC-04 | RF-01…RF-05 | Interface Web, API, Catálogo, Busca, Estoque |
| UC-05…UC-07 | RF-06…RF-09 | Interface Web, API, Usuários/Autenticação, Notificações |
| UC-08, UC-09 | RF-10…RF-13, RF-23 | Carrinho, Catálogo, Estoque |
| UC-10 | RF-14 | Carrinho/Pedidos, Adaptador de Frete, Serviço de Frete |
| UC-11, UC-12 | RF-15…RF-18, RF-24 | Pedidos, Estoque, Adaptador de Pagamento, Gateway |
| UC-13…UC-16 | RF-19…RF-22 | Pedidos, Adaptador de Pagamento, Notificações, Adaptador de E-mail |
| UC-17, UC-18 | RF-27…RF-29 | Painel Administrativo, API, Catálogo |
| UC-19, UC-20 | RF-25, RF-26 | Painel Administrativo, Estoque |
| UC-21…UC-23 | RF-30…RF-32 | Painel Administrativo, Pedidos, Notificações |

### B. Entregáveis esperados pelo professor × onde estão apoiados

| Entregável | Base nesta especificação |
|---|---|
| Capa e identificação | (a preencher pelo grupo) |
| Visão geral | Seção 1 |
| Diagrama de Casos de Uso (fronteira, atores, associação, generalização, include/extend) | Seções 2 e 6 |
| Especificação textual dos casos de uso | Seção 7 |
| Diagrama de Componentes (interfaces fornecidas/requeridas, dependências, portas, pacotes) | Seção 8 |
| Dicionário de componentes | Seção 8.3 |
| Arquivos-fonte e lista de símbolos da ferramenta | P-16 |

---

*Fim da especificação — base para a produção do Diagrama de Casos de Uso e do Diagrama de Componentes exigidos pela APS 2.*
