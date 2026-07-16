# PerformanceProfiler

O `PerformanceProfiler` permite medir e analisar a performance de operações críticas do Cycle ORM, fornecendo dados detalhados sobre tempo, memória e queries executadas.

## Visão Geral
O profiler pode ser habilitado/desabilitado dinamicamente e permite criar "profiles" nomeados para diferentes trechos de código, facilitando a identificação de gargalos.

## Recursos
- Habilitar/desabilitar profiling em tempo de execução.
- Iniciar e finalizar profiles nomeados.
- Coleta tempo de execução, uso de memória e número de queries executadas.
- Permite análise comparativa entre diferentes operações.

## Métodos
- `enable()`: Habilita o profiler.
- `disable()`: Desabilita o profiler.
- `start(string $name)`: Inicia um profile nomeado.
- `end(string $name): array`: Finaliza o profile e retorna dados coletados.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Monitoring\PerformanceProfiler;

PerformanceProfiler::enable();
PerformanceProfiler::start('import_users');
// ... operação crítica ...
$profile = PerformanceProfiler::end('import_users');
print_r($profile);
```

## Boas Práticas
- Utilize para identificar gargalos em operações de importação, exportação ou processamento em lote.
- Analise o uso de memória e queries para otimizar algoritmos.
- Desabilite o profiler em produção para evitar overhead.

## Integração
O `PerformanceProfiler` pode ser integrado a sistemas de monitoramento e dashboards para análise histórica de performance.
