# API de Gestão de Tarefas de Sprint, Sprint Tasks API

[![Build Status](https://github.com/Letytorquato82/Mentoria-Julio-de-Lima/actions/workflows/ci.yml/badge.svg)](https://github.com/Letytorquato82/Mentoria-Julio-de-Lima/actions/workflows/ci.yml)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/Letytorquato82/Mentoria-Julio-de-Lima)
[![GraphiQL](https://img.shields.io/badge/GraphiQL-enabled-brightgreen.svg)](http://localhost:3000/graphiql)

Projeto de API REST para gerenciamento de tarefas de sprint, com foco em automação de testes e validação de contrato.

## Descrição do projeto
Esta aplicação é uma API simples de acompanhamento de entregas de demandas. O objetivo é demonstrar o ciclo completo de CRUD de uma tarefa, com validação por JSON Schema e testes automatizados.

## Tecnologias utilizadas
- Node.js
- Express
- Ajv (validação JSON Schema)
- Jest + Supertest (testes automatizados)
- Newman + Postman (coleção de testes de API)

## Recursos implementados
- CRUD completo de tarefas
- Status de tarefa: `pending`, `in-progress`, `done`
- Validação de payloads com JSON Schema
- Rotas RESTful para gerenciamento de tarefas
- Persistência temporária em arquivo local (`data/tasks.json`)
- Workflow CI no GitHub Actions

## Requisitos
- Node.js 18 ou superior
- npm instalado

> Docker é opcional. Esta documentação presume execução local com Node.js.

## Instalação e execução
1. Clone o repositório e entre na pasta do projeto.
2. Instale dependências:
   ```bash
   npm install
   ```
3. Inicie a API:
   ```bash
   npm start
   ```
4. Acesse a API localmente em:
   ```text
   http://localhost:3000
   ```

## Comandos principais
- `npm install` — instala dependências
- `npm start` — inicia a API
- `npm test` — executa os testes automatizados
- `npm run postman` — executa a coleção Postman via Newman
- `npm run audit:fix` — tenta corrigir vulnerabilidades de dependências

## Estrutura da API
### Endpoints
- `GET /api/tasks`
  - Retorna a lista de tarefas
- `POST /api/tasks`
  - Cria uma nova tarefa
- `GET /api/tasks/:id`
  - Retorna uma tarefa por ID
- `PUT /api/tasks/:id`
  - Atualiza os dados de uma tarefa
- `PATCH /api/tasks/:id/status`
  - Atualiza apenas o status da tarefa
- `DELETE /api/tasks/:id`
  - Remove uma tarefa

### Exemplos de requisições cURL
Use `BASE_URL=http://localhost:3000` para executar os exemplos abaixo.

#### Listar tarefas
```bash
curl -X GET "$BASE_URL/api/tasks" -H "Accept: application/json"
```

#### Criar tarefa
```bash
curl -X POST "$BASE_URL/api/tasks" \
  -H "Content-Type: application/json" \
  -d '{"title":"Implementar endpoint de tarefa","description":"Criar API REST para gestão de entregas","status":"pending"}'
```

#### Consultar tarefa por ID
```bash
curl -X GET "$BASE_URL/api/tasks/{id}" -H "Accept: application/json"
```

#### Atualizar tarefa completa
```bash
curl -X PUT "$BASE_URL/api/tasks/{id}" \
  -H "Content-Type: application/json" \
  -d '{"title":"Tarefa atualizada","description":"Descrição atualizada","status":"in-progress"}'
```

#### Atualizar status da tarefa
```bash
curl -X PATCH "$BASE_URL/api/tasks/{id}/status" \
  -H "Content-Type: application/json" \
  -d '{"status":"done"}'
```

#### Deletar tarefa
```bash
curl -X DELETE "$BASE_URL/api/tasks/{id}"
```

### Modelo de tarefa
O contrato de `Task` está em `contracts/task.schema.json`.

Exemplo de payload para criação:
```json
{
  "title": "Implementar endpoint de tarefa",
  "description": "Criar API REST para gestão de entregas",
  "status": "pending"
}
```

Campos importantes:
- `title` (string, obrigatório)
- `description` (string, opcional)
- `status` (enum: `pending`, `in-progress`, `done`)

## Testes
### Testes automatizados
Use:
```bash
npm test
```

### Coleção Postman
A coleção está em `postman/TaskManagement.postman_collection.json`.
O ambiente de teste está em `postman/TaskManagement.postman_environment.json`.

Configurar `baseUrl` para:
```text
http://localhost:3000
```

### Fluxo de validação recomendado
1. `POST /api/tasks` para criar uma tarefa
2. `GET /api/tasks/:id` para consultar a tarefa
3. `PATCH /api/tasks/:id/status` para marcar como `done`
4. `DELETE /api/tasks/:id` para remover a tarefa

## GraphQL
Este projeto também inclui um endpoint GraphQL disponível em:
```text
http://localhost:3000/graphql
```

Uma interface de exploração GraphiQL também está disponível em:
```text
http://localhost:3000/graphiql
```

### Consultas de exemplo
#### Obter todas as tarefas
```graphql
query {
  tasks {
    id
    title
    description
    status
    createdAt
    updatedAt
  }
}
```

#### Obter tarefa por ID
```graphql
query {
  task(id: "1") {
    id
    title
    status
  }
}
```

### Mutations de exemplo
#### Criar tarefa
```graphql
mutation {
  createTask(input: { title: "Nova tarefa", description: "Descrição", status: "pending" }) {
    id
    title
    status
  }
}
```

#### Atualizar tarefa
```graphql
mutation {
  updateTask(id: "1", input: { title: "Tarefa atualizada", description: "Descrição atualizada", status: "in-progress" }) {
    id
    title
    status
  }
}
```

#### Atualizar status
```graphql
mutation {
  updateTaskStatus(id: "1", status: "done") {
    id
    status
  }
}
```

#### Deletar tarefa
```graphql
mutation {
  deleteTask(id: "1")
}
```

## CI/CD
O projeto inclui uma workflow do GitHub Actions em `.github/workflows/ci.yml` para:
- instalar dependências
- executar testes unitários
- executar a coleção Newman
- executar `npm audit --production` em modo de relatório

## Tabela de decisão
A tabela de decisão de regras de negócio está disponível em `DECISION_TABLE.md`.

## Observações importantes
- O armazenamento atual é em arquivo local (`data/tasks.json`) e serve como persistência simples para desenvolvimento.
- Docker não é obrigatório para este projeto.
- Se você não puder instalar Docker, use apenas Node.js e os comandos acima.

## Contato
Para dúvidas sobre o projeto, abra uma issue no repositório ou entre em contato pelo GitHub.
