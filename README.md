# testephpqajr
Atividade teste API REST
# API REST para Gerenciamento de Ordens de Serviço

Este projeto consiste no desenvolvimento de uma API RESTful que permite a criação e listagem de ordens de serviço, com testes automatizados para o método de criação.

## Requisitos

- **Framework**: Laravel PHP (versão mais recente)
- **Banco de dados**: MySQL
- **Ferramenta de testes**: Pest PHP
- **Formato de dados**: JSON (tanto para parâmetros de entrada quanto para respostas)

## Funcionalidades

1. **Método de Criação**:
   - A API permite a criação de ordens de serviço.
   - Parâmetros necessários: todos os campos presentes na tabela `service_orders`.
   - Validações de entrada para evitar erros de inserção e inconsistências no banco de dados.

2. **Método de Listagem**:
   - A API retorna todas as ordens de serviço.
   - A resposta incluirá o nome do usuário relacionado ao campo `userId`, que é uma chave estrangeira para a tabela `users`.

## Estrutura do Banco de Dados

A estrutura do banco de dados é baseada nas tabelas `service_orders` e `users`, conforme descrito no arquivo `db-structure.sql`. A tabela `service_orders` contém as colunas principais, e a tabela `users` está relacionada à coluna `userId`.

## Endpoints da API

### 1. Criação de Ordem de Serviço

**Método**: `POST`

**URL**: `/api/service-orders`

**Parâmetros de entrada** (em JSON):

```json
{
  "user_id": 1,
  "description": "Analista de TI",
  "status": "closed"
}
```

**Resposta de sucesso** (código 201):

```json
{
 "message": "Service order created successfully",
 "data": {
  "user_id": 1,
  "description": "Analista de TI",
  "status": "closed",
  "updated_at": "2024-11-19T20:26:39.000000Z",
  "created_at": "2024-11-19T20:26:39.000000Z",
  "id": 5
 }
}
```

**Resposta de erro** (código 422, validação falha):

```json
{
 "message": "The description field is required.",
 "errors": {
  "description": [
   "The description field is required."
  ]
 }
}
```

### 2. Listagem de Ordens de Serviço

**Método**: `GET`

**URL**: `/api/service-orders`

**Resposta de sucesso** (código 200):

```json
[
  {
  "id": 5,
  "user_id": 1,
  "description": "Analista de TI",
  "status": "closed",
  "created_at": "2024-11-19T20:26:39.000000Z",
  "updated_at": "2024-11-19T20:26:39.000000Z",
  "userId": null
 }
]
```

## Casos de Teste Automatizados

### Teste 1: Criação de Ordem de Serviço com Parâmetros Válidos

- **Descrição**: Verifica se a criação da ordem de serviço retorna o código 201 e os dados esperados.
- **Resultado esperado**: Código 201, resposta JSON com os dados criados e mensagem de sucesso.

```php
#[Test]
public function it_creates_service_order_with_valid_parameters()
{
    $user = User::factory()->create();
    $data = [
      'user_id' => $user->id,
      'description' => 'Teste de criação de ordem de serviço',
      'status' => 'open',
    ];
    $response = $this->postJson('/api/service-orders', $data);

    $response->assertStatus(201)
      ->assertJson([
        'message' => 'Service order created successfully',
        'data' => [
          'user_id' => $user->id,
          'description' => $data['description'],
          'status' => $data['status'],
        ],
      ]);
    $this->assertDatabaseHas('service_orders', $data);
}
```

### Teste 2: Criação de Ordem de Serviço com Parâmetros Inválidos

- **Descrição**: Verifica se a criação da ordem de serviço com parâmetros inválidos (exemplo: campo `description` vazio) retorna o código 422 e mensagens de erro.
- **Resultado esperado**: Código 422, resposta JSON com mensagens de erro de validação.

```php
#[Test]
public function it_creates_service_order_with_invalid_parameters()
{
    $data = [
      'user_id' => null,
      'description' => '',
      'status' => 'invalid_status',
    ];

    $response = $this->postJson('/api/service-orders', $data);

    $response->assertStatus(422)
      ->assertJsonValidationErrors(['user_id', 'description', 'status']);
}
```

### Teste 3: Listagem de Ordens de Serviço

- **Descrição**: Verifica se a listagem de ordens de serviço retorna os dados corretos com o nome do usuário associado a cada ordem.
- **Resultado esperado**: Código 200, resposta JSON com todas as ordens e o nome do usuário.

```php
#[Test]
public function it_lists_service_orders()
{
    $user = User::factory()->create();
    ServiceOrder::factory()->count(3)->create([
      'user_id' => $user->id,
    ]);

    $response = $this->getJson('/api/service-orders');

    $response->assertStatus(200)
      ->assertJsonCount(3)
      ->assertJsonStructure([
        '*' => ['id', 'user_id', 'description', 'status', 'created_at', 'updated_at'],
      ]);
}
```

## Como Executar o Projeto

### 1. Clonar o Repositório

Clone o repositório para o seu ambiente local:

```bash
git clone https://github.com/usuario/repo.git
cd repo
```

### 2. Instalar Dependências

Execute o comando abaixo para instalar todas as dependências do Laravel:

```bash
composer install
```

### 3. Configurar o Banco de Dados

Crie um banco de dados no MySQL e configure as credenciais no arquivo `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nome_do_banco
DB_USERNAME=usuario
DB_PASSWORD=senha
```

Execute as migrações para criar as tabelas no banco de dados:

```bash
php artisan migrate
```

### 4. Rodar os Testes Automatizados

Para rodar os testes com Pest PHP, execute o comando abaixo:

```bash
php artisan test
```

---

