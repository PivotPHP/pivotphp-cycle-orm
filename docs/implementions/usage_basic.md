# Guia de Integração Básica com PivotPHP Cycle ORM Extension

Este guia mostra como integrar o Cycle ORM ao seu projeto PivotPHP de forma simples e rápida.

## Instalação

1. Instale o pacote via Composer:
```bash
composer require pivotphp/cycle-orm
```

2. Publique as configurações (opcional):
```bash
php bin/express-cycle publish:config
```

## Configuração Básica

No arquivo de configuração `config/cycle.php`, defina as conexões, entidades e repositórios.

## Uso Básico

- Utilize os repositórios para persistir e buscar entidades.
- Utilize os comandos CLI para gerenciar o banco de dados.

## Exemplo
```php
$userRepository = $container->get(UserRepository::class);
$user = $userRepository->findByPK(1);
```

Consulte a documentação técnica para detalhes avançados.
