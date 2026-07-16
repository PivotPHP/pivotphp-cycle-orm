# PivotPHP Cycle ORM

<div align="center">

[![PHP Version](https://img.shields.io/badge/php-%3E%3D8.1-blue.svg)](https://www.php.net/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Latest Stable Version](https://img.shields.io/badge/version-1.0.1-brightgreen.svg)](https://github.com/PivotPHP/pivotphp-cycle-orm/releases)
[![PHPStan](https://img.shields.io/badge/PHPStan-Level%209-success.svg)](https://phpstan.org/)
[![Tests](https://img.shields.io/badge/tests-67%20passed-success.svg)](https://github.com/PivotPHP/pivotphp-cycle-orm/actions)

Robust and well-tested Cycle ORM integration for PivotPHP microframework

</div>

## 🚀 Features

- **Seamless Integration**: Deep integration with PivotPHP Core
- **Type Safety**: Full type safety with PHPStan Level 9
- **Repository Pattern**: Built-in repository pattern support
- **Performance Monitoring**: Query logging and performance profiling
- **Middleware Support**: Transaction and validation middleware
- **Health Checks**: Database health monitoring
- **Zero Configuration**: Works out of the box with sensible defaults

## 📦 Installation

```bash
composer require pivotphp/cycle-orm
```

### Development Setup

When developing locally with both pivotphp-core and pivotphp-cycle-orm:

1. Clone both repositories in the same parent directory:
```bash
git clone https://github.com/PivotPHP/pivotphp-core.git
git clone https://github.com/PivotPHP/pivotphp-cycle-orm.git
```

2. Install dependencies:
```bash
cd pivotphp-cycle-orm
composer install
```

**Note**: `composer.json` does not currently declare a local `repositories` path
to a sibling `pivotphp-core` checkout — it resolves `pivotphp/core` from Packagist
like any other dependency, both locally and in CI. If you need to develop against
an unreleased `pivotphp-core` change, add a local path repository yourself.

## 🔧 Quick Start

### 1. Register the Service Provider

```php
use PivotPHP\Core\Core\Application;
use PivotPHP\CycleORM\CycleServiceProvider;

$app = new Application();
$app->register(new CycleServiceProvider($app));
```

### 2. Configure Database

```php
// config/cycle.php
return [
    'database' => [
        'default' => 'default',
        'databases' => [
            'default' => ['connection' => 'sqlite']
        ],
        'connections' => [
            'sqlite' => [
                'driver' => \Cycle\Database\Driver\SQLite\SQLiteDriver::class,
                'options' => [
                    'connection' => 'sqlite:database.db',
                ]
            ]
        ]
    ]
];
```

### 3. Define Entities

```php
use Cycle\Annotated\Annotation\Entity;
use Cycle\Annotated\Annotation\Column;

#[Entity(repository: UserRepository::class)]
class User
{
    #[Column(type: 'primary')]
    private int $id;

    #[Column(type: 'string')]
    private string $name;

    #[Column(type: 'string', unique: true)]
    private string $email;

    // Getters and setters...
}
```

### 4. Use in Routes

```php
$app->get('/users', function (CycleRequest $request) {
    $users = $request->getRepository(User::class)->findAll();

    return $request->response()->json($users);
});

$app->post('/users', function (CycleRequest $request) {
    $user = new User();
    $user->setName($request->input('name'));
    $user->setEmail($request->input('email'));

    $request->persist($user);

    return $request->response()->json($user, 201);
});
```

## 🎯 Core Features

### Repository Pattern

```php
// Custom repository
class UserRepository extends Repository
{
    public function findByEmail(string $email): ?User
    {
        return $this->findOne(['email' => $email]);
    }

    public function findActive(): array
    {
        return $this->select()
            ->where('active', true)
            ->orderBy('created_at', 'DESC')
            ->fetchAll();
    }
}
```

### Transaction Middleware

```php
use PivotPHP\CycleORM\Middleware\TransactionMiddleware;

// Automatic transaction handling
$app->post('/api/orders',
    new TransactionMiddleware($app),
    function (CycleRequest $request) {
        // All database operations are wrapped in a transaction
        $order = new Order();
        $request->persist($order);

        // If an exception occurs, transaction is rolled back
        foreach ($request->input('items') as $item) {
            $orderItem = new OrderItem();
            $request->persist($orderItem);
        }

        return $request->response()->json($order);
    }
);
```

### Query Monitoring

```php
use PivotPHP\CycleORM\Monitoring\QueryLogger;

// Enable query logging
$logger = $app->get(QueryLogger::class);
$logger->enable();

// Get query statistics
$stats = $logger->getStatistics();
// [
//     'total_queries' => 42,
//     'total_time' => 0.123,
//     'queries' => [...]
// ]
```

### Health Checks

```php
use PivotPHP\CycleORM\Health\CycleHealthCheck;

$app->get('/health', function () use ($app) {
    $health = $app->get(CycleHealthCheck::class);
    $status = $health->check();

    return [
        'status' => $status->isHealthy() ? 'healthy' : 'unhealthy',
        'database' => $status->getData()
    ];
});
```

## 🛠️ Advanced Usage

### Entity Validation Middleware

`EntityValidationMiddleware` takes no constructor arguments and does not support
Laravel-style rule strings (`'required|string|min:3'`). It wraps the request in a
`CycleRequest` and exposes `validateEntity(object $entity): array{valid: bool, errors: array<int, string>}`,
a basic reflection-based check (required non-nullable properties, `string`/`int` type
mismatches) that you call yourself inside your handler:

```php
use PivotPHP\CycleORM\Middleware\EntityValidationMiddleware;

$validation = new EntityValidationMiddleware();

$app->post('/users',
    $validation,
    function (CycleRequest $request, $res) use ($validation) {
        $user = new User();
        // ...populate $user from $request...

        $result = $validation->validateEntity($user);
        if (!$result['valid']) {
            return $res->status(422)->json(['errors' => $result['errors']]);
        }

        // persist $user...
    }
);
```

### Performance Profiling

```php
use PivotPHP\CycleORM\Monitoring\PerformanceProfiler;

$profiler = $app->get(PerformanceProfiler::class);
$profiler->startProfiling();

// Your database operations...

$profile = $profiler->stopProfiling();
// [
//     'duration' => 0.456,
//     'memory_peak' => 2097152,
//     'queries_count' => 15
// ]
```

### Custom Commands

This package does **not** ship a `bin/console` executable or a `vendor/bin/pivotphp`
binary. `Commands\SchemaCommand`, `MigrateCommand`, `EntityCommand`, and `StatusCommand`
are plain PHP classes with a `handle(): int` method — instantiate them yourself
(typically from a small console script your own project provides):

```php
use PivotPHP\CycleORM\Commands\{SchemaCommand, MigrateCommand, EntityCommand, StatusCommand};

(new EntityCommand(['name' => 'User'], $container))->handle();   // create entity
(new MigrateCommand([], $container))->handle();                  // run migrations
(new SchemaCommand(['--sync' => true], $container))->handle();   // sync schema
(new StatusCommand([], $container))->handle();                   // check status
```

See [Configurar Console](docs/integration-guide.md#-comandos-cli) for a complete
example `bin/console` script that wires these into runnable CLI commands.

## 🧪 Testing

```bash
# Run all tests
composer test

# Run specific test suite
composer test:unit
composer test:feature
composer test:integration

# Run with coverage (cross-platform)
composer test-coverage

# Platform-specific alternatives:
# Unix/Linux/macOS
./scripts/test-coverage.sh
# Windows CMD
scripts\test-coverage.bat
# PowerShell
scripts\test-coverage.ps1
```

### Cross-Platform Compatibility

The project includes cross-platform scripts for coverage testing:
- **Primary method**: `composer test-coverage` (works on all platforms)
- **Alternative scripts**: Platform-specific scripts in `scripts/` directory
- **Windows support**: Both CMD and PowerShell scripts included

## 📚 Documentation

- [Integration Guide](docs/integration-guide.md)
- [Complete Guide](docs/guia-completo.md)
- [API Reference](docs/quick-reference.md)
- [Examples](examples/)

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## 📄 License

PivotPHP Cycle ORM is open-sourced software licensed under the [MIT license](LICENSE).

## 🙏 Credits

- Created by [Caio Alberto Fernandes](https://github.com/CAFernandes)
- Built on top of [Cycle ORM](https://cycle-orm.dev/)
- Part of the [PivotPHP](https://github.com/PivotPHP) ecosystem

---

<div align="center">
  <sub>Built with PivotPHP - The modern PHP microframework</sub>
</div>
