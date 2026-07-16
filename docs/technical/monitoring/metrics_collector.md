# MetricsCollector

O `MetricsCollector` é responsável por coletar e expor métricas detalhadas de uso e performance do Cycle ORM, facilitando o monitoramento e a análise de gargalos em aplicações PivotPHP.

## Visão Geral
O coletor centraliza estatísticas como número de queries executadas, tempo total de execução, entidades persistidas, cache hits/misses e queries lentas, permitindo integração com sistemas de observabilidade e tuning de performance.

## Métricas Coletadas
- **queries_executed**: Total de queries executadas.
- **queries_failed**: Total de queries que falharam.
- **total_query_time**: Tempo total (ms) gasto em queries.
- **entities_persisted**: Quantidade de entidades persistidas.
- **entities_loaded**: Quantidade de entidades carregadas.
- **cache_hits**: Quantidade de acertos de cache.
- **cache_misses**: Quantidade de falhas de cache.
- **slow_queries**: Quantidade de queries lentas (>100ms).

## Recursos Adicionais
- **slowQueries**: Lista das últimas 10 queries lentas, com query, tempo e timestamp.

## Métodos
- `increment(string $metric, int $value = 1)`: Incrementa o valor de uma métrica.
- `recordQueryTime(string $query, float $timeMs)`: Registra o tempo de execução de uma query e armazena queries lentas.
- `getMetrics(): array`: Retorna todas as métricas atuais.
- `getSlowQueries(): array`: Retorna a lista de queries lentas registradas.
- `reset()`: Reseta todas as métricas para zero.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Monitoring\MetricsCollector;

MetricsCollector::increment('entities_persisted');
MetricsCollector::recordQueryTime('SELECT * FROM users', 120.5);
$metrics = MetricsCollector::getMetrics();
$slow = MetricsCollector::getSlowQueries();
```

## Boas Práticas
- Monitore queries lentas e ajuste índices/tabelas conforme necessário.
- Integre as métricas com dashboards de observabilidade (ex: Prometheus, Grafana).
- Utilize o reset em cenários de testes automatizados.
- Analise entidades persistidas/carregadas para identificar possíveis otimizações.

## Integração
O `MetricsCollector` pode ser utilizado em conjunto com outros componentes de monitoring para fornecer uma visão completa da saúde e performance do ORM.
