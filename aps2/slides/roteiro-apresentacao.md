# Roteiro de Apresentação — APS2

Site de Loja de Eletrônicos (E-commerce) · Grupo 4 · Modelagem de Software (UFG/INF)

Os slides seguem a ordem do documento. Cada pessoa fala uma vez, em um bloco seguido, na ordem dos nomes da capa. Os números de página são os impressos no PDF do documento.

Para todos: o **Apêndice B (Glossário, Tabela 27, p. 24)** explica os termos técnicos (webhook, adaptador, API, Repositório, porta).

| # | Quem | Seções do documento | Slides |
|---|---|---|---|
| 1 | Carlos | Cap. 1, 2.1 e 2.2 | 1.1 e 1.2 · 1.3 · Atores · Diagrama · 2 ampliações · Como ler · Decisões de modelagem |
| 2 | Gabriel | 2.3 e CU-01 a CU-06 | Os 4 tipos · Relacionamentos no diagrama · CU-01 e 02 · CU-03 · CU-04 a 06 |
| 3 | Kayque | CU-07 a CU-13 | CU-07 a 09 · CU-10 e 11 · CU-12 e 13 |
| 4 | Luiz | Cap. 3 até 3.3 | Arquitetura · Diagrama geral · 2 ampliações · Leitura e percurso · Decisões de arquitetura · Dicionário · Interfaces e dependências |
| 5 | Yury | 3.4, 3.5, Cap. 4 e Apêndices | Micro do Frontend · Frontend por dentro · Micro do Backend · Backend por dentro · Ferramenta e símbolos · Apêndices · Encerramento |

---

## 1. Carlos — Visão geral, diagrama e decisões

**Falar**
- Tema: loja virtual de eletrônicos para o consumidor final; catálogo com filtros por categoria, marca e preço; carrinho; finalização; pagamento e frete por serviços externos; avisos por e-mail; área administrativa.
- Escopo: o que está dentro e o que ficou fora (cupons, avaliações, troca e devolução, nota fiscal, marketplace, aplicativo nativo).
- Premissas: web app responsivo, login só na finalização, um perfil administrativo, cartão, Pix ou boleto pelo gateway, e-mail como único canal, cancelamento automático após o prazo.
- Os 6 atores (3 principais e 3 secundários) e os 13 casos (7 principais em azul e 6 de apoio em amarelo).
- Como ler o diagrama: fronteira, atores à esquerda e à direita, linhas contínuas, notas com a condição do «extend».
- As 6 decisões de modelagem.

**Estudar**

| Seção | Página | Itens |
|---|---|---|
| 1.1, 1.2, 1.3 | 3 e 4 | descrição, escopo, premissas |
| 2 (introdução) e 2.1 Diagrama | 4 e 5 | Tabela 1 (atores), Figura 1, leitura e cores |
| 2.2 Decisões de Modelagem | 5 e 6 | as seis decisões |

## 2. Gabriel — Relacionamentos e CU-01 a CU-06

**Falar**
- Os 4 tipos: associação, generalização, «include» (obrigatório) e «extend» (condicional), com um exemplo de cada.
- Os 8 «include» e os 2 «extend» do diagrama, e o sentido das setas.
- CU-01 Navegar e CU-02 Carrinho.
- CU-03 Finalizar Pedido como exemplo do modelo de especificação: atores, pré-condição, os 9 passos, os fluxos A1, E1, E2 e E3.
- CU-04 Consultar, CU-05 Produtos e Estoque, CU-06 Gerenciar Pedidos (status, rastreio, aviso por e-mail).

**Estudar**

| Seção | Página | Itens |
|---|---|---|
| 2.3 Relacionamentos | 6 e 7 | Tabela 2 (tipos) e Tabela 3 (os do diagrama) |
| CU-01, CU-02, CU-03 | 7 e 8 | Tabelas 4, 5 e 6 |
| CU-04, CU-05, CU-06 | 9 e 10 | Tabelas 7, 8 e 9 |

## 3. Kayque — CU-07 a CU-13

**Falar**
- CU-07 Filtrar / Buscar («extend» do CU-01).
- CU-08 Cancelar Pedido («extend» do CU-04): só até o envio, estorno se já pago, aviso por e-mail.
- CU-09 Autenticar Usuário: incluído por quatro casos; mensagem de erro genérica.
- CU-10 Calcular Frete e CU-11 Processar Pagamento: os casos com sistemas externos; Pix e boleto aguardam confirmação.
- CU-12 Cadastrar-se e CU-13 Notificar por E-mail (falha no envio não desfaz o status).

**Estudar**

| Seção | Página | Itens |
|---|---|---|
| CU-07, CU-08, CU-09 | 11 e 12 | Tabelas 10, 11 e 12 |
| CU-10, CU-11 | 12 e 13 | Tabelas 13 e 14 |
| CU-12, CU-13 | 14 | Tabelas 15 e 16 |

## 4. Luiz — Componentes: geral, decisões e dicionário

**Falar**
- Arquitetura em camadas com cinco pacotes; adaptadores para os serviços externos; geral e dois micros.
- Diagrama geral: pirulito, soquete, porta, dependência e as cores dos pacotes.
- Percurso de Finalizar Pedido pelo diagrama, de `IApi` até `IDados`, passando pelo webhook (`IConfirmacao` e `IPedidos`).
- As decisões de arquitetura (API como entrada única, adaptadores, Repositórios, webhook, só dois micros, pacote versus componente).
- Dicionário: o que cada componente fornece e requer, e por que cada dependência existe.

**Estudar**

| Seção | Página | Itens |
|---|---|---|
| 3 (introdução) e 3.1 Diagrama | 14 e 15 | Figura 2, cores e leitura |
| 3.2 Decisões de Arquitetura | 15 e 16 | as sete decisões |
| 3.3 Dicionário de Componentes | 16 a 18 | Tabelas 17, 18 e 19 |

## 5. Yury — Micros, ferramenta, apêndices e encerramento

**Falar**
- Frontend: telas de cada aplicação e os casos de uso que atendem; Comunicação com a API como único ponto que requer `IApi`; telas independentes.
- Backend: três camadas, módulos usando uns aos outros, portas da borda (duas `IApi` no topo; `IPagamento`, `IFrete`, `IEmail`, `IDados` e a entrada `IPedidos` na base).
- Ferramenta: draw.io, tabelas de símbolos, cores só agrupam.
- Apêndices: 12 RF, 6 RNF, 6 RN e glossário; rastreabilidade (RF-06 vira o CU-03, atendido pela tela Carrinho e Finalização e pelo módulo Pedidos).
- Encerramento e perguntas.

**Estudar**

| Seção | Página | Itens |
|---|---|---|
| 3.4 Detalhe Interno: Frontend | 18 e 19 | Figura 3 e Tabela 20 |
| 3.5 Detalhe Interno: Backend | 20 e 21 | Figura 4 e Tabela 21 |
| 4 Ferramenta e Lista de Símbolos | 21 e 22 | Tabelas 22 e 23 |
| Apêndices A e B | 23 e 24 | Tabelas 24 a 27 |
