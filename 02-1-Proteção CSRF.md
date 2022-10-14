<style>
    span{
        color: #eb4432;
    }
    .underline{
        text-decoration: underline;
    }
</style>

# **Proteção de CSFR**

# <span>\#</span> Introdução

Falsificação de pedidos entre sites são um tipo de exploração maliciosa usando comandos desautorizados que são feitos no lugar de um usuário autenticado. Felizmente, Laravel faz com que seja fácil  proteger sua aplicação de ataques usando a proteção de <span class="underline">**[Falsificação de Pedidos entre Sites](https://en.wikipedia.org/wiki/Cross-site_request_forgery)**</span> (CSRF).

## <span>\#</span> Uma Explicação da Vunerabilidade

Caso você não seja familiarizado com a falsificação de requisiçoes em sites, vamos discutir um exemplo de como essa vulnerabilidade pode ser explorada. Imagine que sua aplicação tenha uma rota <span class="underline">`/user/email`</span> que aceite uma requisção <span>`POST`</span> para mudar o endereço de email do usuário autenticado. Provavelmente, essa rota espera um campo de entrada de <span>`email`</span> que contenha o endereço de email de usuário que gostaria de usar.

Sem a proteção de CSRF , um website malicioso poderia criar um formulário HTML que aponta para a rota <span class="underline"><code>/user/email</code></span> da sua aplicação e enviar o endereço de e-mail do próprio usuário malicioso:

```html
<form action="https://your-application.com/user/email" method="POST">
  <input type="email" value="malicious-email@example.com" />
</form>

<script>
  document.forms[0].submit();
</script>
```

Se o website automaticamente enviar o formulário quando a página é carregada, o usuário malicioso apenas precisa atrair um usuário sem suspeitas de sua aplicação para visitar o website dele e o endereço de email dele será mudado na sua aplicação.

Para prevenir essa vulnerabilidade, nós precisamos inspecionar cada request <span>`POST`</span>, <span>`PUT`</span>, <span>`PATCH`</span>, ou <span>`DELETE`</span> e analisar o valor secreto de sessão que a aplicação maliciosa é incapaz de acessar.

<br>

# <span>\#</span> Impedindo pedidos CSRF

O Laravel automaticamente gera“tokens” CRSF para cada <span class="underline">**[user session](https://laravel.com/docs/9.x/session)**</span> (sessão de usuário) ativa e gerenciada pela aplicação. Esse Token é usado para verificar se o usuário autenticado é a pessoa realmente fazendo a requisição para a aplicação. Uma vez que esse token esteja guardado na sessão de usuário e mude toda vez que a sessão seja regenerada, uma aplicação maliciosa é incapaz de acessá-la.

A atual token CSRF da sessão pode ser acessada via a sessão da requisição ou via a função helper <span>`csrf_token`</code></span>:

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Toda vez que você define um formulário HTML “POST”, “PUT”, “PATCH”, ou “DELETE” em sua aplicação, você deveria incluir um campo CSRF <span>`_token`</span> escondido no formulário para que o middleware de proteção CSRF possa validar a requisição. Por conveniência, você talvez use o diretiva Blade <span>`@csrf`</span> para gerar o campo de entrada do token escondido:

```html
<form method="POST" action="/profile">
  @csrf

  <!-- Equivalent to... -->
  <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

O <span class="underline">[middleware](https://laravel.com/docs/9.x/middleware)</span> <span>`App\Http\Middleware\VerifyCsrfToken`</span>, que é incluído no grupo middleware <span>`web`</span> por padrão, vai automaticamente verificar que o token na requisição do usuário combina com o token guardado na sessão. Quando esses dois tokens combinam, nós sabemos que o usuário autenticado é aquele que iniciou o pedido.

<br>

# <span>\#</span> CSRF Tokens e SPAs

Se você está construindo uma SPA que está utilizando Laravel como API back-end, você deveria consultar o <span class = "underline">[Laravel Sanctum documentation](https://laravel.com/docs/9.x/sanctum)</span> por informações em autenticações com sua API e proteger sua aplicação contra vulnerabilidades CSRF.

<br>

# <span>\#</span>Excluindo URI’s da proteção CSRF

Às vezes você pode querer excluir um conjunto de URI’s da proteção CSRF. Por exemplo, se você está usando o <span class = "underline">[Stripe](https://stripe.com/en-br)</span> para processar pagamentos e está usando o sistema de webhook deles, você precisará excluir sua rota do manipulador do webhook do Stripe da proteção CSRF, pois o Stripe não saberá qual token CSRF enviar para suas rotas.

Normalmente, você deve colocar esses tipos de rotas fora do grupo de rotas <span>`web`</span> do middleware <span>`App\Providers\RouteServiceProvider`</span> que se aplica a todas as rotas no arquivo <span>`routes/web.php`</span>. No entanto, você também pode excluir as rotas adicionando seus URI’s à propriedade <span>`$except`</span> do <span>`VerifyCsrfToken`</span> middleware:

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

> Por conveniência, o middleware CSRF é desabilitado automaticamente para todas as rotas ao <span class="underline">[executar testes](https://laravel.com/docs/9.x/testing)</span>.

<br>

# <span>\#</span> X-CSRF-TOKEN

Além de verificar o token CSRF como um parâmetro POST, o <span>`App\Http\Middleware\VerifyCsrfToken`</span> middleware também verificará o cabeçalho <span>`X-CSRF-TOKEN`</span> da solicitação. Você pode, por exemplo, armazenar o token em uma <span>`meta`</span> tag HTML:

```html
<meta name="csrf-token" content="{{ csrf_token() }}" />
```

Em seguida, você pode instruir uma biblioteca como jQuery para adicionar automaticamente o token a todos os cabeçalhos de solicitação. Isso fornece proteção CSRF simples e conveniente para seus aplicativos baseados em AJAX usando tecnologia de legado do JavaScript:

```php
$.ajaxSetup({
  headers: {
    "X-CSRF-TOKEN": $('meta[name="csrf-token"]').attr("content"),
  },
});
```

<br>

# <span>\#</span> X-XSRF-TOKEN

O Laravel armazena o token CSRF atual em um cookie <span>`XSRF-TOKEN`</span> criptografado que é incluído em cada resposta gerada pela framework. Você pode usar o valor do cookie para definir o cabeçalho <span>`X-XSRF-TOKEN`</span> da solicitação.

Esse cookie é enviado principalmente como uma conveniência do desenvolvedor, pois alguns frameworks e bibliotecas JavaScript, como Angular e Axios, colocam automaticamente seu valor no cabeçalho <span>`X-XSRF-TOKEN`</span> em solicitações de mesma origem.

> Por padrão, o arquivo <span>`resources/js/bootstrap.js`</span> inclui a biblioteca HTTP Axios que enviará automaticamente o cabeçalho <span>`X-XSRF-TOKEN`</span> para você.
