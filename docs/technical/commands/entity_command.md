# Comando: EntityCommand

O comando `EntityCommand` gera novas entidades para o Cycle ORM, padronizando a estrutura e facilitando a manutenção do domínio da aplicação.

## Visão Geral
Permite criar rapidamente classes de entidades seguindo as convenções do projeto, reduzindo erros e acelerando o desenvolvimento.

## Exemplo de Uso
Não há `bin/console` incluído no pacote — instancie a classe diretamente:
```php
(new EntityCommand(['name' => 'User'], $container))->handle();
```

## Métodos Principais
- `handle()`: Executa a criação da entidade.
- `generateEntityContent($className)`: Gera o conteúdo da classe de entidade.
- `getEntityPath($className)`: Retorna o caminho do arquivo da entidade.
- `getTableName($className)`: Gera o nome da tabela a partir do nome da classe.

## Boas Práticas
- Use nomes de entidades no singular e com inicial maiúscula.
- Gere entidades sempre via comando para garantir padrão e consistência.
- Versione as entidades junto com o código-fonte.

## Integração
O `EntityCommand` pode ser utilizado em pipelines de scaffolding e automação de projetos PivotPHP.
