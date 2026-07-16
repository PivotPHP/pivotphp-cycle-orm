# Comando: MigrateCommand

O comando `MigrateCommand` executa e reverte migrações do Cycle ORM, permitindo evoluir ou desfazer alterações no schema do banco de dados de forma controlada.

## Visão Geral
Esse comando é essencial para gerenciar a evolução do banco de dados em ambientes de desenvolvimento, homologação e produção.

## Exemplos de Uso
Não há `bin/console` incluído no pacote — instancie a classe diretamente:
```php
(new MigrateCommand([], $container))->handle();
(new MigrateCommand(['--rollback' => true], $container))->handle();
```

## Opções Disponíveis
- `--rollback`: Reverte a última migração.

## Métodos Principais
- `handle()`: Executa o comando principal. Se a opção `--rollback` for usada, reverte a última migração.
- `migrate()`: Executa as migrações pendentes.
- `rollback()`: Reverte a última migração.

## Boas Práticas
- Execute as migrações sempre que houver alterações no schema.
- Use rollback para desfazer rapidamente alterações recentes.
- Mantenha scripts de migração versionados no repositório.

## Integração
O `MigrateCommand` pode ser integrado a pipelines de CI/CD para automação de deploys e rollback em caso de falhas.
