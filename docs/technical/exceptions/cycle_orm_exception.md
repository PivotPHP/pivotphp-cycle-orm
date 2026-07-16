# Exception: CycleORMException

A exceção `CycleORMException` é a base para erros relacionados ao Cycle ORM Extension, permitindo tratamento global e enriquecimento de logs com contexto adicional.

## Visão Geral
Utilizada como superclasse para todas as exceptions do pacote, facilita o tratamento centralizado de falhas e a análise de contexto em logs e sistemas de monitoramento.

## Propriedades
- `context`: Array de contexto adicional sobre o erro.

## Métodos
- `getContext()`: Retorna o contexto do erro.
- `setContext(array $context)`: Define o contexto do erro.
- `addContext(string $key, mixed $value)`: Adiciona uma informação ao contexto.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Exceptions\CycleORMException;

throw new CycleORMException('Erro ao conectar', 0, null, ['component' => 'database']);
```

## Boas Práticas
- Utilize o contexto para enriquecer logs e facilitar o diagnóstico.
- Sempre capture `CycleORMException` em handlers globais de erro.

## Integração
Combine com sistemas de logging estruturado e APM para rastreamento de falhas.
