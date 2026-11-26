# Especificação de Requisitos: Site de Loja de Eletrônicos (E-commerce)

> **Documento:** APS2: Modelagem de Software
> **Finalidade:** base para o Diagrama de Casos de Uso (~4 casos de uso) e para os Diagramas de Componentes (geral + 2 micros: Frontend e Backend)
> **Versão:** 2.0 (enxuta, conforme adendos do professor)

---

## 1. Visão Geral

### 1.1 Descrição e contexto

O tema é o **Site de Loja de Eletrônicos (e-commerce)**, uma loja virtual de produtos eletrônicos de consumo, como smartphones, notebooks, tablets, televisores, fones de ouvido e acessórios. O público são consumidores finais, que compram pela internet e recebem o pedido em casa. Como esse tipo de produto tem muitos modelos, marcas e faixas de preço, o cliente navega por um catálogo com filtros por categoria, marca e preço, adiciona itens a um carrinho e finaliza o pedido. O pagamento e o cálculo de frete são feitos por serviços externos (um **gateway de pagamento** e um **serviço de frete**), e o cliente é avisado por e-mail sobre o andamento do pedido. Do lado da loja, uma **área administrativa** permite cadastrar produtos, controlar o estoque, que é limitado e muda a cada venda, e acompanhar o status dos pedidos.

### 1.2 Escopo

**Dentro do escopo:** cadastro e autenticação de clientes e do administrador; catálogo com navegação, filtros e busca; carrinho; finalização do pedido, com cálculo de frete e pagamento (cartão, Pix ou boleto); consulta e cancelamento de pedidos pelo cliente; área administrativa (produtos, estoque e status dos pedidos); avisos por e-mail ao cliente.

**Fora do escopo:** cupons e promoções, avaliações de produtos, troca e devolução, nota fiscal, marketplace e aplicativo nativo.

### 1.3 Premissas

| ID | Premissa |
|---|---|
| P-01 | Aplicação web responsiva |
| P-02 | Cadastro e autenticação foram acrescentados: finalizar pedido e acessar a área administrativa pressupõem identificação |
| P-03 | Um único perfil administrativo |
| P-04 | Formas de pagamento: cartão, Pix e boleto, todas pelo gateway de pagamento |
| P-05 | Visitante pode montar o carrinho; a autenticação é exigida só na finalização do pedido |
| P-06 | O e-mail é o único canal de notificação |
| P-07 | Categorias e marcas são criadas pelo administrador ao cadastrar o produto |
| P-08 | O pedido aguardando pagamento é cancelado após um prazo, e o estoque reservado é devolvido |
| P-09 | Os produtos e categorias citados são exemplos adotados pelo grupo; o catálogo real é cadastrado pelo administrador |

---

## 2. Atores

| Ator | Tipo | Descrição |
|---|---|---|
| **Visitante** | Principal | Navega, filtra, busca e monta carrinho sem estar autenticado |
| **Cliente** | Principal | Visitante autenticado; finaliza e consulta pedidos (**herda** de Visitante, generalização) |
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
| RF-07 | Processar pagamento pelo gateway de pagamento (cartão, Pix ou boleto) |
| RF-08 | Consultar os pedidos e o status deles e cancelar antes do envio (com estorno se já pago) |
| RF-09 | Avisar o cliente por e-mail nas mudanças de status do pedido, inclusive no cancelamento |
| RF-10 | Cadastrar, editar e inativar produtos (admin) |
| RF-11 | Gerenciar estoque: ajustar quantidades (admin) |
| RF-12 | Consultar pedidos e atualizar o status (admin) |

---

## 4. Requisitos Não Funcionais

| ID | Requisito |
|---|---|
| RNF-01 | Comunicação por HTTPS; senhas com hash |
| RNF-02 | Acesso por perfil: área administrativa restrita ao Administrador |
| RNF-03 | Dados de cartão não são armazenados pela loja (tratados pelo gateway de pagamento) |
| RNF-04 | Interface responsiva (celular e desktop) |
| RNF-05 | Falha em serviço externo não derruba o restante do sistema |
| RNF-06 | Arquitetura modular, com interfaces bem definidas, para trocar o gateway de pagamento ou o serviço de frete sem afetar o núcleo |

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

**Status do pedido:** aguardando pagamento → pago → em separação → enviado → entregue. O pedido pode ser cancelado a partir de qualquer status anterior a enviado.

---

## 6. Casos de Uso (diagrama)

### 6.1 Casos de uso

Principais:

| ID | Caso de uso | Ator principal | Atores secundários |
|---|---|---|---|
| CU-01 | Navegar no Catálogo | Visitante | Nenhum |
| CU-02 | Gerenciar Carrinho | Visitante, Cliente | Nenhum |
| CU-03 | Finalizar Pedido | Cliente | Gateway de Pagamento, Serviço de Frete |
| CU-04 | Consultar Pedidos | Cliente | Nenhum |
| CU-05 | Gerenciar Produtos e Estoque | Administrador | Nenhum |
| CU-06 | Gerenciar Pedidos | Administrador | Nenhum |

De apoio, que aparecem no diagrama como incluídos ou estendidos:

| ID | Caso de uso | Relação | Ator |
|---|---|---|---|
| CU-07 | Filtrar / Buscar Produtos | estende CU-01 | Visitante |
| CU-08 | Cancelar Pedido | estende CU-04 | Cliente; Gateway de Pagamento |
| CU-09 | Autenticar Usuário | incluído por CU-03, CU-04, CU-05 e CU-06 | Cliente, Administrador |
| CU-10 | Calcular Frete | incluído por CU-03 | Serviço de Frete |
| CU-11 | Processar Pagamento | incluído por CU-03 | Gateway de Pagamento |
| CU-12 | Cadastrar-se | caso de uso próprio do Visitante | Visitante |
| CU-13 | Notificar Cliente por E-mail | incluído por CU-06 e CU-08 | Serviço de E-mail |

### 6.2 Relacionamentos

- **Associação:** cada ator com seus casos de uso (tabelas acima).
- **Generalização:** `Cliente` ▷ `Visitante`.
- **«include»:** CU-03 inclui CU-09 (RN-03), CU-10 e CU-11; CU-04, CU-05 e CU-06 incluem CU-09 (conta própria ou área restrita); CU-06 e CU-08 incluem CU-13 (aviso por e-mail).
- **«extend»:** CU-07 estende CU-01 (nota: {condição: aplica filtro ou busca}); CU-08 estende CU-04 (nota: {condição: pedido ainda não enviado}).

### 6.3 Fronteira

Caixa **"Site de Loja de Eletrônicos"**. Dentro: casos de uso. Fora: Visitante, Cliente, Administrador (esquerda) e Gateway de Pagamento, Serviço de Frete e Serviço de E-mail (direita).

### 6.4 Decisões de modelagem

- **Cliente herda de Visitante:** o visitante já navega e monta o carrinho sem conta; o cliente é o visitante autenticado e ganha o que exige identificação (finalizar e consultar pedidos).
- **Autenticar Usuário é «include»:** passo obrigatório e igual para quem finaliza ou consulta pedidos e para o administrador.
- **Calcular Frete e Processar Pagamento são «include» de Finalizar Pedido:** o pedido não existe sem eles, e cada um envolve um sistema externo.
- **Filtrar / Buscar e Cancelar são «extend»:** o fluxo se completa sem eles; só entram quando a condição acontece.
- **Notificar Cliente por E-mail é um caso à parte:** dois casos disparam o aviso (CU-06 e CU-08), e o Serviço de E-mail aparece em um ponto só.
- **Cadastrar-se é um caso próprio do Visitante:** é a forma de virar cliente, e não uma variação do login.

---

## 7. Especificação Textual dos Casos de Uso

### CU-01: Navegar no Catálogo
- **Atores envolvidos:** **Visitante**.
- **Pré-condições:** Nenhuma.
- **Fluxo principal:**
  1. O **Visitante** acessa o catálogo.
  2. O sistema lista os produtos ativos da loja.
  3. O **Visitante** abre um produto.
  4. O sistema mostra os detalhes e a disponibilidade.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 2, se o **Visitante** aplica um filtro ou faz uma busca,
    - então o sistema atualiza a lista (CU-07).
  - **A2**: No passo 2, se nenhum produto atende ao filtro,
    - então o sistema sugere limpar os filtros.

### CU-02: Gerenciar Carrinho
- **Atores envolvidos:** **Visitante**; **Cliente**.
- **Pré-condições:** Produto ativo e disponível.
- **Fluxo principal:**
  1. O **Visitante** ou o **Cliente** adiciona um produto ao carrinho, informando a quantidade.
  2. O sistema confere a disponibilidade.
  3. O sistema atualiza o carrinho e o subtotal.
  4. O **Visitante** ou o **Cliente** altera quantidades ou remove itens.
  5. O **Visitante** ou o **Cliente** segue para a finalização do pedido (CU-03) ou continua comprando.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 5, se o **Visitante** se autentica na finalização,
    - então os itens do carrinho dele passam para a conta do **Cliente**.
  - **E1**: No passo 2, se a quantidade pedida é maior que o estoque,
    - então o sistema limita ao disponível e avisa.
  - **E2**: No passo 3, se um item fica indisponível,
    - então o sistema sinaliza no carrinho.

### CU-03: Finalizar Pedido
- **Atores envolvidos:** **Cliente**. O **Serviço de Frete** (CU-10) e o **Gateway de Pagamento** (CU-11) participam pelos casos de uso incluídos.
- **Pré-condições:** Carrinho com ao menos um item.
- **Fluxo principal:**
  1. O **Cliente** inicia a finalização do pedido e se autentica (include, CU-09).
  2. O **Cliente** informa o endereço de entrega.
  3. O sistema obtém as opções de frete com o **Serviço de Frete** (include, CU-10).
  4. O **Cliente** escolhe uma opção de frete.
  5. O **Cliente** escolhe a forma de pagamento (cartão, Pix ou boleto).
  6. O **Cliente** confirma o resumo do pedido.
  7. O sistema reserva o estoque e cria o pedido.
  8. O sistema envia a cobrança ao **Gateway de Pagamento** (include, CU-11).
  9. O pagamento é aprovado e o pedido passa para pago.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 9, se o pagamento é por Pix ou boleto,
    - então o pedido aguarda a confirmação do **Gateway de Pagamento**.
  - **E1**: No passo 9, se o pagamento é recusado,
    - então o **Cliente** pode tentar outra forma de pagamento.
  - **E2**: No passo 7, se algum item está sem estoque,
    - então o sistema avisa e ajusta o carrinho.
  - **E3**: No passo 3, se o frete está indisponível,
    - então a finalização do pedido não avança.

### CU-04: Consultar Pedidos
- **Atores envolvidos:** **Cliente**.
- **Pré-condições:** Cliente com conta cadastrada e ao menos um pedido.
- **Fluxo principal:**
  1. O **Cliente** se autentica (include, CU-09).
  2. O **Cliente** abre "Meus Pedidos".
  3. O sistema lista os pedidos do **Cliente**, em andamento e anteriores, com data e status.
  4. O **Cliente** abre um pedido.
  5. O sistema mostra os itens, os valores e o histórico.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 5, se o **Cliente** decide cancelar o pedido,
    - então segue o cancelamento (CU-08).
  - **E1**: No passo 4, se o pedido pertence a outro cliente,
    - então o sistema nega o acesso.

### CU-05: Gerenciar Produtos e Estoque
- **Atores envolvidos:** **Administrador**.
- **Pré-condições:** Conta de administrador cadastrada.
- **Fluxo principal:**
  1. O **Administrador** se autentica (include, CU-09).
  2. O **Administrador** cadastra ou edita um produto (nome, descrição, categoria, marca, preço, peso e dimensões).
  3. O sistema valida os dados e publica o produto no catálogo.
  4. O **Administrador** ajusta a quantidade em estoque.
  5. O sistema atualiza a quantidade.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 2, se o **Administrador** inativa um produto,
    - então ele deixa de aparecer no catálogo.
  - **A2**: No passo 2, se a categoria ou a marca do produto ainda não existe,
    - então o **Administrador** a cria junto com o produto.
  - **E1**: No passo 3, se os dados do produto são inválidos,
    - então o sistema mostra o erro junto ao campo.
  - **E2**: No passo 5, se o ajuste deixaria o estoque negativo,
    - então o sistema rejeita o ajuste.

### CU-06: Gerenciar Pedidos
- **Atores envolvidos:** **Administrador**. O **Serviço de E-mail** participa pelo caso de uso incluído (CU-13).
- **Pré-condições:** Conta de administrador cadastrada e ao menos um pedido na loja.
- **Fluxo principal:**
  1. O **Administrador** se autentica (include, CU-09).
  2. O **Administrador** filtra os pedidos por status, data ou cliente.
  3. O sistema lista os pedidos.
  4. O **Administrador** abre um pedido e escolhe o novo status (em separação, enviado ou entregue).
  5. O sistema valida a mudança e atualiza o pedido.
  6. O sistema avisa o **Cliente** por e-mail (include, CU-13).
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 4, se o novo status é "enviado",
    - então o **Administrador** informa o código de rastreio.
  - **A2**: No passo 4, se o **Administrador** cancela o pedido,
    - então informa o motivo e o sistema segue o cancelamento (CU-08).
  - **E1**: No passo 5, se a mudança de status não é permitida,
    - então o sistema rejeita.
  - **E2**: No passo 5, se o envio não tem código de rastreio,
    - então o sistema bloqueia.

### CU-07: Filtrar / Buscar Produtos
- **Atores envolvidos:** **Visitante**.
- **Pré-condições:** Lista de produtos exibida (CU-01).
- **Fluxo principal:**
  1. O **Visitante** escolhe categoria, marca e/ou faixa de preço, ou digita um termo de busca.
  2. O sistema aplica os filtros.
  3. O sistema atualiza a lista e o total de resultados.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 3, se nenhum produto atende ao filtro,
    - então o sistema sugere limpar os filtros.
  - **E1**: No passo 2, se a faixa de preço é inválida (mínimo maior que o máximo),
    - então o sistema pede a correção.

### CU-08: Cancelar Pedido
- **Atores envolvidos:** **Cliente**; **Gateway de Pagamento**. O **Serviço de E-mail** participa pelo caso de uso incluído (CU-13).
- **Pré-condições:** Pedido do cliente com status aguardando pagamento, pago ou em separação.
- **Fluxo principal:**
  1. O **Cliente** escolhe cancelar o pedido e informa o motivo.
  2. O **Cliente** confirma o cancelamento.
  3. O sistema muda o pedido para cancelado e devolve o estoque.
  4. O sistema avisa o **Cliente** por e-mail (include, CU-13).
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 3, se o pedido já foi pago,
    - então o sistema solicita o estorno ao **Gateway de Pagamento** antes de cancelar.
  - **A2**: No passo 2, se o **Cliente** desiste na confirmação,
    - então nada é alterado.
  - **E1**: No passo 1, se o pedido já foi enviado,
    - então o cancelamento fica indisponível.
  - **E2**: No passo 3, se o estorno falha,
    - então ele fica pendente e é refeito depois, sem bloquear o cancelamento.

### CU-09: Autenticar Usuário
- **Atores envolvidos:** **Cliente**; **Administrador**.
- **Pré-condições:** Conta cadastrada.
- **Fluxo principal:**
  1. O **Cliente** ou o **Administrador** informa e-mail e senha.
  2. O sistema valida as credenciais.
  3. O sistema identifica o perfil e inicia a sessão.
  4. O sistema devolve o usuário ao ponto em que estava.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 1, se o visitante ainda não tem conta,
    - então ele se cadastra (CU-12) e é autenticado em seguida.
  - **E1**: No passo 2, se as credenciais são inválidas,
    - então o sistema mostra uma mensagem genérica, sem indicar qual dado errou.

### CU-10: Calcular Frete
- **Atores envolvidos:** **Serviço de Frete**.
- **Pré-condições:** Carrinho com itens e CEP de entrega informado.
- **Fluxo principal:**
  1. O sistema envia o CEP e o peso e as dimensões dos itens ao **Serviço de Frete**.
  2. O **Serviço de Frete** devolve as opções de entrega.
  3. O sistema exibe o valor e o prazo de cada opção.
  4. O **Cliente** escolhe uma opção.
- **Fluxos alternativos ou de exceção:**
  - **E1**: No passo 1, se o CEP é inválido,
    - então o sistema pede a correção.
  - **E2**: No passo 1, se o **Serviço de Frete** está indisponível,
    - então o sistema avisa e a finalização do pedido não avança.

### CU-11: Processar Pagamento
- **Atores envolvidos:** **Gateway de Pagamento**.
- **Pré-condições:** Pedido criado, com a forma de pagamento escolhida.
- **Fluxo principal:**
  1. O sistema envia a cobrança ao **Gateway de Pagamento**, sem guardar dados de cartão.
  2. O **Gateway de Pagamento** aprova o pagamento.
  3. O sistema registra o resultado.
  4. O sistema passa o pedido para pago e dá baixa no estoque reservado.
- **Fluxos alternativos ou de exceção:**
  - **A1**: No passo 2, se o pagamento é por Pix ou boleto,
    - então o pedido aguarda o aviso de confirmação do **Gateway de Pagamento**.
  - **E1**: No passo 2, se o pagamento é recusado,
    - então o **Cliente** pode tentar outra forma.
  - **E2**: No passo 1, se o **Gateway de Pagamento** está indisponível,
    - então o pedido fica aguardando verificação, sem cobrança duplicada.
  - **E3**: No passo 2, se o prazo de pagamento expira,
    - então o sistema cancela o pedido e devolve o estoque reservado.

### CU-12: Cadastrar-se
- **Atores envolvidos:** **Visitante**.
- **Pré-condições:** Nenhuma.
- **Fluxo principal:**
  1. O **Visitante** informa nome, CPF, e-mail, telefone e senha.
  2. O sistema valida os dados.
  3. O sistema cria a conta de cliente e autentica o usuário.
- **Fluxos alternativos ou de exceção:**
  - **E1**: No passo 2, se o CPF ou o e-mail já estão cadastrados,
    - então o sistema sugere entrar na conta existente.
  - **E2**: No passo 2, se os dados são inválidos,
    - então o sistema mostra o erro junto aos campos.

### CU-13: Notificar Cliente por E-mail
- **Atores envolvidos:** **Serviço de E-mail**.
- **Pré-condições:** Pedido com mudança de status registrada.
- **Fluxo principal:**
  1. O sistema monta a mensagem com o número do pedido e o novo status.
  2. O sistema envia a mensagem ao **Serviço de E-mail**.
  3. O **Serviço de E-mail** entrega a mensagem ao **Cliente**.
  4. O sistema registra o envio.
- **Fluxos alternativos ou de exceção:**
  - **E1**: No passo 2, se o envio falha,
    - então o sistema tenta de novo e, se continuar falhando, registra a falha sem desfazer a mudança de status.

---

## 8. Arquitetura e Componentes

### 8.1 Visão geral (Diagrama de Componentes **geral**)

Aplicação web em camadas, com serviços externos acessados por **adaptadores** (RNF-06): componentes que "traduzem" entre o sistema e cada serviço de fora (pagamento, frete, e-mail), de modo que trocar um serviço altera só o adaptador.

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
| Interface da Loja | Telas de catálogo, carrinho, finalização do pedido e pedidos | Nenhuma | `IApi` |
| Painel Administrativo | Telas de produtos, estoque e pedidos | Nenhuma | `IApi` |
| API | Entrada HTTPS; valida sessão e perfil; encaminha aos módulos (no detalhe do Backend, é a camada de API: controladores e segurança) | `IApi` | `IUsuarios`, `ICatalogo`, `ICarrinho`, `IPedidos`, `IEstoque` |
| Usuários | Cadastro, login e perfis | `IUsuarios` | `IDados` (via Repositórios) |
| Catálogo | Produtos, categorias, marcas, filtros e busca | `ICatalogo` | `IDados` (via Repositórios), `IEstoque` |
| Carrinho | Itens e subtotal; obtém preço e disponibilidade pelo Catálogo | `ICarrinho` | `ICatalogo` |
| Pedidos | Finalização do pedido, status e cancelamento | `IPedidos` | `ICarrinho`, `IEstoque`, `IFrete`, `IPagamento`, `INotificacao`, `IDados` (via Repositórios) |
| Estoque | Disponibilidade, reserva, baixa e ajustes | `IEstoque` | `IDados` (via Repositórios) |
| Notificações | Monta e envia e-mails de status | `INotificacao` | `IEmail` |
| Adaptador de Pagamento | Fala com o gateway de pagamento | `IPagamento`, `IConfirmacao` (webhook) | `IGateway` (externo), `IPedidos` (repassa a confirmação do pagamento) |
| Adaptador de Frete | Fala com o serviço de frete | `IFrete` | `IServicoFrete` (externo) |
| Adaptador de E-mail | Fala com o serviço de e-mail | `IEmail` | `IServicoEmail` (externo) |
| Banco de Dados | Persistência | `IDados` | Nenhuma |

**Pacotes e dependências:** o diagrama geral mostra o pacote **Dados** (Banco de Dados) e setas tracejadas de dependência entre os pacotes: Frontend → Backend → Integrações → Serviços Externos.

**Portas:** a **API** tem uma porta HTTPS (entrada pública); o **Adaptador de Pagamento** tem uma porta de webhook (confirmação de Pix/boleto).

### 8.3 Diagramas **micro** (componentes internos)

São dois diagramas micro: um do **Frontend** e um do **Backend**.

**Micro 1: Frontend**

| Componente interno | Responsabilidade |
|---|---|
| Catálogo e Busca | Listagem, filtros, busca e detalhes (CU-01) |
| Carrinho e Finalização do Pedido | Carrinho, frete, pagamento e confirmação (CU-02 e CU-03) |
| Meus Pedidos | Acompanhar e cancelar pedidos (CU-04 e CU-08) |
| Login e Cadastro | Cadastro e autenticação do cliente (CU-09 e CU-12) |
| Login do Administrador | Autenticação do administrador para entrar no painel (CU-09) |
| Gestão de Produtos / Estoque / Pedidos | Telas do administrador (CU-05 e CU-06) |
| Comunicação com a API | Sessão e requisições HTTPS; **requer** `IApi` |

(Catálogo e Busca, Carrinho e Finalização do Pedido, Meus Pedidos e Login e Cadastro pertencem à Interface da Loja; o Login do Administrador e as três Gestões, ao Painel Administrativo.)

**Micro 2: Backend** (três pacotes, ligados por setas tracejadas de dependência)

| Pacote | Componentes internos | Interfaces |
|---|---|---|
| Camada de API | Controlador da Loja, Controlador Administrativo, Segurança e Perfis (RBAC, usado pelos dois controladores) | **fornece** `IApi` (portas no topo) |
| Módulos de negócio | Usuários, Estoque, Catálogo, Carrinho, Pedidos, Notificações | cada um **fornece** a sua interface; Pedidos **requer** `IPagamento` e `IFrete`; Notificações **requer** `IEmail` |
| Acesso a dados | Repositórios | **requer** `IDados` |

As dependências entre os módulos aparecem só no diagrama geral.

---|---|---|
| API | Controlador da Loja, Controlador Administrativo, Segurança e Perfis (RBAC) | **fornece** `IApi` |
| Módulos de negócio | Usuários, Catálogo, Estoque, Carrinho, Pedidos, Notificações | **requerem** `IFrete`, `IPagamento`, `IEmail` |
| Acesso a dados | Repositórios | **requer** `IDados` |

---


---

## Apêndice: Rastreabilidade

| Artefato | Requisitos |
|---|---|
| CU-01 Navegar no Catálogo | RF-01, RF-02, RN-01 |
| CU-02 Gerenciar Carrinho | RF-04, RN-02 |
| CU-03 Finalizar Pedido | RF-05, RF-06, RF-07, RN-03, RN-04 |
| CU-04 Consultar Pedidos | RF-08, RN-05 |
| CU-05 Gerenciar Produtos e Estoque | RF-10, RF-11 |
| CU-06 Gerenciar Pedidos | RF-12 |
| CU-07 Filtrar / Buscar Produtos | RF-01 |
| CU-08 Cancelar Pedido | RF-08, RN-05 |
| CU-09 Autenticar Usuário | RF-03 |
| CU-10 Calcular Frete | RF-05 |
| CU-11 Processar Pagamento | RF-07 |
| CU-12 Cadastrar-se | RF-03 |
| CU-13 Notificar Cliente por E-mail | RF-09, RN-06 |
| Componentes (geral e micros) | RNF-05, RNF-06 e todos os RF |
