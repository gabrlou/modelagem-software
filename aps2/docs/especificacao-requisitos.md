# Especificação de Requisitos — Site de Loja de Eletrônicos (E-commerce)

> **Documento:** APS 2 — Modelagem de Software
> **Finalidade:** base para o Diagrama de Casos de Uso (~4 casos de uso) e para os Diagramas de Componentes (geral + micros)
> **Versão:** 2.0 (enxuta, conforme adendos do professor)

---

## 1. Visão Geral

O sistema é um **site de loja de eletrônicos**. O cliente navega por um catálogo (filtros por categoria, marca e preço), monta um carrinho e finaliza o pedido, com **pagamento por gateway** e **cálculo de frete** por serviço externo. Um **módulo administrativo** permite cadastrar produtos, gerenciar o estoque e acompanhar o status dos pedidos.

**Fora do escopo:** cupons, avaliações, devolução, nota fiscal, marketplace, app nativo.

---

## 2. Atores

| Ator | Tipo | Descrição |
|---|---|---|
| **Visitante** | Principal | Navega, filtra, busca e monta carrinho sem estar autenticado |
| **Cliente** | Principal | Visitante autenticado; finaliza e acompanha pedidos (**herda** de Visitante — generalização) |
| **Administrador** | Principal | Gerencia produtos, estoque e pedidos |
| **Gateway de Pagamento** | Secundário (externo) | Autoriza e estorna pagamentos |
| **Serviço de Frete** | Secundário (externo) | Calcula valor e prazo de entrega |
| **Serviço de E-mail** | Secundário (externo) | Envia e-mails de status do pedido |

---

## 3. Requisitos Funcionais

| ID | Requisito |
|---|---|
| RF-01 | Listar produtos do catálogo com filtros por categoria, marca e faixa de preço, além de busca por texto |
| RF-02 | Exibir detalhes do produto (descrição, preço, disponibilidade) |
| RF-03 | Cadastrar e autenticar cliente (e-mail e senha) |
| RF-04 | Adicionar, alterar e remover itens do carrinho |
| RF-05 | Calcular frete por CEP via serviço externo |
| RF-06 | Finalizar pedido (endereço, frete, forma de pagamento, confirmação) |
| RF-07 | Processar pagamento pelo gateway (cartão, Pix ou boleto) |
| RF-08 | Acompanhar o status do pedido e cancelá-lo antes do envio (com estorno se já pago) |
| RF-09 | Notificar o cliente por e-mail nas mudanças de status |
| RF-10 | Cadastrar, editar e inativar produtos (admin) |
| RF-11 | Gerenciar estoque: ajustar quantidades (admin) |
| RF-12 | Consultar pedidos e atualizar o status (admin) |

---

## 4. Requisitos Não Funcionais

| ID | Requisito |
|---|---|
| RNF-01 | Comunicação por HTTPS; senhas com hash |
| RNF-02 | Acesso por perfil: área administrativa restrita ao Administrador |
| RNF-03 | Dados de cartão não são armazenados pela loja (tratados pelo gateway) |
| RNF-04 | Interface responsiva (celular e desktop) |
| RNF-05 | Falha em serviço externo não derruba o restante do sistema |
| RNF-06 | Arquitetura modular, com interfaces bem definidas, para trocar o gateway ou o frete sem afetar o núcleo |

---

## 5. Regras de Negócio

| ID | Regra |
|---|---|
| RN-01 | Só produtos ativos aparecem no catálogo; sem estoque, o produto fica indisponível |
| RN-02 | A quantidade no carrinho não pode exceder o estoque disponível |
| RN-03 | Só cliente autenticado finaliza pedido |
| RN-04 | O estoque é reservado ao criar o pedido e baixado quando o pagamento é aprovado |
| RN-05 | Pedido pode ser cancelado até o envio; se já estava pago, o valor é estornado |
| RN-06 | Falha ao enviar e-mail não desfaz nenhuma operação |

**Status do pedido:** `AGUARDANDO_PAGAMENTO` → `PAGO` → `EM_SEPARACAO` → `ENVIADO` → `ENTREGUE`; `CANCELADO` a partir de qualquer status anterior a `ENVIADO`.

---

## 6. Casos de Uso (diagrama)

### 6.1 Casos de uso principais

| ID | Caso de uso | Ator principal | Atores secundários |
|---|---|---|---|
| UC-01 | Navegar no Catálogo | Visitante | — |
| UC-02 | Finalizar Pedido | Cliente | Gateway de Pagamento, Serviço de Frete |
| UC-03 | Acompanhar Pedido | Cliente | Serviço de E-mail |
| UC-04 | Gerenciar Produtos e Estoque | Administrador | — |

### 6.2 Relacionamentos

- **Associação:** cada ator com seus casos de uso (tabela acima).
- **Generalização:** `Cliente` ▷ `Visitante`.
- **«include»:**
  - UC-02 inclui **Autenticar Usuário** (RN-03);
  - UC-02 inclui **Calcular Frete** (Serviço de Frete);
  - UC-02 inclui **Processar Pagamento** (Gateway de Pagamento);
  - UC-04 inclui **Autenticar Usuário** (área restrita).
- **«extend»:**
  - **Filtrar/Buscar Produtos** estende UC-01;
  - **Cancelar Pedido** estende UC-03 (condição: pedido ainda não enviado).

> Casos de uso de apoio (Autenticar Usuário, Calcular Frete, Processar Pagamento, Filtrar/Buscar, Cancelar) aparecem apenas como incluídos/estendidos; os principais são os 4 da tabela.

### 6.3 Fronteira

Caixa **"Sistema de E-commerce de Eletrônicos"**. Dentro: casos de uso. Fora: Visitante, Cliente, Administrador (esquerda) e Gateway, Frete e E-mail (direita).

---

## 7. Especificação Textual dos Casos de Uso

### UC-01 — Navegar no Catálogo
- **Descrição:** o visitante explora os produtos ativos, com filtros (categoria, marca, preço) e busca.
- **Atores:** Visitante.
- **Pré-condições:** nenhuma.
- **Fluxo principal:** 1. Visitante acessa o catálogo; 2. Sistema lista os produtos; 3. Visitante abre um produto; 4. Sistema mostra detalhes e disponibilidade.
- **Alternativos/exceção:** **A1** — aplica filtro ou busca (extend): sistema atualiza a lista. **A2** — sem resultados: mensagem sugerindo limpar filtros.

### UC-02 — Finalizar Pedido
- **Descrição:** transforma o carrinho em pedido, com endereço, frete e pagamento.
- **Atores:** Cliente; Serviço de Frete; Gateway de Pagamento.
- **Pré-condições:** carrinho com ao menos um item.
- **Fluxo principal:** 1. Cliente inicia o checkout e se autentica (include); 2. Informa o endereço; 3. Sistema calcula o frete com o Serviço de Frete (include) e o cliente escolhe; 4. Escolhe a forma de pagamento; 5. Confirma o resumo; 6. Sistema reserva o estoque e cria o pedido; 7. Sistema envia a cobrança ao Gateway (include); 8. Pagamento aprovado: pedido fica `PAGO`.
- **Alternativos/exceção:** **A1** — Pix/boleto: pedido aguarda a confirmação do gateway. **E1** — pagamento recusado: cliente tenta outra forma. **E2** — item sem estoque: sistema avisa e ajusta o carrinho. **E3** — frete indisponível: checkout não avança.

### UC-03 — Acompanhar Pedido
- **Descrição:** o cliente consulta status e histórico de seus pedidos; pode cancelar antes do envio.
- **Atores:** Cliente; Serviço de E-mail.
- **Pré-condições:** cliente autenticado com ao menos um pedido.
- **Fluxo principal:** 1. Cliente abre "Meus Pedidos"; 2. Sistema lista os pedidos com status; 3. Cliente abre um pedido; 4. Sistema mostra itens, valores e histórico.
- **Alternativos/exceção:** **A1** — cancelar (extend): cliente informa o motivo e confirma; sistema cancela, devolve o estoque, estorna o pagamento (se pago) e envia e-mail. **E1** — pedido já enviado: cancelamento indisponível.

### UC-04 — Gerenciar Produtos e Estoque
- **Descrição:** o administrador mantém o catálogo e as quantidades em estoque, e acompanha os pedidos.
- **Atores:** Administrador.
- **Pré-condições:** administrador autenticado (include).
- **Fluxo principal:** 1. Administrador cadastra ou edita um produto (nome, descrição, categoria, marca, preço, peso/dimensões); 2. Sistema valida e publica; 3. Administrador ajusta o estoque; 4. Sistema atualiza a quantidade; 5. Administrador consulta pedidos e atualiza o status (em separação, enviado, entregue).
- **Alternativos/exceção:** **A1** — inativar produto: some do catálogo. **E1** — dados inválidos: erro junto ao campo. **E2** — ajuste deixaria o estoque negativo: rejeitado.

---

## 8. Arquitetura e Componentes

### 8.1 Visão geral (Diagrama de Componentes — **geral**)

Aplicação web em camadas, com serviços externos acessados por **adaptadores** (RNF-06).

| Pacote | Componentes |
|---|---|
| **Frontend** | Interface da Loja; Painel Administrativo |
| **Backend** | API; Catálogo; Carrinho; Pedidos; Estoque; Usuários; Notificações |
| **Integrações** | Adaptador de Pagamento; Adaptador de Frete; Adaptador de E-mail |
| **Dados** | Banco de Dados |
| **Externos** | Gateway de Pagamento; Serviço de Frete; Serviço de E-mail |

### 8.2 Dicionário de componentes (geral)

| Componente | Responsabilidade | Fornece | Requer |
|---|---|---|---|
| Interface da Loja | Telas de catálogo, carrinho, checkout e pedidos | — | `IApi` |
| Painel Administrativo | Telas de produtos, estoque e pedidos | — | `IApi` |
| API | Entrada HTTPS; valida sessão e perfil; encaminha aos módulos | `IApi` | `IUsuarios`, `ICatalogo`, `ICarrinho`, `IPedidos`, `IEstoque` |
| Usuários | Cadastro, login e perfis | `IUsuarios` | `IDados` |
| Catálogo | Produtos, categorias, marcas, filtros e busca | `ICatalogo` | `IDados`, `IEstoque` |
| Carrinho | Itens e subtotal | `ICarrinho` | `ICatalogo`, `IEstoque`, `IFrete` |
| Pedidos | Checkout, status e cancelamento | `IPedidos` | `ICarrinho`, `IEstoque`, `IFrete`, `IPagamento`, `INotificacao`, `IDados` |
| Estoque | Disponibilidade, reserva, baixa e ajustes | `IEstoque` | `IDados` |
| Notificações | Monta e envia e-mails de status | `INotificacao` | `IEmail` |
| Adaptador de Pagamento | Fala com o gateway | `IPagamento` | `IGateway` (externo) |
| Adaptador de Frete | Fala com o serviço de frete | `IFrete` | `IServicoFrete` (externo) |
| Adaptador de E-mail | Fala com o serviço de e-mail | `IEmail` | `IServicoEmail` (externo) |
| Banco de Dados | Persistência | `IDados` | — |

**Portas:** a **API** tem uma porta HTTPS (entrada pública); o **Adaptador de Pagamento** tem uma porta de webhook (confirmação de Pix/boleto).

### 8.3 Diagramas **micro** (componentes internos)

Um diagrama por componente principal, mostrando os componentes dentro dele.

**Micro 1 — Pedidos** (núcleo do checkout)

| Componente interno | Responsabilidade |
|---|---|
| Gerenciador de Checkout | Conduz endereço, frete e pagamento e cria o pedido |
| Máquina de Status | Valida as transições de status do pedido |
| Repositório de Pedidos | Grava e consulta pedidos (usa `IDados`) |
| Orquestrador de Pagamento | Chama `IPagamento` e trata aprovado/recusado/pendente |

Interface fornecida pelo componente: `IPedidos`, exposta pelo Gerenciador de Checkout e pela Máquina de Status. Requeridas: `ICarrinho`, `IEstoque`, `IFrete`, `IPagamento`, `INotificacao`, `IDados`.

**Micro 2 — Catálogo**

| Componente interno | Responsabilidade |
|---|---|
| Gerenciador de Produtos | Cadastro, edição e inativação (admin) |
| Serviço de Consulta | Listagem, filtros e busca |
| Repositório de Produtos | Persistência (usa `IDados`) |

**Micro 3 — Adaptador de Pagamento**

| Componente interno | Responsabilidade |
|---|---|
| Cliente do Gateway | Envia cobrança e estorno ao gateway externo |
| Receptor de Webhook | Recebe a confirmação assíncrona (porta de webhook) |
| Tradutor de Resultado | Converte a resposta do gateway para o modelo da loja |

> Se o grupo preferir, os micros de **Estoque** e **Carrinho** podem ser adicionados da mesma forma.

---

## 9. Premissas

| ID | Premissa |
|---|---|
| P-01 | Aplicação web responsiva (não nativa) |
| P-02 | Cadastro e autenticação foram acrescentados: "finalizar pedido" e "módulo administrativo" pressupõem identificação |
| P-03 | Um único perfil administrativo |
| P-04 | Formas de pagamento: cartão, Pix e boleto, todas via gateway |
| P-05 | Visitante pode montar carrinho; a autenticação é exigida só no checkout |
| P-06 | O Serviço de E-mail é o único canal de notificação |
| P-07 | Pedidos aguardando pagamento por muito tempo expiram e devolvem o estoque |
| P-08 | Foram modelados só ~4 casos de uso principais (orientação do professor); as demais funcionalidades constam em RF mas não viram caso de uso no diagrama |
| P-09 | Ferramenta de modelagem: PlantUML, com fontes `.puml` entregues junto *(a confirmar com o grupo)* |

---

## Apêndice — Rastreabilidade

| Artefato | Requisitos |
|---|---|
| UC-01 Navegar no Catálogo | RF-01, RF-02, RN-01 |
| UC-02 Finalizar Pedido | RF-03…RF-07, RN-02…RN-04 |
| UC-03 Acompanhar Pedido | RF-08, RF-09, RN-05, RN-06 |
| UC-04 Gerenciar Produtos e Estoque | RF-10…RF-12 |
| Componentes (geral e micros) | RNF-05, RNF-06 e todos os RF |
