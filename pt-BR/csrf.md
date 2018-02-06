# Proteção CSRF

- [Introdução](#csrf-introduction)
- [Excluindo URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Introdução

O Laravel facilita a proteção da sua aplicação contra ataques [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Os ataques CSRF são um tipo de exploração maliciosa, onde os comandos não autorizados são realizados em nome de um usuário autenticado.

O Laravel gera automaticamente um token "CSRF" para cada sessão de usuário ativa, que é gerenciada pela aplicação. Este token é usado para verificar se o usuário autenticado é aquele que realmente faz as requisições na aplicação.

Sempre que você definir um formulário HTML na sua aplicação, você deve incluir um campo token CSRF oculto no formulário para que o middleware de proteção CSRF possa validar a solicitação. Você pode ajudar o _helper_ `csrf_field` para gerar o campo token:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

O [middleware](/docs/{{version}}/middleware) `VerifyCsrfToken`, que está adicionado ao grupo de middleware `web`, verificará automaticamente se o token na requisição corresponde ao token armazenado na sessão.

#### Tokens CSRF & JavaScript

Ao criar aplicações JavaScript, é conveniente que sua biblioteca HTTP de JavaScript inclua automaticamente o Token CSRF a todas as requisições de saída. Por padrão, o arquivo `resources/assets/js/bootstrap.js` registra o valor da meta tag `csrf-token` com a biblioteca Axios HTTP. Se você não estiver usando esta biblioteca, você precisará configurar esse comportamento manualmente para sua aplicação.

<a name="csrf-excluding-uris"></a>
## Excluindo URIs da proteção CSRF

Às vezes, você pode querer excluir um conjunto de URIs para proteção CSRF. Por exemplo, se você está usando o [Stripe](https://stripe.com) para processar pagamentos e estiver usando seu sistema de webhooks, você precisará excluir a rota do Stripe, uma vez que ele não saberá o token CSRF para enviar para suas rotas.

Normalmente, você deve colocar essas rotas fora do grupo middleware `web` que o `RouteServiceProvider` aplica para todas as rotas no arquivo `routes/web.php`. Entretanto, você pode excluir as rotas adicionando as URIs na propriedade `$except` no middleware `VerifyCsrfToken`:

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
     protected $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
     ];
}
```
<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Além de verificar o token CSRF como parâmetro na requisição POST, o middleware `VerifyCsrfToken` também verificará o cabeçalho `X-CSRF-TOKEN` da requisição. Você poderia, por exemplo, adicionar o token em uma `meta` tag no HTML:

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Uma vez criado a `meta` tag, você pode instruir uma biblioteca, como por exemplo jQuery, para adicionar automaticamente o token para todos os cabeçalhos das requisições. Isso fornece uma proteção CSRF simples e conveniente para sua aplicação.

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

> Dica: Por padrão, o arquivo `resources/assets/js/bootstrap.js` registra o valor da meta tag `csrf-token` na biblioteca Axios HTTP. Se você não estiver usando esta biblioteca, você precisará configurar esse comportamento manualmente para sua aplicação.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

O Laravel armazena o token CSRF no cookie `XSRF-TOKEN` que está incluído em cada requisição gerada pelo framework. Você pode usar o valor do cookie para definir o cabeçalho `X-XSRF-TOKEN` da requisição.

Este cookie é enviado principalmente como uma conveniência, uma vez que algumas estruturas e bibliotecas JavaScript, como Angular e Axios, colocam automaticamente seu valor no cabeçalho `X-XSRF-TOKEN`.
