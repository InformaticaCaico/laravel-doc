# # Manipulação de erros

## # **Introdução**

Quando você começa um novo projeto Laravel, o tratamento de exceções já está configurado para você. A classe ```App\Exceptions\Handler```  está onde 
todas as exceções lançadas pelo seu aplicativo são registradas e renderizadas para o usuário. Nós iremos explorar mais a fundo essa classe ao longo dessa documentação. 

## # **Configuração**

A opção `debug`  em seu arquivo de configuração `config/app.php` determina quanta informação sobre um erro está atualmente exibida para o usuário. Por padrão, essa opção 
é configurado para respeitar o valor da variável de ambiente `APP_DEBUG`, que é armazenada em seu arquivo `.env`.

Durante o desenvolvimento local, você deve definir a variável de ambiente `APP_DEBUG` como `true`. **Em seu ambiente de produção, esse valor deve sempre ser `false`. Se o valor for definido como `true` na produção, você corre o risco de expor valores de configuração confidenciais aos usuários finais do seu aplicativo.**

## # O manipulador de exceções

### # Reportando exceções

Todas as exceções são manipuladas pela classe `App\Exceptions\Handler`. Essa classe contém um método `register` onde você deve registrar a o relatório de exceções personalizadas e renderizar retornos de chamada. Nós examinaremos cada um desses conceitos em detalhe. O relatório de exceção é usado para registrar exceções ou enviá-las para um serviço externo como [Flare](https://flareapp.io/), [Bugsnag](https://bugsnag.com/) ou [Sentry](https://github.com/getsentry/sentry-laravel). Por padrão, exceções vão ser registradas baseadas em sua configuração de [logging](https://laravel.com/docs/9.x/logging). De qualquer forma, você é livre para registrar exceções como você desejar.

Por exemplo, se você precisa reportar diferentes tipos de exceções de diferentes jeitos. você deve usar o método `reportable` para registrar um encerramento que deveria ter sido executado quando uma exceção de um determinado tipo que precisa ser reportado. O Laravel deduzirá que tipo de exceção o fechamento relata examinando o type-hint:

``` php
use App\Exceptions\InvalidOrderException;
 
/**
 * Register the exception handling callbacks for the application.
 *
 * @return void
 */
public function register()
{
    $this->reportable(function (InvalidOrderException $e) {
        //
    });
} 
```

Quando você registrar um callback de relaório de exceção personalizado usando métodoo `reportable`, o Laravel ainda registrará a exceção usando a configuração de registro padrão para o aplicativo. Se você deseja interromper a propagação da exceção para a pilha de registro padrão, você pode usar o método `stop` ao definir seu retorno de chamada de relatório ou retornar `false` do retorno de chamada:

``` php
$this->reportable(function (InvalidOrderException $e) {
    //
})->stop();
 
$this->reportable(function (InvalidOrderException $e) {
    return false;
});
```

> Para personalizar o relatório de exceções para um exceção específica, você deve utilizar exceções reportáveis

### # Contexto de Registro Global

Se disponível, o Laravel automaticamente vai adicionar o ID do usuário atual para cada mensagem de registro de exceção como dado contextual. Você deve definir seu próprio dado contextual global substituindo o método `context`  da classe `App\Exceptions\Handler` da sua aplicação. Essa informação vai ser incluída em cada mensagem de registro de exceção escrita pela sua aplicação.

```php
/**
 * Get the default context variables for logging.
 *
 * @return array
 */
protected function context()
{
    return array_merge(parent::context(), [
        'foo' => 'bar',
    ]);
}
```

### # Contexto de Registro de Exceção

Embora adicionar o contexto para cada mensagem de registro possa ser útil, às vezes uma exceção particular pode ter um contexto único que você gostaria de incluir em seus registros. Ao definir o método `context` de uma das exceções personalizadas da sua aplicação, você pode especificar qualquer dado relevante para a aplicação que você gostaria de adicionar à entrada de registro de exceção:

```php
<?php
 
namespace App\Exceptions;
 
use Exception;
 
class InvalidOrderException extends Exception
{
    // ...
 
    /**
     * Get the exception's context information.
     *
     * @return array
     */
    public function context()
    {
        return ['order_id' => $this->orderId];
    }
}
```

### # O Helper `report`

Às vezes você pode precisar relatar uma exceção mas continuar usando a requisição atual.  A função helper `report` permite a você reportar rapidamente uma exceção atravès da manipulação da exceção sem exibir e ocupar uma página de erro para o usuário:

```php
public function isValid($value)
{
    try {
        // Validate the value...
    } catch (Throwable $e) {
        report($e);
 
        return false;
    }
}
```

## # Níveis de registro de exceção

Quando as mensagens são gravadas nos [logs](https://laravel.com/docs/9.x/logging) do seu aplicativo, as mensagens são gravadas em um nível de registro especificado, que indica a gravidade ou importância da mensagem que está sendo registrada.

Como observado acima, mesmo quando você registra um retorno de chamada de relatório de exceção personalizado usando o método `reportable`, o Laravel ainda registrará a exceção usando a configuração de registro padrão para o aplicativo; no entanto, como o nível de registro às vezes pode influenciar os canais nos quais uma mensagem é registrada, você pode querer configurar o nível de registro em que certas exceções são registradas.

Para fazer isso, você pode definir uma array de tipos de exceção e seus níveis de registro associados dentro da propriedade ```$levels``` do manipulador de exceção do seu aplicativo:

```php
use PDOException;
use Psr\Log\LogLevel;
 
/**
 * A list of exception types with their corresponding custom log levels.
 *
 * @var array<class-string<\Throwable>, \Psr\Log\LogLevel::*>
 */
protected $levels = [
    PDOException::class => LogLevel::CRITICAL,
];
```

## # Ignorando as Exceções Por Modelo

Quando construímos a aplicação, haverá alguns tipos de exceção que você irá ignorar e nunca reportar. Sua manipulação da exceção contém uma propriedade `$dontReport`, propriedade que é inicializada com uma array vazia. Qualquer classe que você adicionar nesta propriedade nunca vai ser reportada. Contudo, eles podem ter uma lógica de renderização personalizada:

```php
use App\Exceptions\InvalidOrderException;
 
/**
 * A list of the exception types that are not reported.
 *
 * @var array<int, class-string<\Throwable>>
 */
protected $dontReport = [
    InvalidOrderException::class,
];
```

> Nos bastidores, o Laravel já ignora alguns tipos de erros para você, como exceções resultantes de erros 404 HTTP "não encontrados" ou respostas HTTP 419 geradas por tokens CSRF inválidos.

## # Exceções de renderização

Por padrão, o manipulador de exceção do Laravel converterá exceções em uma resposta HTTP para você. No entanto, você pode registrar uma closure de renderização personalizada para exceções de um determinado tipo. Você pode fazer isso através do método `renderable` do seu manipulador de exceção.

A closure passada para o `renderable` deve retornar uma instância de `Illuminate\Http\Response`, que pode ser gerada por meio do `renderable`. O Laravel deduzirá que tipo de exceção a clojure renderiza examinando a dica de tipo do encerramento:

```php
use App\Exceptions\InvalidOrderException;
 
/**
 * Register the exception handling callbacks for the application.
 *
 * @return void
 */
public function register()
{
    $this->renderable(function (InvalidOrderException $e, $request) {
        return response()->view('errors.invalid-order', [], 500);
    });
}
```

Você também pode usar o método `renderable` para substituir o comportamento de renderização para exceções internas do Laravel ou Symfony, como `NotFoundHttpException`. Se o encerramento dado ao método `renderable` não retornar um valor, a renderização de exceção padrão do Laravel será utilizada:

```php
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
 
/**
 * Register the exception handling callbacks for the application.
 *
 * @return void
 */
public function register()
{
    $this->renderable(function (NotFoundHttpException $e, $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
}
```

## # Exceções reportáveis ​​e renderizáveis

Em vez de verificar exceções no médoto `register` do manipulador de exceções, você pode definir métodos `report` e `render` em suas exceções personalizadas. Quando esses métodos existirem, eles serão chamados automaticamente pelo framework:

```php
<?php
 
namespace App\Exceptions;
 
use Exception;
 
class InvalidOrderException extends Exception
{
    /**
     * Report the exception.
     *
     * @return bool|null
     */
    public function report()
    {
        //
    }
 
    /**
     * Render the exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        return response(/* ... */);
    }
}
```

Se sua exceção estender uma exceção que já é renderizável, como uma exceção integrada do Laravel ou do Symfony, você pode retornar `false` do método `render` da exceção para renderizar a resposta HTTP padrão da exceção:

```php
/**
 * Render the exception into an HTTP response.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function render($request)
{
    // Determine if the exception needs custom rendering...
 
    return false;
}
```

Se a sua exceção contiver uma lógica de relatório personalizada que só é necessária quando certas condições são atendidas, você pode precisar instruir o Laravel a algumas vezes relatar a exceção usando a configuração padrão de manipulação de exceção. Para fazer isso, você pode retornar `false` do método `report` da exceção:

```php
/**
 * Report the exception.
 *
 * @return bool|null
 */
public function report()
{
    // Determine if the exception needs custom reporting...
 
    return false;
}
```

> Você pode digitar quaisquer dependências necessárias do método de relatório e elas serão automaticamente injetadas no método pelo [contêiner de serviço](https://laravel.com/docs/9.x/container) do Laravel.

# # **Exceções do HTTP**

Algumas exceções descrevem códigos de erro HTTP do servidor. Por exemplo, isso pode ser um erro de "página não encontrada" (404), um "erro não autorizado" (401) ou até mesmo um erro 500 gerado pelo desenvolvedor. Para gerar tal resposta de qualquer lugar em seu aplicativo, você pode usar o helper `abort`:

```php
abort(404);
```


## # Personalização das Páginas de Erro do HTTP

O Laravel facilita a exibição de páginas de erro personalizadas para vários códigos de status HTTP. Por exemplo, se você deseja customizar a página de erro para o código de status HTTP crie um modelo de visualização `resources/views/errors/404.blade.php`. Essa visualização vai ser renderizada em todos os erros 404 gerados pela sua aplicação. A visualização dentro desse diretório deve ser nomeado para corresponder ao código de status HTTP a qual eles correspondem. A instância `Symfony\Component\HttpKernel\Exception\HttpException` criada pela função `abort` vai ser passada para a visualização como a variável `$exception`.

```php
<h2>{{ $exception->getMessage() }}</h2>
```

Você pode publicar os modelos de página de erro padrão do Laravel usando o comando artisan vendor:publish. Uma vez que os templates tenham sido publicados, você pode personalizá-los conforme o seu gosto. 

```php
php artisan vendor:publish --tag=laravel-errors
```

#### # Páginas de erro HTTP de fallback

Você também pode definir uma página de erro "fallback" para uma determinada série de códigos de status HTTP. Esta página será renderizada se não houver uma página correspondente para o código de status HTTP específico que ocorreu. Para fazer isso, defina um template `4xx.blade.php` e um `5xx.blade.php` no diretório do seu aplicativo `resources/views/errors`.
