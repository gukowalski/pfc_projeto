# Tuscan Shop — Contexto e Roadmap do Projeto

## Visão Geral

**Tuscan Shop** é um e-commerce temático voltado para produtos inspirados no universo do jogo Counter-Strike 2 (CS2), desenvolvido como Trabalho de Conclusão de Curso (TCC) do curso de Engenharia de Software da Universidade de Mogi das Cruzes (UMC), turma 2026.

**Integrantes:**
- Gabriel de Alvarenga
- Gustavo Eiji Kowalski Hatada
- Lucas Margenet de Oliveira
- Victor Teixeira Novaes

**Orientadora:** Prof. Viviane Guimarães Ribeiro

---

## Foco do Projeto

O objetivo principal **não é** construir um e-commerce completo, mas sim demonstrar uma **arquitetura de microsserviços bem definida, coesa e testada**. O valor do trabalho está nas decisões arquiteturais e na qualidade da implementação, não na quantidade de funcionalidades.

---

## Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Serviços | Spring Boot 3 (Java) |
| API Gateway | Spring Cloud Gateway |
| Comunicação síncrona | Spring Cloud OpenFeign |
| Mensageria assíncrona | RabbitMQ + Spring AMQP |
| Banco de dados | PostgreSQL (um por serviço) |
| Autenticação | Spring Security + JWT |
| Cache | Redis + Spring Cache (opcional, mês 3) |
| Testes unitários | JUnit 5 + Mockito |
| Testes de integração | Testcontainers |
| Documentação de API | Springdoc OpenAPI (Swagger) |
| Containers | Docker + Docker Compose |
| Orquestração | Kubernetes (Minikube) + Helm Charts |

---

## Arquitetura de Microsserviços

### Serviços definidos

```
tuscan-shop/
├── api-gateway/           # Roteamento + validação JWT
├── user-service/          # Cadastro, login, perfis
├── product-service/       # Catálogo, categorias, estoque
├── order-service/         # Carrinho, pedidos, status
├── payment-service/       # Integração com gateway de pagamento
├── notification-service/  # Envio de e-mails por evento
└── docker-compose.yml
```

### Regra fundamental
Cada serviço possui **seu próprio banco de dados PostgreSQL**. Nenhum serviço acessa diretamente o banco de outro. Essa é a separação que diferencia microsserviços reais de um monólito disfarçado.

### Tipos de comunicação

- **Síncrona (OpenFeign):** quando a resposta é necessária imediatamente. Ex: `order-service` consultando `product-service` para verificar estoque antes de criar um pedido.
- **Assíncrona (RabbitMQ):** quando não precisa de resposta imediata. Ex: `order-service` publica evento de pedido confirmado → `payment-service` e `notification-service` consomem o evento.

---

## Fluxo Principal (Happy Path)

```
Usuário se cadastra (user-service)
    → Faz login e recebe JWT (user-service)
    → Navega no catálogo (product-service)
    → Adiciona produto ao carrinho (order-service)
    → Finaliza pedido (order-service → product-service via Feign)
    → Realiza pagamento (payment-service)
    → Recebe e-mail de confirmação (notification-service via RabbitMQ)
```

---

## Requisitos Funcionais Mantidos

Os requisitos abaixo foram selecionados por serem essenciais ao fluxo e adequados ao escopo de microsserviços:

| Módulo | Requisitos mantidos |
|---|---|
| Usuários | RF01, RF02, RF03, RF04, RF05, RF07 |
| Catálogo | RF08, RF09, RF10, RF11, RF13 |
| Carrinho e Pedidos | RF16, RF17, RF18, RF19, RF20, RF21, RF22 |
| Pagamento | RF23, RF24, RF25, RF27 |
| Logística | RF28, RF29, RF30, RF32 |
| Backoffice | RF33, RF34, RF35, RF36, RF37 |
| Avaliações | RF40, RF41 |

## Requisitos Cortados ou Simplificados

| Requisito | Motivo |
|---|---|
| RF14 — Visualizador 3D interativo | Alta complexidade, baixo valor arquitetural |
| RF12 — Recomendações personalizadas | Exige ML/filtragem colaborativa, fora do escopo |
| RF26 — NF-e fiscal | Integração fiscal burocrática e cara |
| RF31 — Notificações por SMS | API paga, substituído por e-mail |
| RF38 — Relatórios em PDF | Apenas CSV ou exibição em tela |
| RF43 — Q&A nos produtos | Baixa prioridade |
| RF28 — API dos Correios | Substituída por frete fixo simulado por CEP |

---

## Requisitos Não Funcionais Chave

| ID | Descrição |
|---|---|
| RNF01 | Página inicial carrega em até 3 segundos |
| RNF03 | HTTPS com TLS 1.2+ |
| RNF04 | Senhas com bcrypt ou Argon2 |
| RNF05 | Proteção contra XSS, SQL Injection e CSRF |
| RNF10 | Disponibilidade mínima de 99,5% ao mês |
| RNF11 | Cobertura de testes unitários mínima de 70% |
| RNF12 | Escalabilidade horizontal (demonstrada com HPA no Kubernetes) |

---

## Roadmap — 4 Meses

### Mês 1 — Estrutura base e fundação

**Objetivo:** Ter todos os serviços criados, Docker Compose funcionando e o `user-service` completo com autenticação JWT.

- [ ] Criar monorepo no GitHub com os 5 projetos Spring Boot via start.spring.io
- [ ] Docker Compose com PostgreSQL (um por serviço) e RabbitMQ
- [ ] Implementar `user-service` — cadastro, login, JWT
- [ ] Configurar `api-gateway` com Spring Cloud Gateway + validação JWT
- [ ] Esqueleto dos demais serviços com health check endpoint (`/actuator/health`)
- [ ] Swagger (Springdoc OpenAPI) configurado em todos os serviços

> **Dica:** Começa pelo `user-service` — ele desbloqueia a autenticação para tudo que vem depois.

---

### Mês 2 — Fluxo principal e testes unitários

**Objetivo:** Happy path completo funcionando de ponta a ponta, com testes unitários cobrindo os casos principais.

- [ ] Implementar `product-service` — CRUD de produtos, categorias, estoque
- [ ] Implementar `order-service` — carrinho, criação de pedido, controle de status
- [ ] Implementar `payment-service` — integração com gateway (Mercado Pago ou Stripe)
- [ ] Implementar `notification-service` — envio de e-mail por evento
- [ ] Comunicação síncrona com OpenFeign (`order-service` → `product-service`)
- [ ] Testes unitários com JUnit 5 + Mockito — mínimo 70% de cobertura por serviço

> **Dica:** Ao fim do mês 2, o demo de: usuário cria conta → adiciona produto → faz pedido → paga → recebe e-mail deve funcionar de ponta a ponta.

---

### Mês 3 — Mensageria, integração e qualidade

**Objetivo:** Introduzir comunicação assíncrona, testes de integração robustos e cache opcional.

- [ ] Comunicação assíncrona com RabbitMQ — `order-service` publica evento, `payment-service` e `notification-service` consomem
- [ ] Testes de integração com Testcontainers — PostgreSQL e RabbitMQ reais durante os testes
- [ ] Backoffice básico no `product-service` — painel de estoque com alertas de baixo estoque
- [ ] Redis no `product-service` para cache de catálogo — `@Cacheable` e `@CacheEvict` (opcional)
- [ ] Logs centralizados e health checks padronizados em todos os serviços
- [ ] Revisão geral — validar que nenhum serviço acessa o banco de outro

> **Dica:** Testcontainers é o diferencial mais fácil de implementar e que mais impressiona na banca. Sobe banco e fila reais nos testes, sem mock.

---

### Mês 4 — Kubernetes, documentação e apresentação

**Objetivo:** Orquestrar tudo no Kubernetes, finalizar documentação e preparar a apresentação para a banca.

- [ ] Migrar Docker Compose para manifests Kubernetes (Minikube local)
- [ ] Helm Charts para organizar os deployments de cada serviço
- [ ] Horizontal Pod Autoscaler (HPA) no `product-service` — demonstra escalabilidade (RNF12)
- [ ] Diagrama de arquitetura final — serviços, comunicação, bancos, mensageria
- [ ] Documentação das decisões arquiteturais — justificar cada fronteira entre serviços
- [ ] Preparar demo do fluxo ponta a ponta para a banca

> **Dica:** Na apresentação, foca nas decisões: por que RabbitMQ aqui, por que Feign ali, o que acontece se o `payment-service` cair.

---

## Dependências por Serviço (start.spring.io)

### `user-service` e `product-service`
- Spring Web
- Spring Data JPA
- PostgreSQL Driver
- Spring Security
- Validation
- Lombok

### `order-service`
- Tudo acima
- Spring AMQP (RabbitMQ)
- OpenFeign

### `payment-service` e `notification-service`
- Spring Web
- Spring AMQP
- Lombok

### `api-gateway`
- Spring Cloud Gateway
- Spring Security (validação JWT centralizada)

---

## Diferenciais para a Banca

| Diferencial | Por que impressiona |
|---|---|
| Banco separado por serviço | Demonstra entendimento real de microsserviços |
| Dois tipos de comunicação (Feign + RabbitMQ) | Mostra quando usar síncrono vs assíncrono |
| Testcontainers nos testes de integração | Testa com infraestrutura real, não mocks |
| Kubernetes + HPA | Liga diretamente ao RNF12 do documento |
| Justificativa das decisões | Maturidade de engenharia, não só código funcionando |

---

## Próximas Etapas Imediatas

1. Criar o monorepo no GitHub
2. Gerar os 5 projetos no [start.spring.io](https://start.spring.io)
3. Configurar o `docker-compose.yml` com PostgreSQL e RabbitMQ
4. Implementar o `user-service` com cadastro, login e JWT
