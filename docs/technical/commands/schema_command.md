# Comando: SchemaCommand

O comando `SchemaCommand` permite exibir e sincronizar o schema do Cycle ORM, garantindo que o banco de dados esteja alinhado com as entidades do seu projeto.

## Visão Geral
Esse comando é fundamental para manter a integridade do schema, especialmente após alterações em entidades ou configurações do ORM.

## Exemplos de Uso
Não há `bin/console` incluído no pacote — instancie a classe diretamente (veja
[integration-guide.md](../../integration-guide.md#-comandos-cli) para um script completo):
```php
(new SchemaCommand([], $container))->handle();
(new SchemaCommand(['--sync' => true], $container))->handle();
```

## Opções Disponíveis
- `--sync`: Sincroniza o schema com o banco de dados.

## Métodos Principais
- `handle()`: Executa o comando principal. Se a opção `--sync` for usada, sincroniza o schema com o banco de dados.
- `syncSchema()`: Sincroniza o schema do banco de dados.
- `showSchema()`: Exibe informações do schema atual.

## Boas Práticas
- Utilize este comando após alterações em entidades para garantir que o banco esteja sincronizado.
- Sempre faça backup do banco antes de sincronizar schemas em produção.

## Integração
O `SchemaCommand` pode ser utilizado em pipelines de CI/CD para automação de deploys e validação de consistência do banco.
