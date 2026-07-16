# Comando: StatusCommand

O comando `StatusCommand` verifica o status de saúde do Cycle ORM e do banco de dados, fornecendo informações essenciais para monitoramento e diagnóstico.

## Visão Geral
Esse comando executa uma série de verificações automáticas, como conexão com o banco, integridade do schema e performance, retornando um relatório detalhado.

## Exemplo de Uso
Não há `bin/console` incluído no pacote — instancie a classe diretamente. Requer que
uma função global `app()` esteja disponível (usada internamente por `CycleHealthCheck`):
```php
(new StatusCommand([], $container))->handle();
```

## Métodos Principais
- `handle()`: Executa a verificação de saúde e exibe o status.
- `displayHealthStatus(array $health)`: Exibe o resultado detalhado da verificação.

## Saída
- Mostra status geral, tempo de resposta, detalhes de cada verificação e ícones de sucesso/falha.

## Boas Práticas
- Use este comando para monitorar a saúde do ORM e do banco em produção.
- Automatize execuções periódicas para alertas proativos.

## Integração
O `StatusCommand` pode ser integrado a sistemas de monitoramento e health checks externos para garantir alta disponibilidade.
