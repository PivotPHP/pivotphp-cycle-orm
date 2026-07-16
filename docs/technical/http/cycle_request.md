# CycleRequest

O objeto `CycleRequest` é um wrapper que estende o `PivotPHP\Core\Http\Request` para integrar recursos do Cycle ORM diretamente na requisição, mantendo compatibilidade total com o objeto original.

## Visão Geral
Permite acessar ORM, EntityManager, Database, usuário autenticado e informações de autenticação diretamente na requisição, facilitando o desenvolvimento de middlewares e handlers avançados.

## Mudanças e Recursos Adicionados
- **Compatibilidade total** com o objeto `Request` original.
- **Propriedades adicionais**:
  - `orm`: Instância do ORM do Cycle.
  - `em`: EntityManager do Cycle.
  - `db`: DatabaseInterface do Cycle.
  - `user`: Usuário autenticado (opcional).
  - `auth`: Array de informações de autenticação.
- **Encaminhamento automático** de métodos e propriedades dinâmicas para o objeto original.

## Exemplo de Uso
```php
use PivotPHP\CycleORM\Http\CycleRequest;

function handle(CycleRequest $request) {
    $user = $request->user;
    $repo = $request->orm->getRepository(User::class);
    // ...
}
```

## Boas Práticas
- Utilize o `CycleRequest` em middlewares e handlers para acessar ORM e autenticação de forma centralizada.
- Não modifique o objeto original diretamente, utilize as propriedades expostas pelo wrapper.
- Documente handlers e middlewares para garantir clareza no uso das propriedades extras.

## Integração
Combine o `CycleRequest` com middlewares de autenticação, validação e logging para pipelines robustos e seguros.
