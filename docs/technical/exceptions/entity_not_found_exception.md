# Exception: EntityNotFoundException

A exceção `EntityNotFoundException` é lançada quando uma entidade não é encontrada no banco de dados, permitindo tratamento específico para recursos inexistentes.

## Visão Geral
Utilizada principalmente em repositórios e serviços de domínio, facilita o retorno de respostas 404 em APIs REST e logs detalhados de falhas de busca.

## Propriedades
- `entityClass`: Classe da entidade buscada.
- `identifier`: Identificador utilizado na busca.

## Métodos
- `getEntityClass()`: Retorna a classe da entidade.
- `getIdentifier()`: Retorna o identificador utilizado.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Exceptions\EntityNotFoundException;

throw new EntityNotFoundException(User::class, 1);
```

## Boas Práticas
- Capture esta exceção para retornar respostas 404 em APIs REST.
- Utilize logs detalhados para auditoria de falhas de busca.

## Integração
Combine com middlewares de tratamento de erro para respostas automáticas e padronizadas.
