# Referência Rápida - PivotPHP Cycle ORM Extension

## Instalação e Configuração

```bash
composer require pivotphp/cycle-orm
```

```php
// index.php
chdir(dirname(__DIR__)); // IMPORTANTE!
require 'vendor/autoload.php';

use PivotPHP\CycleORM\CycleServiceProvider;
use PivotPHP\CycleORM\Middleware\CycleMiddleware;

$app = new PivotPHP\Core\Core\Application();

// Configurar banco
$_ENV['DB_CONNECTION'] = 'sqlite';
$_ENV['DB_DATABASE'] = './database/database.sqlite';

// Registrar extensão
$app->register(new CycleServiceProvider($app));
$app->use(new CycleMiddleware($app));
```

## Estrutura de Diretórios Obrigatória

```
projeto/
├── app/
│   └── Entities/    # Obrigatório (mesmo vazio)
├── database/
│   └── database.sqlite
└── public/
    └── index.php
```

## Métodos CycleRequest

```php
// Em qualquer rota com CycleMiddleware ativo
$app->get('/exemplo', function ($req, $res) {
    // Acessar serviços via container
    $db = $req->getContainer()->get('cycle.database');
    $orm = $req->getContainer()->get('cycle.orm');
    $em = $req->getContainer()->get('cycle.em');

    // Métodos de conveniência
    $repo = $req->repository(User::class);
    $user = $req->entity(User::class, ['name' => 'João']);
    // paginate() takes a Cycle Select query, not a class-string — signature is
    // paginate(Select $query, int $page = 1, int $perPage = 15)
    $query = $req->repository(User::class)->select();
    $page = $req->paginate($query, 1, 20);

    // Propriedades diretas
    $orm = $req->orm;
    $em = $req->em;
});
```

## Operações de Banco de Dados

### Queries Diretas
```php
$db = $app->make('cycle.database');

// SELECT
$users = $db->database()->query('SELECT * FROM users')->fetchAll();
$user = $db->database()->query('SELECT * FROM users WHERE id = ?', [1])->fetch();

// INSERT
$db->database()->execute(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ['João', 'joao@example.com']
);
$id = $db->database()->getDriver()->lastInsertID();

// UPDATE
$db->database()->execute(
    'UPDATE users SET name = ? WHERE id = ?',
    ['João Silva', 1]
);

// DELETE
$db->database()->execute('DELETE FROM users WHERE id = ?', [1]);
```

### Query Builder
```php
// SELECT com builder
$users = $db->database()
    ->select('name', 'email')
    ->from('users')
    ->where('active', '=', 1)
    ->orderBy('created_at', 'DESC')
    ->limit(10)
    ->fetchAll();

// INSERT com builder
$db->database()
    ->insert('users')
    ->values([
        'name' => 'Maria',
        'email' => 'maria@example.com',
        'created_at' => date('Y-m-d H:i:s')
    ])
    ->run();
```

## Padrão Repository

```php
// Definir repositório
class UserRepository
{
    public function __construct(
        private DatabaseInterface $database
    ) {}

    public function findAll(): array
    {
        return $this->database->database()
            ->query('SELECT * FROM users')
            ->fetchAll();
    }

    public function findById(int $id): ?array
    {
        return $this->database->database()
            ->query('SELECT * FROM users WHERE id = ?', [$id])
            ->fetch();
    }
}

// Registrar no container
$app->getContainer()->bind(UserRepository::class, function ($c) {
    return new UserRepository($c->get('cycle.database'));
});

// Usar em rotas
$repo = $app->make(UserRepository::class);
$users = $repo->findAll();
```

## Middlewares

### TransactionMiddleware
```php
// Transações automáticas
$app->use(new TransactionMiddleware($app));

$app->post('/api/transfer', function ($req, $res) {
    // Início automático da transação
    // ...operações no banco...
    // Commit automático em sucesso
    // Rollback automático em erro
});
```

### EntityValidationMiddleware
```php
// Validação automática de entidades
$app->use(new EntityValidationMiddleware($app));
```

## Monitoramento e Debug

```php
// Ativar logging de queries
$_ENV['CYCLE_LOG_QUERIES'] = true;
$_ENV['CYCLE_PROFILE_QUERIES'] = true;

// Coletar métricas
use PivotPHP\CycleORM\Monitoring\MetricsCollector;

$metrics = MetricsCollector::getMetrics();
// ['queries' => 10, 'time' => 0.125, 'cache_hits' => 5]
```

## Comandos CLI

Não há `bin/console` incluído no pacote. `SchemaCommand`, `MigrateCommand`, `StatusCommand`
são classes PHP simples (`handle(): int`) — você as invoca a partir de um script `bin/console`
próprio do seu projeto (veja o exemplo completo em [integration-guide.md](integration-guide.md#-comandos-cli)):

```php
(new SchemaCommand(['--sync' => true], $container))->handle(); // sincronizar schema
(new StatusCommand([], $container))->handle();                 // status das migrações
(new MigrateCommand([], $container))->handle();                // executar migrações
```

## Resolução de Problemas

### PHP 8.4 - Avisos de Depreciação
```bash
# Usar PHP 8.1 ou 8.3
php8.1 -S localhost:8000 public/index.php

# Ou suprimir avisos (não recomendado)
php -d error_reporting=E_ALL~E_DEPRECATED -S localhost:8000
```

### Erro: "app/Entities directory does not exist"
```bash
# Criar diretório
mkdir -p app/Entities

# E definir working directory no código
chdir(dirname(__DIR__));
```

### Erro de Tipo com CycleMiddleware
```php
// Registrar apenas uma vez, globalmente
$app->use(new CycleMiddleware($app)); // ✓

// Não registrar em rotas individuais
```

## Exemplo Completo

```php
<?php
chdir(dirname(__DIR__));
require 'vendor/autoload.php';

use PivotPHP\CycleORM\CycleServiceProvider;
use PivotPHP\CycleORM\Middleware\CycleMiddleware;
use PivotPHP\Core\Core\Application;

$app = new Application();

$_ENV['DB_CONNECTION'] = 'sqlite';
$_ENV['DB_DATABASE'] = './database/database.sqlite';

$app->register(new CycleServiceProvider($app));
$app->use(new CycleMiddleware($app));

// API REST
$app->get('/api/users', function ($req, $res) {
    $db = $req->getContainer()->get('cycle.database');
    $users = $db->database()->query('SELECT * FROM users')->fetchAll();
    return $res->json(['data' => $users]);
});

$app->post('/api/users', function ($req, $res) {
    $db = $req->getContainer()->get('cycle.database');
    $data = $req->body;

    $db->database()->execute(
        'INSERT INTO users (name, email) VALUES (?, ?)',
        [$data->name, $data->email]
    );

    return $res->json(['message' => 'Created'], 201);
});

$app->run();
```
