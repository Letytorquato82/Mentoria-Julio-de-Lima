# 📋 Task Management API  
**Demonstração de Arquitetura, Testes e Qualidade de Software em APIs REST e GraphQL**

> Uma API completa de gestão de tarefas em sprint, desenvolvida para ensinar boas práticas de qualidade, arquitetura em camadas, testes automatizados e integração contínua. O projeto implementa um CRUD funcional exposto via REST e GraphQL, com validação rigorosa de dados, testes abrangentes (Jest, Supertest, Postman/Newman) e pipeline CI/CD automatizado no GitHub Actions.

---

## 🎯 O que é este projeto?

Este é um projeto de portfólio educacional que demonstra o ciclo completo de desenvolvimento de uma API de qualidade:

- ✅ **Arquitetura bem estruturada** — camadas separadas (Controllers → Services → Data)
- ✅ **Duas interfaces de API** — REST e GraphQL compartilhando a mesma lógica
- ✅ **Validação rigorosa** — JSON Schema com Ajv para garantir integridade de dados
- ✅ **Testes abrangentes** — testes unitários, integração e coleções Postman
- ✅ **Especificação viva** — casos de teste em Gherkin/BDD para documentação clara
- ✅ **CI/CD automatizado** — pipeline completo no GitHub Actions
- ✅ **Qualidade comprovada** — relatórios de testes, cobertura e cargas

Ideal para quem está aprendendo **QA/SDET**, **engenharia de software**, **testes de API** ou buscando referências de **boas práticas em arquitetura backend**.

---

## 🏗️ Arquitetura do Projeto

O projeto segue uma **arquitetura em camadas (layered architecture)**, organizada de forma a separar claramente as responsabilidades entre entrada de dados, lógica de negócio e persistência. Essa abordagem facilita a manutenção, os testes e a evolução do sistema.

### 1. Camada de Consumo (Cliente)

O ponto de entrada da aplicação pode ser qualquer cliente HTTP — como um **frontend web**, o **Postman** (para testes manuais) ou qualquer outro consumidor de API. Essa camada é responsável apenas por enviar requisições e consumir as respostas.

### 2. Camada de API

O projeto expõe **duas interfaces de comunicação** em paralelo:

- **Express API REST** — endpoints tradicionais RESTful;
- **GraphQL API** — consultas e mutações via GraphQL.

Ambas convivem no mesmo backend, oferecendo flexibilidade para diferentes tipos de consumidores, mas convergem para o mesmo núcleo de processamento.

### 3. Controllers

Independentemente da interface utilizada (REST ou GraphQL), as requisições são direcionadas para os **Controllers**, que atuam como intermediários: recebem os dados de entrada, delegam o processamento para a camada de serviços e formatam as respostas.

### 4. Services

Os **Services** concentram a orquestração das regras da aplicação, atuando como uma ponte entre os Controllers e as camadas mais internas do sistema. É aqui que o fluxo de negócio é coordenado, distribuindo tarefas para três frentes:

- **Data Layer / File Storage** — responsável pelo acesso e persistência de dados, seja em banco de dados ou armazenamento de arquivos;
- **Validation Layer** — garante que os dados estejam corretos antes de serem processados, utilizando **JSON Schema / Ajv** para validação estrutural;
- **Business Rules** — aplica as regras de negócio específicas do domínio da aplicação.

### 5. Validação com JSON Schema / Ajv

A camada de validação utiliza **Ajv**, uma biblioteca de validação baseada em **JSON Schema**, garantindo que os dados que trafegam pelo sistema estejam sempre em conformidade com os formatos esperados, prevenindo erros e inconsistências antes que cheguem à lógica de negócio ou à persistência.

### Fluxo resumido

```
Cliente → API (REST ou GraphQL) → Controllers → Services → { Dados, Validação, Regras de Negócio }
```

Essa estrutura promove **baixo acoplamento** entre a interface de comunicação e a lógica interna, permitindo que múltiplas formas de acesso (REST/GraphQL) compartilhem a mesma base de regras e validações, sem duplicação de código.

---

##  Casos de Teste (Gherkin/BDD)

Os principais fluxos da API são documentados em formato Gherkin, permitindo especificar o comportamento esperado em linguagem legível para todos os stakeholders (devs, QAs, Product Owners).

### Feature: Gerenciar Tarefas - REST API

```gherkin
Feature: Task Management REST API
  Como usuário da API
  Quero gerenciar tarefas (criar, listar, buscar, atualizar e deletar)
  Para organizar e acompanhar o progresso de trabalhos em sprint

  Background:
    Given a aplicação está rodando em "http://localhost:3000"
    And o endpoint "/api/tasks" está disponível

  # ===== CREATE TASK =====
  Scenario: Criar uma tarefa com sucesso
    When faço uma requisição POST para "/api/tasks" com o corpo:
      """json
      {
        "title": "Implementar autenticação",
        "description": "Adicionar JWT à API",
        "status": "pending"
      }
      """
    Then o status da resposta deve ser 201
    And a resposta deve conter um "id"
    And o campo "title" deve ser "Implementar autenticação"
    And o campo "status" deve ser "pending"

  Scenario: Criar tarefa sem título retorna erro 400
    When faço uma requisição POST para "/api/tasks" com o corpo:
      """json
      {
        "description": "Descrição sem título"
      }
      """
    Then o status da resposta deve ser 400
    And a resposta deve conter mensagem de erro indicando que "title" é obrigatório

  Scenario: Criar tarefa com status inválido retorna erro 400
    When faço uma requisição POST para "/api/tasks" com o corpo:
      """json
      {
        "title": "Tarefa teste",
        "status": "invalid_status"
      }
      """
    Then o status da resposta deve ser 400
    And a resposta deve conter mensagem de erro sobre "status" inválido

  Scenario: Criar tarefa com description opcionalmente vazia
    When faço uma requisição POST para "/api/tasks" com o corpo:
      """json
      {
        "title": "Tarefa simples"
      }
      """
    Then o status da resposta deve ser 201
    And o campo "description" deve ser vazio ou nulo

  # ===== READ TASKS =====
  Scenario: Listar todas as tarefas com sucesso
    Given que existem 3 tarefas cadastradas
    When faço uma requisição GET para "/api/tasks"
    Then o status da resposta deve ser 200
    And a resposta deve ser um array
    And o array deve conter 3 itens

  Scenario: Listar tarefas quando não há nenhuma
    Given não há tarefas cadastradas
    When faço uma requisição GET para "/api/tasks"
    Then o status da resposta deve ser 200
    And a resposta deve ser um array vazio

  Scenario: Buscar uma tarefa específica por ID
    Given que existe uma tarefa com ID "1" e título "Tarefa teste"
    When faço uma requisição GET para "/api/tasks/1"
    Then o status da resposta deve ser 200
    And o campo "id" deve ser "1"
    And o campo "title" deve ser "Tarefa teste"

  Scenario: Buscar tarefa com ID inexistente retorna 404
    When faço uma requisição GET para "/api/tasks/9999"
    Then o status da resposta deve ser 404
    And a resposta deve conter mensagem indicando que a tarefa não foi encontrada

  # ===== UPDATE TASK (PUT - completa) =====
  Scenario: Atualizar tarefa completa com sucesso
    Given que existe uma tarefa com ID "1"
    When faço uma requisição PUT para "/api/tasks/1" com o corpo:
      """json
      {
        "title": "Tarefa atualizada",
        "description": "Nova descrição",
        "status": "in-progress"
      }
      """
    Then o status da resposta deve ser 200
    And o campo "title" deve ser "Tarefa atualizada"
    And o campo "status" deve ser "in-progress"

  Scenario: Atualizar tarefa sem título retorna erro 400
    Given que existe uma tarefa com ID "1"
    When faço uma requisição PUT para "/api/tasks/1" com o corpo:
      """json
      {
        "description": "Descrição sem título"
      }
      """
    Then o status da resposta deve ser 400
    And a resposta deve conter mensagem de erro

  Scenario: Atualizar tarefa inexistente retorna 404
    When faço uma requisição PUT para "/api/tasks/9999" com o corpo:
      """json
      {
        "title": "Título qualquer",
        "status": "pending"
      }
      """
    Then o status da resposta deve ser 404

  # ===== UPDATE STATUS (PATCH - parcial) =====
  Scenario: Atualizar apenas status da tarefa com sucesso
    Given que existe uma tarefa com ID "1" com status "pending"
    When faço uma requisição PATCH para "/api/tasks/1/status" com o corpo:
      """json
      {
        "status": "in-progress"
      }
      """
    Then o status da resposta deve ser 200
    And o campo "status" deve ser "in-progress"
    And os outros campos da tarefa devem permanecer inalterados

  Scenario: Atualizar para status inválido retorna erro 400
    Given que existe uma tarefa com ID "1"
    When faço uma requisição PATCH para "/api/tasks/1/status" com o corpo:
      """json
      {
        "status": "invalid"
      }
      """
    Then o status da resposta deve ser 400

  Scenario: Workflow de status válido: pending → in-progress → done
    Given que existe uma tarefa com ID "1" com status "pending"
    When faço uma requisição PATCH para "/api/tasks/1/status" com status "in-progress"
    Then o status da resposta deve ser 200
    And o campo "status" deve ser "in-progress"
    
    When faço uma requisição PATCH para "/api/tasks/1/status" com status "done"
    Then o status da resposta deve ser 200
    And o campo "status" deve ser "done"

  # ===== DELETE TASK =====
  Scenario: Deletar uma tarefa com sucesso
    Given que existe uma tarefa com ID "1"
    When faço uma requisição DELETE para "/api/tasks/1"
    Then o status da resposta deve ser 204

  Scenario: Deletar tarefa inexistente retorna 404
    When faço uma requisição DELETE para "/api/tasks/9999"
    Then o status da resposta deve ser 404

  Scenario: Tarefa deletada não pode ser recuperada
    Given que existe uma tarefa com ID "1"
    When faço uma requisição DELETE para "/api/tasks/1"
    And faço uma requisição GET para "/api/tasks/1"
    Then o status da resposta deve ser 404

  # ===== VALIDATION =====
  Scenario Outline: Validar tipos de dados na criação
    When faço uma requisição POST para "/api/tasks" com o corpo:
      """json
      {
        "title": <title>,
        "description": <description>,
        "status": <status>
      }
      """
    Then o status da resposta deve ser <expectedStatus>

    Examples:
      | title           | description    | status       | expectedStatus |
      | "Tarefa 1"      | "Desc"         | "pending"    | 201            |
      | "Tarefa 2"      | null           | "pending"    | 201            |
      | null            | "Desc"         | "pending"    | 400            |
      | 123             | "Desc"         | "pending"    | 400            |
      | "Tarefa 3"      | "Desc"         | "unknown"    | 400            |
```

### Feature: Gerenciar Tarefas - GraphQL API

```gherkin
Feature: Task Management GraphQL API
  Como desenvolvedor frontend
  Quero consultar e modificar tarefas via GraphQL
  Para obter apenas os campos necessários em uma única requisição

  Background:
    Given a aplicação está rodando em "http://localhost:3000"
    And o endpoint GraphQL "/graphql" está disponível

  Scenario: Consultar todas as tarefas via GraphQL
    When faço uma requisição GraphQL com a query:
      """graphql
      query {
        tasks {
          id
          title
          description
          status
        }
      }
      """
    Then o status da resposta deve ser 200
    And a resposta deve conter um array de tarefas
    And cada tarefa deve ter "id", "title" e "status"

  Scenario: Criar tarefa via GraphQL mutation
    When faço uma requisição GraphQL com a mutation:
      """graphql
      mutation {
        createTask(input: {
          title: "Nova tarefa GraphQL",
          description: "Teste de mutation",
          status: "pending"
        }) {
          id
          title
          status
        }
      }
      """
    Then o status da resposta deve ser 200
    And o campo "data.createTask.title" deve ser "Nova tarefa GraphQL"
    And o campo "data.createTask.id" deve estar presente

  Scenario: Atualizar tarefa via GraphQL mutation
    Given que existe uma tarefa com ID "1" via GraphQL
    When faço uma requisição GraphQL com a mutation:
      """graphql
      mutation {
        updateTask(id: "1", input: {
          title: "Tarefa atualizada via GraphQL",
          status: "in-progress"
        }) {
          id
          title
          status
        }
      }
      """
    Then o status da resposta deve ser 200
    And o campo "data.updateTask.status" deve ser "in-progress"

  Scenario: Deletar tarefa via GraphQL mutation
    Given que existe uma tarefa com ID "1" via GraphQL
    When faço uma requisição GraphQL com a mutation:
      """graphql
      mutation {
        deleteTask(id: "1") {
          success
          message
        }
      }
      """
    Then o status da resposta deve ser 200
    And o campo "data.deleteTask.success" deve ser true

  Scenario: Consultar apenas campos específicos via GraphQL
    Given que existem tarefas cadastradas
    When faço uma requisição GraphQL com a query:
      """graphql
      query {
        tasks {
          id
          title
        }
      }
      """
    Then o status da resposta deve ser 200
    And cada tarefa deve conter apenas "id" e "title" (sem "description" ou "status")

  Scenario: GraphQL retorna erro para query inválida
    When faço uma requisição GraphQL com a query:
      """graphql
      query {
        tasks {
          invalidField
        }
      }
      """
    Then o status da resposta deve ser 400
    And a resposta deve conter mensagem de erro indicando campo inválido
```

### Feature: Validação de Dados (JSON Schema)

```gherkin
Feature: Validação de Entrada (JSON Schema / Ajv)
  Como sistema
  Quero validar todos os dados de entrada
  Para garantir que apenas dados válidos cheguem à lógica de negócio

  Scenario: Payload com campos obrigatórios presentes é válido
    When valido o payload:
      """json
      {
        "title": "Tarefa completa",
        "description": "Descrição aqui",
        "status": "pending"
      }
      """
    Then a validação deve passar

  Scenario: Payload sem campo obrigatório "title" falha
    When valido o payload:
      """json
      {
        "description": "Sem título",
        "status": "pending"
      }
      """
    Then a validação deve falhar
    And a mensagem deve indicar que "title" é obrigatório

  Scenario: Campo "title" deve ser string
    When valido o payload:
      """json
      {
        "title": 123,
        "status": "pending"
      }
      """
    Then a validação deve falhar
    And a mensagem deve indicar que "title" deve ser string

  Scenario: Campo "status" só aceita valores permitidos
    When valido o payload com "status" = "pending"
    Then a validação deve passar
    
    When valido o payload com "status" = "in-progress"
    Then a validação deve passar
    
    When valido o payload com "status" = "done"
    Then a validação deve passar
    
    When valido o payload com "status" = "rejected"
    Then a validação deve falhar

  Scenario: Campo "description" é opcional
    When valido o payload:
      """json
      {
        "title": "Tarefa sem descrição"
      }
      """
    Then a validação deve passar

  Scenario: Payload com campos extras é rejeitado (strict mode)
    When valido o payload:
      """json
      {
        "title": "Tarefa",
        "status": "pending",
        "extraField": "não deve estar aqui"
      }
      """
    Then a validação deve falhar
    And a mensagem deve indicar campo desconhecido
```

**Nota:** Esses casos de teste (Gherkin) podem ser automatizados com ferramentas como **Cucumber.js** ou **CodeceptJS**, permitindo executar os mesmos cenários descritos acima de forma programática integrada ao pipeline de CI/CD.
