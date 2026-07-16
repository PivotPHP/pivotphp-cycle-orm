# CycleHealthCheck

O `CycleHealthCheck` executa verificações de saúde do Cycle ORM e seus serviços, fornecendo informações detalhadas para monitoramento e diagnóstico.

## Visão Geral
Realiza checks automáticos de serviços essenciais, conexão com banco, integridade do schema e performance, retornando um relatório estruturado.

## Métodos
- `check($app)`: Verifica saúde geral (serviços, banco, schema, performance).
- `detailedCheck($app)`: Retorna informações detalhadas.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Health\CycleHealthCheck;

$status = CycleHealthCheck::check($app);
```

## Boas Práticas
- Utilize para monitoramento interno e endpoints de health check.
- Integre com sistemas de monitoramento externos e dashboards.

## Integração
Combine com o `HealthCheckMiddleware` para expor endpoints REST de health check.
