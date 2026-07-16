# Guia Completo - PivotPHP Cycle ORM Extension

Este guia apresenta o uso da extensão desde o básico até implementações avançadas com arquitetura limpa.

## Índice

1. [Instalação e Configuração](#instalação-e-configuração)
2. [Uso Básico - Primeiros Passos](#uso-básico---primeiros-passos)
3. [Uso Intermediário - Repositórios](#uso-intermediário---repositórios)
4. [Uso Avançado - Clean Architecture](#uso-avançado---clean-architecture)
5. [Recursos Extras](#recursos-extras)

## Instalação e Configuração

### 1. Instalar via Composer

```bash
composer require pivotphp/cycle-orm
```

### 2. Configurar Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
# Configuração SQLite (desenvolvimento)
DB_CONNECTION=sqlite
DB_DATABASE=./database/database.sqlite

# Configuração MySQL (produção)
# DB_CONNECTION=mysql
# DB_HOST=localhost
# DB_PORT=3306
# DB_DATABASE=meu_banco
# DB_USERNAME=usuario
# DB_PASSWORD=senha

# Configurações de Debug
APP_DEBUG=true
CYCLE_LOG_QUERIES=true
CYCLE_PROFILE_QUERIES=true
```

### 3. Estrutura Inicial do Projeto

```
meu-projeto/
├── public/
│   └── index.php
├── database/
│   └── database.sqlite
├── config/
│   └── cycle.php
├── src/
│   └── (seus arquivos)
├── composer.json
└── .env
```

### 4. Bootstrap da Aplicação

Crie o arquivo `public/index.php`:

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';

use PivotPHP\Core\Core\Application;
use PivotPHP\CycleORM\CycleServiceProvider;

// Criar aplicação PivotPHP
$app = new Application();

// Configurar variáveis de ambiente
$_ENV['DB_CONNECTION'] = 'sqlite';
$_ENV['DB_DATABASE'] = __DIR__ . '/../database/database.sqlite';

// Registrar o Cycle ORM
$app->register(new CycleServiceProvider($app));

// Criar tabela de usuários (apenas para exemplo)
$app->get('/setup', function ($req, $res) use ($app) {
    $database = $app->make('cycle.database');

    // Criar tabela users
    $database->database()->execute('
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name VARCHAR(255) NOT NULL,
            email VARCHAR(255) NOT NULL UNIQUE,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
            updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    ');

    return $res->json(['message' => 'Database setup completed']);
});

// Iniciar aplicação
$app->run();
```

## Uso Básico - Primeiros Passos

### 1. CRUD Simples com Database Direct

```php
// Listar todos os usuários
$app->get('/api/users', function ($req, $res) use ($app) {
    try {
        $database = $app->make('cycle.database');
        $users = $database->database()->query('SELECT * FROM users')->fetchAll();

        return $res->json([
            'status' => 'success',
            'data' => $users
        ]);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});

// Buscar usuário por ID
$app->get('/api/users/:id', function ($req, $res) use ($app) {
    try {
        $database = $app->make('cycle.database');
        $id = $req->params->id;

        $user = $database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        if (!$user) {
            return $res->json([
                'status' => 'error',
                'message' => 'User not found'
            ], 404);
        }

        return $res->json([
            'status' => 'success',
            'data' => $user
        ]);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});

// Criar novo usuário
$app->post('/api/users', function ($req, $res) use ($app) {
    try {
        $database = $app->make('cycle.database');
        $data = $req->body();

        // Validação básica
        if (empty($data->name) || empty($data->email)) {
            return $res->json([
                'status' => 'error',
                'message' => 'Name and email are required'
            ], 400);
        }

        // Verificar email duplicado
        $exists = $database->database()
            ->query('SELECT id FROM users WHERE email = ?', [$data->email])
            ->fetch();

        if ($exists) {
            return $res->json([
                'status' => 'error',
                'message' => 'Email already exists'
            ], 409);
        }

        // Inserir usuário
        $database->database()->insert('users')->values([
            'name' => $data->name,
            'email' => $data->email,
            'created_at' => date('Y-m-d H:i:s'),
            'updated_at' => date('Y-m-d H:i:s')
        ])->run();

        $id = $database->database()->getDriver()->lastInsertID();

        // Buscar usuário criado
        $user = $database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        return $res->json([
            'status' => 'success',
            'data' => $user
        ], 201);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});

// Atualizar usuário
$app->put('/api/users/:id', function ($req, $res) use ($app) {
    try {
        $database = $app->make('cycle.database');
        $id = $req->params->id;
        $data = $req->body();

        // Verificar se usuário existe
        $user = $database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        if (!$user) {
            return $res->json([
                'status' => 'error',
                'message' => 'User not found'
            ], 404);
        }

        // Preparar dados para atualização
        $updateData = ['updated_at' => date('Y-m-d H:i:s')];

        if (isset($data->name)) {
            $updateData['name'] = $data->name;
        }

        if (isset($data->email)) {
            // Verificar email duplicado
            $exists = $database->database()
                ->query('SELECT id FROM users WHERE email = ? AND id != ?', [$data->email, $id])
                ->fetch();

            if ($exists) {
                return $res->json([
                    'status' => 'error',
                    'message' => 'Email already exists'
                ], 409);
            }

            $updateData['email'] = $data->email;
        }

        // Atualizar usuário
        $database->database()
            ->update('users', $updateData, ['id' => $id])
            ->run();

        // Buscar usuário atualizado
        $user = $database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        return $res->json([
            'status' => 'success',
            'data' => $user
        ]);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});

// Deletar usuário
$app->delete('/api/users/:id', function ($req, $res) use ($app) {
    try {
        $database = $app->make('cycle.database');
        $id = $req->params->id;

        // Verificar se usuário existe
        $user = $database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        if (!$user) {
            return $res->json([
                'status' => 'error',
                'message' => 'User not found'
            ], 404);
        }

        // Deletar usuário
        $database->database()
            ->delete('users', ['id' => $id])
            ->run();

        return $res->json([
            'status' => 'success',
            'message' => 'User deleted successfully'
        ]);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});
```

### 2. Usando Query Builder

```php
// Busca com filtros
$app->get('/api/users/search', function ($req, $res) use ($app) {
    $database = $app->make('cycle.database');
    $queryParams = $req->query; // Objeto stdClass com os parâmetros da query

    // Construir query dinamicamente
    $query = $database->database()->select()->from('users');

    if (!empty($queryParams->name)) {
        $query->where('name', 'LIKE', '%' . $queryParams->name . '%');
    }

    if (!empty($queryParams->email)) {
        $query->where('email', 'LIKE', '%' . $queryParams->email . '%');
    }

    // Ordenação
    $orderBy = $queryParams->order_by ?? 'id';
    $orderDir = $queryParams->order_dir ?? 'ASC';
    $query->orderBy($orderBy, $orderDir);

    // Paginação
    $page = (int)($queryParams->page ?? 1);
    $perPage = (int)($queryParams->per_page ?? 10);
    $offset = ($page - 1) * $perPage;

    $query->limit($perPage)->offset($offset);

    // Executar query
    $users = $query->fetchAll();

    // Contar total de registros
    $total = $database->database()
        ->select()
        ->from('users')
        ->count('id');

    return $res->json([
        'status' => 'success',
        'data' => $users,
        'pagination' => [
            'total' => $total,
            'page' => $page,
            'per_page' => $perPage,
            'total_pages' => ceil($total / $perPage)
        ]
    ]);
});
```

## Uso Intermediário - Repositórios

### 1. Criar Entidade

```php
// src/Entities/User.php
<?php

namespace App\Entities;

class User
{
    private ?int $id;
    private string $name;
    private string $email;
    private \DateTimeInterface $createdAt;
    private \DateTimeInterface $updatedAt;

    public function __construct(
        string $name,
        string $email,
        ?int $id = null,
        ?\DateTimeInterface $createdAt = null,
        ?\DateTimeInterface $updatedAt = null
    ) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
        $this->createdAt = $createdAt ?? new \DateTime();
        $this->updatedAt = $updatedAt ?? new \DateTime();
    }

    // Getters
    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function getCreatedAt(): \DateTimeInterface
    {
        return $this->createdAt;
    }

    public function getUpdatedAt(): \DateTimeInterface
    {
        return $this->updatedAt;
    }

    // Setters
    public function setName(string $name): void
    {
        $this->name = $name;
        $this->touch();
    }

    public function setEmail(string $email): void
    {
        $this->email = $email;
        $this->touch();
    }

    private function touch(): void
    {
        $this->updatedAt = new \DateTime();
    }

    // Converter para array
    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->createdAt->format('Y-m-d H:i:s'),
            'updated_at' => $this->updatedAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

### 2. Criar Interface do Repositório

```php
// src/Repositories/UserRepositoryInterface.php
<?php

namespace App\Repositories;

use App\Entities\User;

interface UserRepositoryInterface
{
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function findAll(array $filters = [], int $page = 1, int $perPage = 10): array;
    public function save(User $user): void;
    public function delete(User $user): void;
    public function count(array $filters = []): int;
}
```

### 3. Implementar Repositório

```php
// src/Repositories/UserRepository.php
<?php

namespace App\Repositories;

use App\Entities\User;
use Cycle\Database\DatabaseInterface;

class UserRepository implements UserRepositoryInterface
{
    public function __construct(
        private DatabaseInterface $database
    ) {}

    public function findById(int $id): ?User
    {
        $result = $this->database
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();

        return $result ? $this->mapToEntity($result) : null;
    }

    public function findByEmail(string $email): ?User
    {
        $result = $this->database
            ->query('SELECT * FROM users WHERE email = ?', [$email])
            ->fetch();

        return $result ? $this->mapToEntity($result) : null;
    }

    public function findAll(array $filters = [], int $page = 1, int $perPage = 10): array
    {
        $query = $this->database->select()->from('users');

        // Aplicar filtros
        if (!empty($filters['name'])) {
            $query->where('name', 'LIKE', '%' . $filters['name'] . '%');
        }

        if (!empty($filters['email'])) {
            $query->where('email', 'LIKE', '%' . $filters['email'] . '%');
        }

        // Paginação
        $offset = ($page - 1) * $perPage;
        $query->limit($perPage)->offset($offset);

        // Ordenação
        $query->orderBy('id', 'DESC');

        $results = $query->fetchAll();

        return array_map([$this, 'mapToEntity'], $results);
    }

    public function save(User $user): void
    {
        $data = [
            'name' => $user->getName(),
            'email' => $user->getEmail(),
            'updated_at' => $user->getUpdatedAt()->format('Y-m-d H:i:s'),
        ];

        if ($user->getId()) {
            // Atualizar
            $this->database
                ->update('users', $data, ['id' => $user->getId()])
                ->run();
        } else {
            // Inserir
            $data['created_at'] = $user->getCreatedAt()->format('Y-m-d H:i:s');

            $this->database
                ->insert('users')
                ->values($data)
                ->run();

            // Definir ID no objeto
            $id = $this->database->getDriver()->lastInsertID();
            $reflection = new \ReflectionClass($user);
            $property = $reflection->getProperty('id');
            $property->setAccessible(true);
            $property->setValue($user, $id);
        }
    }

    public function delete(User $user): void
    {
        if ($user->getId()) {
            $this->database
                ->delete('users', ['id' => $user->getId()])
                ->run();
        }
    }

    public function count(array $filters = []): int
    {
        $query = $this->database->select()->from('users');

        if (!empty($filters['name'])) {
            $query->where('name', 'LIKE', '%' . $filters['name'] . '%');
        }

        if (!empty($filters['email'])) {
            $query->where('email', 'LIKE', '%' . $filters['email'] . '%');
        }

        return $query->count('id');
    }

    private function mapToEntity(array $data): User
    {
        return new User(
            name: $data['name'],
            email: $data['email'],
            id: $data['id'],
            createdAt: new \DateTime($data['created_at']),
            updatedAt: new \DateTime($data['updated_at'])
        );
    }
}
```

### 4. Configurar Container de Injeção de Dependência

```php
// src/Container.php
<?php

namespace App;

class Container
{
    private array $bindings = [];
    private array $instances = [];

    public function bind(string $abstract, \Closure $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }

    public function singleton(string $abstract, \Closure $concrete): void
    {
        $this->bind($abstract, function () use ($abstract, $concrete) {
            if (!isset($this->instances[$abstract])) {
                $this->instances[$abstract] = $concrete($this);
            }
            return $this->instances[$abstract];
        });
    }

    public function get(string $abstract)
    {
        if (!isset($this->bindings[$abstract])) {
            throw new \Exception("Binding [{$abstract}] not found");
        }

        return $this->bindings[$abstract]($this);
    }
}
```

### 5. Usar Repositórios nos Controllers

```php
// Configurar container
$container = new \App\Container();

// Registrar database
$container->singleton('database', function () use ($app) {
    return $app->make('cycle.database');
});

// Registrar repositório
$container->singleton(\App\Repositories\UserRepositoryInterface::class, function ($container) {
    return new \App\Repositories\UserRepository(
        $container->get('database')
    );
});

// Controller usando repositório
$app->get('/api/v2/users', function ($req, $res) use ($container) {
    try {
        $repository = $container->get(\App\Repositories\UserRepositoryInterface::class);
        $query = $req->query; // stdClass com parâmetros

        // Converter stdClass para array para o repositório
        $filters = [];
        if (isset($query->name)) $filters['name'] = $query->name;
        if (isset($query->email)) $filters['email'] = $query->email;

        $page = (int)($query->page ?? 1);
        $perPage = (int)($query->per_page ?? 10);

        $users = $repository->findAll($filters, $page, $perPage);
        $total = $repository->count($filters);

        return $res->json([
            'status' => 'success',
            'data' => array_map(fn($user) => $user->toArray(), $users),
            'pagination' => [
                'total' => $total,
                'page' => $page,
                'per_page' => $perPage,
                'total_pages' => ceil($total / $perPage)
            ]
        ]);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});

$app->post('/api/v2/users', function ($req, $res) use ($container) {
    try {
        $repository = $container->get(\App\Repositories\UserRepositoryInterface::class);
        $data = $req->body();

        // Validação
        if (empty($data->name) || empty($data->email)) {
            return $res->json([
                'status' => 'error',
                'message' => 'Name and email are required'
            ], 400);
        }

        // Verificar email duplicado
        if ($repository->findByEmail($data->email)) {
            return $res->json([
                'status' => 'error',
                'message' => 'Email already exists'
            ], 409);
        }

        // Criar usuário
        $user = new \App\Entities\User(
            name: $data->name,
            email: $data->email
        );

        $repository->save($user);

        return $res->json([
            'status' => 'success',
            'data' => $user->toArray()
        ], 201);
    } catch (\Exception $e) {
        return $res->json([
            'status' => 'error',
            'message' => $e->getMessage()
        ], 500);
    }
});
```

## Uso Avançado - Clean Architecture

### 1. Estrutura de Diretórios

```
src/
├── Domain/
│   ├── Entities/
│   │   └── User.php
│   ├── ValueObjects/
│   │   ├── Email.php
│   │   └── Name.php
│   ├── Repositories/
│   │   └── UserRepositoryInterface.php
│   └── Exceptions/
│       ├── InvalidEmailException.php
│       └── UserNotFoundException.php
├── Application/
│   ├── UseCases/
│   │   ├── CreateUser/
│   │   │   ├── CreateUserUseCase.php
│   │   │   ├── CreateUserDTO.php
│   │   │   └── CreateUserPresenter.php
│   │   └── ListUsers/
│   │       ├── ListUsersUseCase.php
│   │       └── ListUsersDTO.php
│   └── Services/
│       └── EmailService.php
├── Infrastructure/
│   ├── Repositories/
│   │   └── CycleUserRepository.php
│   ├── Services/
│   │   └── SmtpEmailService.php
│   └── Persistence/
│       └── CycleORM/
└── Presentation/
    ├── Controllers/
    │   └── UserController.php
    └── Middleware/
        └── ValidationMiddleware.php
```

### 2. Value Objects

```php
// src/Domain/ValueObjects/Email.php
<?php

namespace App\Domain\ValueObjects;

use App\Domain\Exceptions\InvalidEmailException;

final class Email
{
    private string $value;

    public function __construct(string $value)
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException("Invalid email format: {$value}");
        }

        $this->value = strtolower($value);
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function equals(Email $other): bool
    {
        return $this->value === $other->value;
    }

    public function __toString(): string
    {
        return $this->value;
    }
}

// src/Domain/ValueObjects/Name.php
<?php

namespace App\Domain\ValueObjects;

use App\Domain\Exceptions\InvalidNameException;

final class Name
{
    private string $value;

    public function __construct(string $value)
    {
        $value = trim($value);

        if (empty($value)) {
            throw new InvalidNameException("Name cannot be empty");
        }

        if (strlen($value) < 2) {
            throw new InvalidNameException("Name must be at least 2 characters long");
        }

        if (strlen($value) > 255) {
            throw new InvalidNameException("Name cannot exceed 255 characters");
        }

        $this->value = $value;
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function __toString(): string
    {
        return $this->value;
    }
}
```

### 3. Entidade com Value Objects

```php
// src/Domain/Entities/User.php
<?php

namespace App\Domain\Entities;

use App\Domain\ValueObjects\Email;
use App\Domain\ValueObjects\Name;

class User
{
    private ?int $id;
    private Name $name;
    private Email $email;
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;

    private function __construct(
        Name $name,
        Email $email,
        ?int $id = null,
        ?\DateTimeImmutable $createdAt = null,
        ?\DateTimeImmutable $updatedAt = null
    ) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
        $this->createdAt = $createdAt ?? new \DateTimeImmutable();
        $this->updatedAt = $updatedAt ?? new \DateTimeImmutable();
    }

    public static function create(string $name, string $email): self
    {
        return new self(
            new Name($name),
            new Email($email)
        );
    }

    public static function fromPrimitives(
        int $id,
        string $name,
        string $email,
        string $createdAt,
        string $updatedAt
    ): self {
        return new self(
            new Name($name),
            new Email($email),
            $id,
            new \DateTimeImmutable($createdAt),
            new \DateTimeImmutable($updatedAt)
        );
    }

    public function updateName(string $name): void
    {
        $this->name = new Name($name);
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function updateEmail(string $email): void
    {
        $this->email = new Email($email);
        $this->updatedAt = new \DateTimeImmutable();
    }

    // Getters
    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): Name
    {
        return $this->name;
    }

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function getCreatedAt(): \DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function getUpdatedAt(): \DateTimeImmutable
    {
        return $this->updatedAt;
    }

    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name->getValue(),
            'email' => $this->email->getValue(),
            'created_at' => $this->createdAt->format('Y-m-d H:i:s'),
            'updated_at' => $this->updatedAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

### 4. Use Cases

```php
// src/Application/UseCases/CreateUser/CreateUserDTO.php
<?php

namespace App\Application\UseCases\CreateUser;

final class CreateUserDTO
{
    public function __construct(
        public readonly string $name,
        public readonly string $email
    ) {}

    public static function fromArray(array $data): self
    {
        return new self(
            name: $data['name'] ?? '',
            email: $data['email'] ?? ''
        );
    }
}

// src/Application/UseCases/CreateUser/CreateUserUseCase.php
<?php

namespace App\Application\UseCases\CreateUser;

use App\Domain\Entities\User;
use App\Domain\Repositories\UserRepositoryInterface;
use App\Domain\Exceptions\EmailAlreadyExistsException;

final class CreateUserUseCase
{
    public function __construct(
        private UserRepositoryInterface $userRepository
    ) {}

    public function execute(CreateUserDTO $dto): User
    {
        // Verificar se email já existe
        $existingUser = $this->userRepository->findByEmail($dto->email);
        if ($existingUser !== null) {
            throw new EmailAlreadyExistsException("Email {$dto->email} already exists");
        }

        // Criar usuário (validação acontece nos Value Objects)
        $user = User::create($dto->name, $dto->email);

        // Salvar no repositório
        $this->userRepository->save($user);

        return $user;
    }
}

// src/Application/UseCases/ListUsers/ListUsersUseCase.php
<?php

namespace App\Application\UseCases\ListUsers;

use App\Domain\Repositories\UserRepositoryInterface;

final class ListUsersUseCase
{
    public function __construct(
        private UserRepositoryInterface $userRepository
    ) {}

    public function execute(ListUsersDTO $dto): ListUsersResult
    {
        $users = $this->userRepository->findAll(
            filters: $dto->filters,
            page: $dto->page,
            perPage: $dto->perPage
        );

        $total = $this->userRepository->count($dto->filters);

        return new ListUsersResult(
            users: $users,
            total: $total,
            page: $dto->page,
            perPage: $dto->perPage
        );
    }
}
```

### 5. Controller Final

```php
// src/Presentation/Controllers/UserController.php
<?php

namespace App\Presentation\Controllers;

use PivotPHP\PivotPHP\Core\Request;
use PivotPHP\PivotPHP\Core\Response;
use App\Application\UseCases\CreateUser\CreateUserUseCase;
use App\Application\UseCases\CreateUser\CreateUserDTO;
use App\Application\UseCases\ListUsers\ListUsersUseCase;
use App\Application\UseCases\ListUsers\ListUsersDTO;

final class UserController
{
    public function __construct(
        private CreateUserUseCase $createUserUseCase,
        private ListUsersUseCase $listUsersUseCase
    ) {}

    public function index(Request $request): Response
    {
        try {
            // Converter query object para array de filtros
            $filters = [];
            if (isset($request->query->name)) $filters['name'] = $request->query->name;
            if (isset($request->query->email)) $filters['email'] = $request->query->email;

            $dto = new ListUsersDTO(
                filters: $filters,
                page: (int)($request->query->page ?? 1),
                perPage: (int)($request->query->per_page ?? 10)
            );

            $result = $this->listUsersUseCase->execute($dto);

            return Response::json([
                'status' => 'success',
                'data' => array_map(fn($user) => $user->toArray(), $result->users),
                'pagination' => [
                    'total' => $result->total,
                    'page' => $result->page,
                    'per_page' => $result->perPage,
                    'total_pages' => $result->getTotalPages()
                ]
            ]);
        } catch (\Exception $e) {
            return Response::json([
                'status' => 'error',
                'message' => $e->getMessage()
            ], 500);
        }
    }

    public function store(Request $request): Response
    {
        try {
            // Converter stdClass para array para o DTO
            $data = [
                'name' => $request->body()->name ?? '',
                'email' => $request->body()->email ?? ''
            ];
            $dto = CreateUserDTO::fromArray($data);
            $user = $this->createUserUseCase->execute($dto);

            return Response::json([
                'status' => 'success',
                'data' => $user->toArray()
            ], 201);
        } catch (EmailAlreadyExistsException $e) {
            return Response::json([
                'status' => 'error',
                'message' => $e->getMessage()
            ], 409);
        } catch (InvalidEmailException | InvalidNameException $e) {
            return Response::json([
                'status' => 'error',
                'message' => $e->getMessage()
            ], 400);
        } catch (\Exception $e) {
            return Response::json([
                'status' => 'error',
                'message' => 'An error occurred while creating the user'
            ], 500);
        }
    }
}
```

### 6. Configuração Final do Container

```php
// bootstrap/container.php
<?php

use App\Infrastructure\Container\DIContainer;

$container = new DIContainer();

// Registrar serviços do Cycle ORM
$container->singleton('database', fn() => $app->make('cycle.database'));
$container->singleton('orm', fn() => $app->make('cycle.orm'));
$container->singleton('entityManager', fn() => $app->make('cycle.em'));

// Registrar repositórios
$container->singleton(
    \App\Domain\Repositories\UserRepositoryInterface::class,
    fn($c) => new \App\Infrastructure\Repositories\CycleUserRepository(
        $c->get('database')
    )
);

// Registrar use cases
$container->bind(
    \App\Application\UseCases\CreateUser\CreateUserUseCase::class,
    fn($c) => new \App\Application\UseCases\CreateUser\CreateUserUseCase(
        $c->get(\App\Domain\Repositories\UserRepositoryInterface::class)
    )
);

$container->bind(
    \App\Application\UseCases\ListUsers\ListUsersUseCase::class,
    fn($c) => new \App\Application\UseCases\ListUsers\ListUsersUseCase(
        $c->get(\App\Domain\Repositories\UserRepositoryInterface::class)
    )
);

// Registrar controllers
$container->bind(
    \App\Presentation\Controllers\UserController::class,
    fn($c) => new \App\Presentation\Controllers\UserController(
        $c->get(\App\Application\UseCases\CreateUser\CreateUserUseCase::class),
        $c->get(\App\Application\UseCases\ListUsers\ListUsersUseCase::class)
    )
);

return $container;
```

## Recursos Extras

### Métodos Corretos do PivotPHP

**Request:**
- `$req->params->id` - Parâmetros de rota (não getAttribute)
- `$req->query->page` - Query string (não getQueryParams)
- `$req->body()` - Corpo da requisição como stdClass (não getParsedBody)
- `$req->input('key', 'default')` - Helper para acessar valores do body

**Response:**
- `$res->json($data, $status)` - Resposta JSON
- `$res->send($content, $status)` - Resposta texto
- `$res->status($code)` - Define status HTTP

**Rotas:**
- Use `/users/:id` ao invés de `/users/{id}`
- Use `$req->params->id` ao invés de `$req->getAttribute('id')`

### 1. Middleware de Transação

```php
use PivotPHP\CycleORM\Middleware\TransactionMiddleware;

// Aplicar em todas as rotas
$app->use(new TransactionMiddleware($app));

// Ou em rotas específicas
$app->post('/api/users', function ($req, $res) {
    // Transação iniciada automaticamente
    // Commit em sucesso, rollback em erro
})->add(new TransactionMiddleware($app));
```

### 2. Migrations e Schema

Não há `bin/console` incluído no pacote nem um comando para gerar arquivos de migration
(`SchemaCommand`/`MigrateCommand`/`StatusCommand` não expõem isso). Invoque as classes
diretamente a partir do seu próprio script `bin/console` (veja
[integration-guide.md](integration-guide.md#-comandos-cli)):

```php
(new SchemaCommand(['--sync' => true], $container))->handle(); // sincronizar schema
(new MigrateCommand([], $container))->handle();                // executar migrations
(new StatusCommand([], $container))->handle();                 // verificar status
```

### 3. Debugging e Profiling

```php
// Ativar log de queries
$_ENV['CYCLE_LOG_QUERIES'] = true;

// Ativar profiling
$_ENV['CYCLE_PROFILE_QUERIES'] = true;

// Coletar métricas
use PivotPHP\CycleORM\Monitoring\MetricsCollector;

$app->get('/metrics', function ($req, $res) {
    $metrics = MetricsCollector::getMetrics();
    return $res->json($metrics);
});
```

### 4. Health Check

```php
use PivotPHP\CycleORM\Health\HealthCheckMiddleware;

$app->get('/health', function ($req, $res) {
    return $res->json(['status' => 'ok']);
})->add(new HealthCheckMiddleware($app));
```

## Conclusão

Esta extensão oferece flexibilidade para trabalhar desde implementações simples até arquiteturas complexas. Escolha o nível de complexidade adequado ao seu projeto:

- **Básico**: Para MVPs e prototipagem rápida
- **Intermediário**: Para aplicações médias com necessidade de organização
- **Avançado**: Para sistemas empresariais com requisitos complexos

Para mais informações, consulte a documentação completa ou abra uma issue no GitHub.
