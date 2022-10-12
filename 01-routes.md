# **Rotas**

## # **Roteamento Básico**

O mais básico roteamento do Laravel aceita uma URI e um fechamento, fornecendo um método muito simples e expressivo de definir rotas e comportamento sem arquivos de configuração de roteamento complicados:

``` php
use Illuminate\Support\Facades\Route;
 
Route::get('/greeting', function () {
    return 'Hello World';
});
```
#### # **Os arquivos de rota padrão**
Toda rota no Laravel é definida dentro do arquivo de rotas, que é localizado no diretório `rotas/`. Esses arquivos são carregados automaticamente pelo `App\Providers\RouteServiceProvider` da sua aplicação. O arquivo `routes/web.php` define rotas para sua interface web. Essas rotas são atribuídas ao grupo de middleware da `web`, fornece recursos como *estado de sessão* e *proteção CSRF*. As rotas em `routes/api.php` não tem estado e são atribuídos o grupo de middleware `api`.

Para a maioria das aplicações, você começará definindo rotas no seu arquivo `routes/web.php`. As rotas definidas em `routes/web.php` podem ser acessada digitando a URL no seu browser. Por exemplo, você pode acessar a seguinte rota navegando no `http://example.com/user` de seu browser: 

``` php
use App\Http\Controllers\UserController;
 
Route::get('/user', [UserController::class, 'index']);
```
As rotas definidas no arquivo routes/api.php são aninhadas em um grupo de rotas pelo `RouteServiceProvider`. Dentro desse grupo, o prefixo /api URI é aplicado automaticamente para que você não precise aplicá-lo manualmente a todas as rotas do arquivo. Você pode modificar o prefixo e outras opções de grupo de rotas modificando sua classe `RouteServiceProvider`.

#### # **Métodos de roteador disponíveis**

O roteador permite registrar rotas que respondem a qualquer verbo HTTP:

``` php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```
Às vezes você pode precisar registrar uma rota que responda a vários verbos HTTP. Você pode fazer isso usando o método `match`. Ou você pode até registrar uma rota que responda a todos os verbos HTTP usando o método `any`:
```php
Route::match(['get', 'post'], '/', function () {
    //
});
 
Route::any('/', function () {
    //
});
```

> Ao definir várias rotas que compartilham o mesmo URI, as rotas que usam os métodos `get`, `post`, `put`, `patch`, `delete` e `options` devem ser definidas antes das rotas usando os métodos `any`, `match` e `redirect`. Isso garante que a solicitação recebida seja correspondida com a rota correta.

#### # **Injeção de dependência**
Você pode indicar quaisquer dependências exigidas pela sua rota na assinatura da função de *`callback`* da sua rota. As dependências declaradas serão automaticamente resolvidas e injetadas no callback pelo container de `serviço Laravel`. Por exemplo, você pode digitar a classe `Illuminate\Http\Request` para que a solicitação HTTP atual seja automaticamente injetada em seu retorno de chamada de rota:

```php
use Illuminate\Http\Request;
 
Route::get('/users', function (Request $request) {
    // ...
});
```

#### # **Proteção CSRF**
Lembre-se de que quaisquer formulários HTML apontando para rotas `POST`, `PUT`, `PATCH` ou `DELETE` definidos no arquivo de rotas da `web` devem incluir um campo de token CSRF. Caso contrário, o pedido será rejeitado. Você pode ler mais sobre a proteção CSRF no https://laravel.com/docs/9.x/csrf:
```php
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```
### # **Redirecionar Rotas**

Se você estiver definindo uma rota que redireciona para outro URI, você pode usar o método `Route::redirect`. Este método fornece um atalho conveniente para que você não precise definir uma rota ou controlador completo para realizar um redirecionamento simples:

```php
Route::redirect('/here', '/there');
```

Por padrão, `Route::redirect` retorna um código de status `302`. Você pode personalizar o código de status usando o terceiro parâmetro opcional:

```php
Route::redirect('/here', '/there', 301);
```

Ou, você pode usar o método `Route::permanentRedirect` para retornar um código de status `301`:
```php
Route::permanentRedirect('/here', '/there');
```

> Ao usar parâmetros de rota em rotas de redirecionamento, os seguintes parâmetros são reservados pelo Laravel e não podem ser usados: `destino` e `status`.


### # **Ver Rotas**
Se sua rota precisar apenas retornar uma `view`, você pode usar o método `Route::view`. Assim como o método de `redirecionamento`, esse método fornece um atalho simples para que você não precise definir uma rota ou controlador completo. O método de `view` aceita um URI como seu primeiro argumento e um nome de uma `view` como seu segundo argumento. Além disso, você pode fornecer um *array* de dados para passar para a `view` como um terceiro argumento opcional:

```php
Route::view('/welcome', 'welcome');
 
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

> Ao usar parâmetros de rota em rotas de view, os seguintes parâmetros são reservados pelo Laravel e não podem ser usados: `visualização`, `dados`, `status` e `cabeçalhos`.


### # **A Lista de Rotas**
O comando do Artinsa `route:list` fornece uma visão geral de todas as rotas definidas pelo seu aplicativo:

```php
php artisan route:list
```

Por padrão, o middleware de rota atribuído a cada rota não será exibido na saída route:list; no entanto, você pode instruir o Laravel a exibir o middleware de rota adicionando a opção -v ao comando:

```php
php artisan route:list -v
```

Você também pode instruir o Laravel a mostrar apenas as rotas que começam com um determinado URI:

```php
php artisan route:list --path=api
```

Além disso, você pode instruir o Laravel a ocultar quaisquer rotas definidas por pacotes de terceiros fornecendo a opção --except-vendor ao executar o comando route:list:

```php
php artisan route:list --except-vendor
```

Da mesma forma, você também pode instruir o Laravel a mostrar apenas as rotas definidas por pacotes de terceiros fornecendo a opção --only-vendor ao executar o comando route:list:

```php
php artisan route:list --only-vendor
```

## # **Parâmetros de Rota**

### # **Parâmetros obrigatórios**

Às vezes, você precisará capturar segmentos do URI em sua rota. Por exemplo, pode ser necessário capturar o ID de um usuário do URL. Você pode fazer isso definindo parâmetros de rota:

```php
Route::get('/user/{id}', function ($id) {
    return 'User '.$id;
});
```

Você pode definir quantos parâmetros de rota forem necessários para sua rota:

```php
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

Os parâmetros de rota são sempre colocados entre chaves {} e devem consistir em caracteres alfabéticos. Os sublinhados (_) também são aceitáveis em nomes de parâmetros de rota. Os parâmetros de rota são injetados em retornos de chamada/controladores de rota com base em sua ordem - os nomes dos argumentos de retorno de chamada/controlador de rota não importam.

#### # **Parâmetros e injeção de dependência**
Se sua rota tiver dependências que você gostaria que o contêiner de serviço Laravel injetasse automaticamente no retorno de chamada da sua rota, você deve listar seus parâmetros de rota após suas dependências:

```php
use Illuminate\Http\Request;
 
Route::get('/user/{id}', function (Request $request, $id) {
    return 'User '.$id;
});
```
### # **Parâmetros opcionais**

Ocasionalmente, pode ser necessário especificar um parâmetro de rota que nem sempre está presente no URI. Você pode fazer isso colocando um `?` marque após o nome do parâmetro. Certifique-se de dar um valor padrão à variável correspondente da rota:

```php
Route::get('/user/{name?}', function ($name = null) {
    return $name;
});
 
Route::get('/user/{name?}', function ($name = 'John') {
    return $name;
});
```
### # **Restrições de Expressão Regular**

Você pode restringir o formato de seus parâmetros de rota usando o método `where` em uma instância de rota. O método `where` aceita o nome do parâmetro e uma expressão regular que define como o parâmetro deve ser restringido:

```php
Route::get('/user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');
 
Route::get('/user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');
 
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

Por conveniência, alguns padrões de expressão regular comumente usados têm métodos auxiliares que permitem adicionar rapidamente restrições de padrão às suas rotas:

```php
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->whereNumber('id')->whereAlpha('name');
 
Route::get('/user/{name}', function ($name) {
    //
})->whereAlphaNumeric('name');
 
Route::get('/user/{id}', function ($id) {
    //
})->whereUuid('id');
 
Route::get('/category/{category}', function ($category) {
    //
})->whereIn('category', ['movie', 'song', 'painting']);
```
Se a solicitação recebida não corresponder às restrições do padrão de rota, uma resposta HTTP 404 será retornada.

#### # **Restrições Globais**

Se você quiser que um parâmetro de rota seja sempre restringido por uma determinada expressão regular, você pode usar o método `pattern`. Você deve definir esses padrões no método de `inicialização` de sua classe `App\Providers\RouteServiceProvider`:

```php
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::pattern('id', '[0-9]+');
}
```

Uma vez definido o padrão, ele é aplicado automaticamente a todas as rotas usando esse nome de parâmetro:

```php
Route::get('/user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```
#### # **Barras codificadas**

O componente de roteamento do Laravel permite que todos os caracteres, exceto /, estejam presentes nos valores dos parâmetros de rota. Você deve permitir explicitamente que / seja parte de seu espaço reservado usando uma expressão regular de condição `where`:

```php
Route::get('/search/{search}', function ($search) {
    return $search;
})->where('search', '.*');
```

> As barras codificadas são suportadas apenas no último segmento de rota.

## # **Rotas nomeadas**

As rotas nomeadas permitem a geração conveniente de URLs ou redirecionamentos para rotas específicas. Você pode especificar um nome para uma rota encadeando o método name na definição da rota:

```php
Route::get('/user/profile', function () {
    //
})->name('profile');
```

Você também pode especificar nomes de rotas para ações do controlador:

```php
Route::get('/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

> Os nomes de rota devem sempre ser exclusivos.

#### # **Gerando URLs para Rotas Nomeadas**

Depois de atribuir um nome a uma determinada rota, você pode usar o nome da rota ao gerar URLs ou `redirecionamentos` via `rota` do Laravel e funções auxiliares de redirecionamento:

```php
// Generating URLs...
$url = route('profile');
 
// Generating Redirects...
return redirect()->route('profile');
 
return to_route('profile');
```

Se a rota nomeada define parâmetros, você pode passar os parâmetros como segundo argumento para a função de rota. Os parâmetros fornecidos serão inseridos automaticamente na URL gerada em suas posições corretas:

```php
Route::get('/user/{id}/profile', function ($id) {
    //
})->name('profile');
 
$url = route('profile', ['id' => 1]);
```

Se você passar parâmetros adicionais no array, esses pares chave/valor serão automaticamente adicionados à string de consulta da URL gerada:

```php
Route::get('/user/{id}/profile', function ($id) {
    //
})->name('profile');
 
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
 
// /user/1/profile?photos=yes
```

> Às vezes, você deseja especificar valores padrão em toda a solicitação para parâmetros de URL, como a localidade atual. Para fazer isso, você pode usar o `método URL::defaults`.

#### # **Inspecionando a rota atual**

Se você deseja determinar se a solicitação atual foi roteada para uma determinada rota nomeada, você pode usar o método `nomeado` em uma instância de Rota. Por exemplo, você pode verificar o nome da rota atual de um middleware de rota:
```php
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }
 
    return $next($request);
}
```
## # **Grupos de rotas**

Os grupos de rotas permitem que você compartilhe atributos de rota, como middleware, em um grande número de rotas sem precisar definir esses atributos em cada rota individual.
Grupos aninhados tentam "mesclar" atributos de forma inteligente com seu grupo pai. Middleware e `where` as condições são mescladas enquanto os nomes e prefixos são anexados. Delimitadores de namespace e barras em prefixos de URI são adicionados automaticamente quando apropriado.

### # **Middleware**

Para atribuir `middleware` a todas as rotas dentro de um grupo, você pode usar o método `middleware` antes de definir o grupo. Middleware são executados na ordem em que estão listados no array:

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });
 
    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```
### # **Controladores**
Se um grupo de rotas utilizar o mesmo `controlador`, você poderá usar o método do `controlador` para definir o controlador comum para todas as rotas dentro do grupo. Então, ao definir as rotas, você só precisa fornecer o método do controlador que eles invocam:
```php
use App\Http\Controllers\OrderController;
 
Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```
### # **Roteamento de subdomínio**
Os grupos de rotas também podem ser usados para lidar com o roteamento de subdomínio. Os subdomínios podem receber parâmetros de rota assim como URIs de rota, permitindo que você capture uma parte do subdomínio para uso em sua rota ou controlador. O subdomínio pode ser especificado chamando o método de domínio antes de definir o grupo:
```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```
Para garantir que suas rotas de subdomínio sejam alcançáveis, você deve registrar as rotas de subdomínio antes de registrar as rotas de domínio raiz. Isso impedirá que as rotas de domínio raiz sobrescrevam rotas de subdomínio que tenham o mesmo caminho de URI.
### # **Prefixos de rota**
O método `prefix` pode ser usado para prefixar cada rota no grupo com um determinado URI. Por exemplo, você pode querer prefixar todos os URIs de rota dentro do grupo com `admin`:
```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```
### # **Prefixos de nome de rota**
O método `name` pode ser usado para prefixar cada nome de rota no grupo com uma determinada string. Por exemplo, você pode querer prefixar todos os nomes da rota agrupada com `admin`. A string fornecida é prefixada para o nome da rota exatamente como é especificado, portanto, teremos certeza de fornecer o trailing . caractere no prefixo:
```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```
## # **Vinculação do modelo de rota**
Ao injetar um ID de modelo em uma rota ou ação do controlador, você frequentemente consultará o banco de dados para recuperar o modelo que corresponde a esse ID. A vinculação do modelo de rota do laravel fornece um meio coveniente de injetar automaticamente as instâncias do modelo diretamente em suas rotas. Por exemplo, ao invés de injetar o ID de um usuário, você pode injetar toda a instância do modelo de ``usuário`` que corresponde ao ID fornecido.

### # **Vinculação implícita**
O Laravel resolve automaticamente os modelos do Eloquent definidos em rotas ou ações do controlador cujos nomes de variáveis tipificadas correspondem a um nome de segmento de rota. Por Exemplo:
```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```
Como a variável ``$user`` é tipificada como o modelo ``App\Models\User`` Eloquent e o nome da variável corresponde ao segmento URI ``{user}``, o Laravel injetará automaticamente a instância do modelo que possui um ID correspondente ao valor correspondente do URI de solicitação. Se uma instância de modelo correspondente não for encontrada no banco de dados, uma resposta HTTP 404 será gerada automaticamente.

Obviamente, a vinculação implícita também é possível ao usar metódos do controlador. Novamente, Novamente, observe que o segmento URI {user} corresponde à variável $user no controlador que contém uma dica de tipo App\Models\User:

```php
use App\Http\Controllers\UserController;
use App\Models\User;
 
// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);
 
// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

#### # **Exluir modelos temporariamente**
Tipicamente, a vinculação de modelo implícita não recuperará modelos que foram excluídos de forma reversível. No entanto, você pode instruir a vinculação implícita para recuperar esses modelos encadeando o método ``withTrashed`` na definição da sua rota:`
```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```
#### # **Customizando a chave**
Às vezes você pode querer resolver modelos do Eloquent usando uma coluna diferente de ``id``. Para fazer isso, você pode especificar a coluna na definição do parâmetro de rota:
```php
use App\Models\Post;
 
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```
Se você deseja que a vinculação de modelo sempre use uma coluna de banco de dados diferente de id ao recuperar uma determinada classe de modelo, você pode substituir o método ``getRouteKeyName`` no modelo Eloquent:
```php
/**
 * Get the route key for the model.
 *
 * @return string
 */
public function getRouteKeyName()
{
    return 'slug';
}
```
#### # **Chaves e escopo personalizados**
Ao vincular implicitamente vários modelos Eloquent em uma única definição de rota, você pode desejar definir o escopo do segundo modelo Eloquent de forma que ele seja um filho do modelo Eloquent anterior. Por exemplo, considere esta definição de rota que recupera uma postagem de blog por slug para um usuário específico:
```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```
Ao usar uma associação implícita com chave personalizada como um parâmetro de rota aninhado, O Laravel automaticamente fará o escopo da consulta para recuperar o modelo aninhado por seu pai usando convenções para adivinhar o nome do relacionamento no pai. Nesse caso, será assumido que o modelo ``User`` tem um relacionamento chamado ``posts`` (a forma plural do nome do parâmetro de rota) que pode ser usado para recuperar o modelo ``Post``.

Se desejar, você pode instruir o Laravel a definir o escopo de ligações "filhos" mesmo quando uma chave personalizada não for fornecida. Para fazer isso, você pode invocar o método ``scopeBindings`` ao definir sua rota:
```php
use App\Models\Post;
use App\Models\User;
 
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```
Ou você pode instruir um grupo inteiro de definições de rota para usar associações com escopo:
```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```
#### # **Personalizando o comportamento do modelo ausente**
Normalmente, uma resposta HTTP 404 será gerada se um modelo vinculado implicitamente não for encontrado. No entanto, você pode personalizar esse comportamento chamando o método ``ausente`` ao definir sua rota. O método ``ausente`` aceita um encerramento que será invocado se um modelo vinculado implicitamente não puder ser encontrado:
```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
        ->name('locations.view')
        ->missing(function (Request $request) {
            return Redirect::route('locations.index');
        });
```
### # **Ligação de enumeração implícita**
O PHP 8.1 introduziu suporte para **[Enums](https://www.php.net/manual/en/language.enumerations.backed.php)**. Para complementar esse recurso, o Laravel permite que você digite um **[backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php)** em sua definição de rota e o Laravel só invocará a rota se esse segmento de rota corresponder a um valor Enum válido. Caso contrário, uma resposta HTTP 404 será retornada automaticamente. Por exemplo, dado o seguinte Enum:
```php
<?php
 
namespace App\Enums;
 
enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```
Você pode definir uma rota que só será invocada se o segmento de rota ``{category}`` for ``fruits`` ou ``people``. Caso contrário, o Laravel retornará uma resposta HTTP 404:
```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;
 
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
```
### # **Vinculação explícita**
Você não é obrigado a usar a resolução de modelo implícita e baseada em convenção do Laravel para usar a associação de modelo. Você também pode definir explicitamente como os parâmetros de rota correspondem aos modelos. Para registrar uma ligação explícita, use o método ``model`` do roteador para especificar a classe de um determinado parâmetro. Você deve definir suas associações de modelo explícitas no início do método de ``inicialização`` de sua classe ``RouteServiceProvider``:
```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::model('user', User::class);
 
    // ...
}
```
Em seguida, defina uma rota que contenha um parâmetro ``{user}``:
```php
use App\Models\User;
 
Route::get('/users/{user}', function (User $user) {
    //
});
```
Como vinculamos todos os parâmetros ``{user}`` ao modelo ``App\Models\User``, uma instância dessa classe será injetada na rota. Assim, por exemplo, uma solicitação para ``users/1`` injetará a instância ``User`` do banco de dados que possui um ID de ``1``.

Se uma instância de modelo correspondente não for encontrada no banco de dados, uma resposta HTTP 404 será gerada automaticamente.

#### # **Personalizando a lógica de resolução**

Se você deseja definir sua própria lógica de resolução de model binding, você pode usar o método ``Route::bind``. A closure que você passar para o método ``bind`` receberá o valor do segmento URI e deverá retornar a instância da classe que deve ser injetada na rota. Novamente, essa personalização deve ocorrer no método de ``inicialização`` do ``RouteServiceProvider`` do seu aplicativo:
```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::bind('user', function ($value) {
        return User::where('name', $value)->firstOrFail();
    });
 
    // ...
}
```
Alternativamente, você pode substituir o método ``resolveRouteBinding`` em seu modelo Eloquent. Este método receberá o valor do segmento URI e deverá retornar a instância da classe que deve ser injetada na rota:
```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
```
Se uma rota estiver utilizando o **[escopo de vinculação implícita](https://laravel.com/docs/9.x/routing#implicit-model-binding-scoping)**, o método ``resolveChildRouteBinding`` será usado para resolver a vinculação filha do modelo pai: 
```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```
### # **Rotas substitutas**

Usando o método ``Route::fallback``, você pode definir uma rota que será executada quando nenhuma outra rota corresponder à solicitação recebida.Normalmente, solicitações não tratadas renderizarão automaticamente uma página "404" por meio do manipulador de exceção do seu aplicativo. No entanto, como você normalmente definiria a rota de ``fallback`` em seu arquivo ``routes/web.php``, todos os middlewares no grupo de middlewares da ``web`` serão aplicados à rota. Você pode adicionar middleware adicional a esta rota conforme necessário:
```php
Route::fallback(function () {
    //
});
```
``A rota de fallback deve ser sempre a última rota registrada pelo seu aplicativo.``

### # **Limitação de taxa**

#### # **Definindo limitadores de taxa**
O Laravel inclui serviços de limitação de taxa poderosos e personalizáveis ​​que você pode utilizar para restringir a quantidade de tráfego para uma determinada rota ou grupo de rotas. Para começar, você deve definir configurações de limitador de taxa que atendam às necessidades do seu aplicativo. Normalmente, isso deve ser feito no método ``configureRateLimiting`` da classe ``App\Providers\RouteServiceProvider`` do seu aplicativo.

Os limitadores de taxa são definidos usando o método ``for`` da fachada ``RateLimiter``. O método ``for`` aceita um nome de limitador de taxa e um encerramento que retorna a configuração de limite que deve ser aplicada às rotas atribuídas ao limitador de taxa. A configuração de limite são instâncias da classe ``Illuminate\Cache\RateLimiting\Limit``. Essa classe contém métodos "construtores" úteis para que você possa definir rapidamente seu limite. O nome do limitador de taxa pode ser qualquer string que você desejar: 
```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Configure the rate limiters for the application.
 *
 * @return void
 */
protected function configureRateLimiting()
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```
Se a solicitação recebida exceder o limite de taxa especificado, uma resposta com um código de status HTTP 429 será retornada automaticamente pelo Laravel. Se você deseja definir sua própria resposta que deve ser retornada por um limite de taxa, você pode usar o método de ``resposta``:
```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```
Como os retornos de chamada do limitador de taxa recebem a instância de solicitação HTTP recebida, você pode criar o limite de taxa apropriado dinamicamente com base na solicitação recebida ou no usuário autenticado:
```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});
```

#### # **Limites de taxa de segmentação**
Às vezes você pode querer segmentar os limites de taxa por algum valor arbitrário. Por exemplo, você pode permitir que os usuários acessem uma determinada rota 100 vezes por minuto por endereço IP. Para fazer isso, você pode usar o método ``by`` ao criar seu limite de taxa:
```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100)->by($request->ip());
});
```
Para ilustrar esse recurso usando outro exemplo, podemos limitar o acesso à rota a 100 vezes por minuto por ID de usuário autenticado ou 10 vezes por minuto por endereço IP para convidados:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
});
```
#### # **Limites de taxa múltipla**
Se necessário, você pode retornar uma matriz de limites de taxa para uma determinada configuração de limitador de taxa. Cada limite de taxa será avaliado para a rota com base na ordem em que são colocados na matriz:
```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```
### # **Anexando limitadores de taxa a rotas**
Limitadores de taxa podem ser anexados a rotas ou grupos de rotas usando o **[middleware](https://laravel.com/docs/9.x/middleware)** de ``aceleração``. O middleware de aceleração aceita o nome do limitador de taxa que você deseja atribuir à rota:
```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        //
    });
 
    Route::post('/video', function () {
        //
    });
});
```
#### # **Acelerando com o Redis**
Normalmente, o middleware de ``aceleração`` é mapeado para a classe ``Illuminate\Routing\Middleware\ThrottleRequests``. Esse mapeamento é definido no kernel HTTP do seu aplicativo ``(App\Http\Kernel)``. No entanto, se você estiver usando o Redis como o driver de cache do seu aplicativo, convém alterar esse mapeamento para usar a classe ``Illuminate\Routing\Middleware\ThrottleRequestsWithRedis``. Esta classe é mais eficiente no gerenciamento da limitação de taxa usando o Redis: 
```php
'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
```
### # **Falsificação de método de formulário**
Os formulários HTML não suportam as ações ``PUT``, ``PATCH`` ou ``DELETE``. Portanto, ao definir rotas ``PUT``, ``PATCH`` ou ``DELETE`` que são chamadas de um formulário HTML, você precisará adicionar um campo ``_method`` oculto ao formulário. O valor enviado com o campo ``_method`` será utilizado como método de requisição HTTP:
```php
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```
Por conveniência, você pode usar ``@method`` **[Blade directive](https://laravel.com/docs/9.x/blade)** para gerar o campo de entrada ``_method``:

```php
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```
### # **Acessando a Rota Atual**
Você pode usar os métodos ``current``, ``currentRouteName`` e ``currentRouteAction`` na fachada ``Route`` para acessar informações sobre a rota que trata a solicitação recebida:
```php
use Illuminate\Support\Facades\Route;
 
$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```
Você pode consultar a documentação da API para a **[classe subjacente da fachada Route](https://laravel.com/api/9.x/Illuminate/Routing/Router.html)** e da **[instância Route](https://laravel.com/api/9.x/Illuminate/Routing/Route.html)** para revisar todos os métodos disponíveis no roteador e nas classes de rota.

### # **Compartilhamento de recursos entre origens (CORS)**
O Laravel pode responder automaticamente às solicitações HTTP CORS ``OPTIONS`` com valores que você configura. Todas as configurações do CORS podem ser configuradas no arquivo de configuração ``config/cors.php`` da sua aplicação. As solicitações ``OPTIONS`` serão tratadas automaticamente pelo **[middleware](https://laravel.com/docs/9.x/middleware)** ``HandleCors`` que é incluído por padrão em sua pilha de middleware global. Sua pilha de middleware global está localizada no kernel HTTP do seu aplicativo ``(App\Http\Kernel)``.

Para obter mais informações sobre cabeçalhos CORS e CORS, consulte a **[documentação da Web do MDN sobre CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers)**.

### # **Cache de rota**
Ao implantar seu aplicativo em produção, você deve aproveitar o cache de rota do Laravel. O uso do cache de rotas diminuirá drasticamente o tempo necessário para registrar todas as rotas do seu aplicativo. Para gerar um cache de rota, execute o comando Artisan ``route:cache``:
```php
php artisan route:cache
```

Depois de executar este comando, seu arquivo de rotas em cache será carregado em cada solicitação. Lembre-se, se você adicionar novas rotas, precisará gerar um novo cache de rota. Por isso, você só deve executar o comando ``route:cache`` durante a implantação do seu projeto. 

Você pode usar o comando ``route:clear`` para limpar o cache da rota:

```php
php artisan route:clear
```