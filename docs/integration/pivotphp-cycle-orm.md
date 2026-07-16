# Guia de Integração: PivotPHP + Cycle ORM

Este guia mostra como integrar o Cycle ORM ao microframework PivotPHP, cobrindo desde a instalação até exemplos de uso real com entidades, migrations, repositórios customizados, endpoints HTTP, troubleshooting e dicas avançadas.

## Índice

1. [Visão Geral](#visão-geral)
2. [Instalação](#1-instalação)
3. [Dependências](#dependências)
4. [Estrutura Recomendada de Pastas](#estrutura-recomendada-de-pastas)
5. [Configuração Básica](#2-configuração-básica)
6. [Criando uma Entidade](#3-criando-uma-entidade)
7. [Migration](#4-migration)
8. [Uso no Controller/Service](#5-uso-no-controller/service)
9. [Repositórios Customizados](#6-repositórios-customizados)
10. [Health Check](#7-health-check)
11. [Monitoramento e Métricas](#8-monitoramento-e-métricas)
12. [Exemplo de Endpoint HTTP](#exemplo-de-endpoint-http)
13. [Ciclo de Vida e Dicas de Debug](#ciclo-de-vida-e-dicas-de-debug)
14. [Troubleshooting](#troubleshooting)
15. [Links Úteis](#links-úteis)

---

## Visão Geral

A extensão PivotPHP Cycle ORM integra o Cycle ORM ao microframework PivotPHP, fornecendo:
- Registro automático de serviços do ORM no container
- Suporte a migrations, entidades anotadas, repositórios customizados
- Health check, métricas e monitoramento
- Facilidade para CRUD e integração RESTful

## Dependências

- PHP >= 8.1
- pivotphp/core >= 2.1.1
- cycle/orm >= 2.10
- cycle/annotated, cycle/migrations, cycle/schema-builder

## Estrutura Recomendada de Pastas

```
config/
    app.php
    cycle.php
src/
    Entities/
        User.php
    Repositories/
        UserRepository.php
public/
    index.php
var/
    db.sqlite
```

## 1. Instalação

Adicione os pacotes ao seu projeto:

```bash
composer require pivotphp/cycle-orm cycle/orm cycle/annotated cycle/migrations
```

## 2. Configuração Básica

No seu `config/cycle.php`:

```php
return [
    'database' => [
        'default' => 'default',
        'connections' => [
            'default' => [
                'driver' => 'sqlite',
                'database' => __DIR__ . '/../var/db.sqlite',
            ],
        ],
    ],
    'entities' => [
        'paths' => [__DIR__ . '/../src/Entities'],
    ],
];
```

No `config/app.php`, adicione o provider:

```php
return [
    'providers' => [
        PivotPHP\CycleORM\CycleServiceProvider::class,
    ],
];
```

## 3. Criando uma Entidade

Em `src/Entities/User.php`:

```php
namespace App\Entities;

use Cycle\Annotated\Annotation as Cycle;

/**
 * @Cycle\Entity(repository=App\Repositories\UserRepository::class)
 */
class User
{
    /** @Cycle\Column(type="primary") */
    public int $id;

    /** @Cycle\Column(type="string") */
    public string $name;
}
```

## 4. Migration

Não há binário `vendor/bin/cycle` neste pacote. `SchemaCommand`/`MigrateCommand` são
classes PHP simples — invoque-as a partir de um script `bin/console` próprio (veja
[integration-guide.md](../integration-guide.md#-comandos-cli) para um exemplo completo):

```php
(new SchemaCommand([], $container))->handle();
(new MigrateCommand([], $container))->handle();
```

## 5. Uso no Controller/Service

```php
use Cycle\ORM\ORMInterface;
use App\Entities\User;

// Injeção via container
$orm = $app->make(ORMInterface::class);
$repo = $orm->getRepository(User::class);

// Criar
$user = new User();
$user->name = 'Alice';
$em = $app->make('cycle.em');
$em->persist($user);
$em->run();

// Buscar
$found = $repo->findByPK($user->id);

// Atualizar
$found->name = 'Bob';
$em->persist($found);
$em->run();

// Remover
$em->delete($found);
$em->run();
```

## 6. Repositórios Customizados

```php
namespace App\Repositories;

use Cycle\ORM\Select\Repository;

class UserRepository extends Repository
{
    public function findByName(string $name): ?object
    {
        return $this->findOne(['name' => $name]);
    }
}
```

## 7. Health Check

```php
use PivotPHP\CycleORM\Health\CycleHealthCheck;

$result = CycleHealthCheck::check($app);
if ($result['cycle_orm'] !== 'healthy') {
    // Trate problemas de conexão/configuração
}
```

## 8. Monitoramento e Métricas

```php
use PivotPHP\CycleORM\Monitoring\MetricsCollector;

MetricsCollector::increment('entities_persisted');
$metrics = MetricsCollector::getMetrics();
```

## Exemplo de Endpoint HTTP

```php
// public/index.php
use PivotPHP\Core\Core\Application;
use App\Entities\User;
use Cycle\ORM\ORMInterface;

$app = new Application();

$app->get('/users/{id}', function ($req, $res, $args) use ($app) {
    $orm = $app->make(ORMInterface::class);
    $repo = $orm->getRepository(User::class);
    $user = $repo->findByPK($args['id']);
    if (!$user) {
        return $res->withStatus(404)->write('Usuário não encontrado');
    }
    return $res->json($user);
});

$app->run();
```

## Ciclo de Vida e Dicas de Debug

- O provider registra `cycle.database`, `cycle.orm`, `cycle.em` no container PivotPHP.
- Não há comando `artisan` neste pacote (convenção de outro framework). Instancie `SchemaCommand`/`MigrateCommand` diretamente para atualizar o banco.
- Para debug, ative logs SQL no Cycle ORM ou use o `MetricsCollector` para monitorar queries.
- Para resetar métricas em testes: `MetricsCollector::reset();`

## Troubleshooting

- **Erro: Unable to resolve role**: Verifique se a entidade está registrada em `config/cycle.php` e se o caminho está correto.
- **Erro de migration**: Certifique-se de invocar `SchemaCommand`/`MigrateCommand` (via seu `bin/console`) após criar/alterar entidades.
- **Problemas de autoload**: Rode `composer dump-autoload -o` após criar novas classes.
- **Problemas com SQLite**: Verifique permissões da pasta `var/` e se o arquivo `db.sqlite` existe.
- **Testes de integração**: Veja exemplos em `tests/Integration/FullIntegrationTest.php`.

## Links Úteis

- [PivotPHP](https://github.com/PivotPHP/pivotphp-core)
- [Cycle ORM](https://cycle-orm.dev/docs/intro/2.x/en)
- [Cycle ORM Annotations](https://cycle-orm.dev/docs/annotated/2.x/en)
- [Cycle ORM Migrations](https://cycle-orm.dev/docs/migrations/2.x/en)
- [Exemplo de projeto PivotPHP + Cycle ORM](https://github.com/PivotPHP/pivotphp-cycle-orm)

---

Para mais exemplos, veja os testes de integração em `tests/Integration/FullIntegrationTest.php`.
