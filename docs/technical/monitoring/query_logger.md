# QueryLogger

O `QueryLogger` é responsável por registrar todas as queries executadas pelo Cycle ORM, facilitando a auditoria, depuração e análise de performance do banco de dados.

## Visão Geral
O logger captura a query SQL, parâmetros utilizados e o tempo de execução, registrando essas informações no `error_log` do PHP ou em outro sistema de log configurado.

## Recursos
- Log detalhado de cada query executada.
- Registro de parâmetros e tempo de execução.
- Pode ser integrado a sistemas de log externos.

## Métodos
- `log(string $query, array $params = [], float $timeMs = 0.0)`: Registra uma query, seus parâmetros e o tempo de execução.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Monitoring\QueryLogger;

$logger = new QueryLogger();
$logger->log('SELECT * FROM users WHERE id = ?', [1], 2.5);
```

## Boas Práticas
- Utilize em ambientes de desenvolvimento para identificar queries ineficientes.
- Em produção, direcione logs para sistemas centralizados e monitore queries lentas.
- Combine com o `MetricsCollector` para análise completa de performance.

## Integração
O `QueryLogger` pode ser utilizado em conjunto com middlewares de logging, sistemas de monitoramento e ferramentas de APM para rastreamento de queries em tempo real.
