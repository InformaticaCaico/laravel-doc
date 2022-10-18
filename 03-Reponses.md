# Reponses

## # CRIANDO RESPOSTAS

#### # Arrays e Strings

Todas as rotas e controladores devem retornar uma Reponse de volta para o browser do usuário. Laravel fornece várias maneiras diferentes de retornar Responses. A resposta mais básica é retornando uma String de uma rota ou controlador. O framework irá automaticamente converter uma String para uma resposta HTTP completa.

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Além de retornar strings a partir de suas rotas e controladores, você também pode retornar *arrays*. O framework irá automaticamente converter o *array* em uma resposta JSON.

```php
Route::get('/', function () {
    return [1, 2, 3];
});

```

> Você sabia que também pode retornar um `Collection do eloquent` a partir das rotas e controladores? Elas serão automaticamente convertidas para JSON. Faça um teste.

#### # Objetos de respostas

Geralmente, você não estará retornando apenas strings ou arrays simples de suas ações de rotas. Em vez disso, você retornará instâncias completas de `Illuminate\Http\Response` completas ou `views`. 

Retornar uma instância `Response` completa permite que você personalize um código de status e cabeçalhos da resposta. Uma instância `Response` herda da classe `Symfony\Component\HttpFoundation\Response`, que fornece uma variedade de métodos para construção de respostas HTTP:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

#### # Modelos e coleções eloquente

Você também pode retornar modelos de ORM eloquente e coleções diretamente de suas rotas e controladores. Quando você retornar, Laravel irá automaticamente converter os modelos e coleções para respostas JSON respeitando os atributos escondidos do modelo (senhas, por exemplo):

```php
use App\Models\User;
 
Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### # Anexando cabeçalhos em repostas

Mantenha em mente que a maioria dos métodos de `Response` pode ser encadeados, permitindo a construção fluente de instâncias Response. Por exemplo, você pode usar o método `header` para adicionar uma série de cabeçalhos para a resposta antes de enviá-la de volta para o usuário:

```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```

Ou você pode usar o método `withHeaders` para especificar uma série de cabeçalhos para serem adicionados na Response.

```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);

```
#### # Middleware de controle de cache

Laravel possui um middleware `cache.headers`, que pode ser usado para rapidamente definir um cabeçalho `Cache-control` para um grupo de rotas. Diretivas devem ser fornecidas usando o equivalente “snake case” da diretiva de controle de cache correspondente e deve ser separado por um ponto e vírgula. Se `etag` está especificado na lista de diretivas, um hash MD5 do conteúdo irá automaticamente ser definido como o identificador ETag:

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });
 
    Route::get('/terms', function () {
        // ...
    });
});
```

### # Anexando Cookies às respostas

Você pode anexar um cookie a uma instância de saída `Illuminate\Http\Response` usando o método `cookie`. Você deve dar o nome, valor, e o número de minutos que o cookie deve ser considerado válido para esse método:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

O método `cookie` também argumentos adicionais que não são tão utilizados. Geralmente, esses argumentos tem o mesmo propósito e significado que os argumentos utilizados no método nativo `setcookie` do PHP:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Se você gostaria de garantir que um cookie é enviado junto ao Response e você não tem uma instância Response, você pode lançar mão do facade `Cookie` para "queue" (enfileirar) os cookies como anexos de um Response quando esta for enviada. O método `queue` aceita arguemntos necessários para criar instâncias do tipo cookie. Estes cookies serão anexados a Response que será enviada antes mesmo dela ser enviada para o browser.

```php
use Illuminate\Support\Facades\Cookie;
 
Cookie::queue('name', 'value', $minutes);
```

#### # Gerando instâncias do cookie

Se você quiser gerar uma insância de `Symfony\Component\HttpFoundation\Cookie` para ser anexada a uma Response futura, você deve usar o *helper* global `cookie`. O cookie criando não será enviado para o usuário até que ele seja explicitamente anexado a um Response:

```php
$cookie = cookie('name', 'value', $minutes);
 
return response('Hello World')->cookie($cookie);
```

#### # Expirando cookies antecipadamente

Você pode remover um cookie fazendo-o expirar via o método `withoutCookie` de um Response.

```php 
return response('Hello World')->withoutCookie('name');
```

Se você ainda não tem uma instância de um Response, você pode usar o Facade `Cookie` e aplicar o método `expire` para expirar o cookie:
 
```php
Cookie::expire('name');
```

### # Cookies e Criptografia

Por defeito, todos os cookies gerados por Laravel são criptografados e assinados para que não possam ser modificados ou lidos pelo cliente. Se desejar desativar a encriptação para um subconjunto de cookies gerados pela sua aplicação, pode utilizar a propriedade `$except` do middleware `App\Http\Middleware\EncryptCookies`, que se encontra no `app/Http/Middleware` diretório:

```php
/**
 * Os nomes dos cookies que não devem ser codificados.
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

## # Redirecionamentos

Responses de redirecionamentos são instâncias de  `illuminate\http\RedirectResponse`, contém cabeçalhos adequados necessários para redirecionar o usuário para outra URL. Existem várias maneiras de gerar a instância `RedirectResponse`. O método mais simples é usar o *helper* global `redirect`:

```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```

Às vezes você deseja redirecionar o usuário para uma localização anterior, por exemplo quando um formulário inválido é enviado. Você pode fazer isso usando uma função auxiliar global `back`. Uma vez que esse recurso utiliza a sessão, certifique-se de que a rota chamando a função back está usando o grupo middleware `web`:

```php
Route::post('/user/profile', function () {
    // Validate the request...
 
    return back()->withInput();
});
```

### # Redirecionando para rotas nomeadas

Quando você chama o *helper* `redirect` sem parâmetro, uma instância `Illuminate\Routing\Rediretor` é retornada, permitindo que você chame qualquer método na instância `Redirector`. Por exemplo, para gerar um `RedirectResponse` para uma rota nomeada, você deve usar o método `route`: 


```php
return redirect()->route('login');
```

Se a sua rota tem parâmetros, você pode passar eles como um argumento secundário para o método `route`:

```php
// para uma rota com o seguinte URI: /profile/{id}
 
return redirect()->route('profile', ['id' => 1]);

```
#### # Preenchendo parâmetros através de modelos Eloquentes

Se você estiver redirecionando para uma rota com um parâmetro "ID" que está sendo preenchido a partir de um modelo Eloquent, você pode passar o próprio modelo. O ID será extraído automaticamente:

```php
// Para uma rota com o seguinte URI: /profile/{id}
 
return redirect()->route('profile', [$user]);
```

Se você quiser personalizar o valor que está colocado no parâmetro de rota, você pode especificar a coluna na definição do parâmetro de rota (`/profile/{id:slug}`) ou você pode substituir o método `getRouteKey` no modelo Eloquente:

```php
/**
 * Get the value of the model's route key.
 *
 * @return mixed
 */
public function getRouteKey()
{
    return $this->slug;
}
```

### # Redirecionamento para ações do controlador

Você também pode gerar redirecionamentos para ações do controlador. Para isso, passe o controlador e o nome de ação para o método `action`:

```php
use App\Http\Controllers\UserController;
 
return redirect()->action([UserController::class, 'index']);
```

Se a rota do controlador exigir parâmetros, você pode passá-los como segundo argumento para o método `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

### # Redirecionando para domínios externos

Às vezes, você pode precisar redirecionar para um domínio fora da sua aplicação. Você pode fazer isso chamando o método `away`, que cria um `RedirectResponse` sem qualquer codificação, validação ou verificação de URL adicional:

```php
return redirect()->away('https://www.google.com');
```

### # Redirecionando com dados de sessão flashed

O redirecionamento para uma nova URL e o flashing de dados para a sessão são feitos, geralmente, ao mesmo tempo. Normalmente, isto é quando uma ação é realizada e uma mensagem é adicionada a sessão (como um flash). Por conveniência, você pode criar uma instância `RedirectResponse` e adicionar(flash) os dados para uma sessão em um único método fluente e encadeado:

```php
Route::post('/user/profile', function () {
    // ...
 
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```

Depois que o usuário ser redirecionado, você mostrar a messagem da sessão. Por exemplo, usando a sintaxe do Blade:

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

#### # Redirecionamento com input

Você pode utilizar o metódo `withInput` da instância `RedirectResponse` para atuatilizar a sessão com os dados de input da Request (enviados num formulário por exemplo) antes de enviar o usuário para um novo local (url). Isso é feito sempre que o usuário encontra um problema de validação dos dados. Dado que os dados de entrada são atualizados na sessão, você pode recuperá-los durante a próxima Request e repopular os formulário (evita que o usuário digite todos os dados):

```php
return back()->withInput();
```
## # Outros tipos de resposta

O *helper* `response` pode ser utilizado para gerar outros tipos de instâncias Response. Quando o *helper* `response` é chamado sem argumentos, uma implementação do *`contract`* `Illuminate\Contracts\Routing\ResponseFactory` sé retornado. Este contrato fornece vários métodos úteis para gerar Responses. 

### # Responses como View

Se você precisar controlar o status e os cabeçalhos mas também precisa retornar uma `view` como conteúdo da resposta, você deve usar o método `view`:

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```
Claro, se você não precisar passar um código de status HTTP personalizado ou cabeçalhos personalizados, você pode usar a função `view`.

### #Respostas JSON

O método `json` definirá automaticamente o cabeçalho `Content-type` para a `application/json` bem como converterá o array de dados para JSON usando a função PHP `json_encode`:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Se você quiser criar uma resposta JSON,  utilize a combinação do método `json` com o método `withCallback`:

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

### # Download de Arquivos

O método `download` pode ser utilizado para gerar uma resposta que força o navegador do usuário a baixar o arquivo.  O método `download` aceita um nome de arquivo como segundo argumento, que determinará o nome de arquivo que é visto pelo utilizador que baixa o arquivo. Por fim, você pode passar um conjunto de cabeçalhos HTTP como terceiro argumento:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);

```

> O `Symfoy HttpFoundation` que gerencia downloads, requer que o arquivo baiado tenham um nome ASCII.

#### # Download em fluxo contínuo

Às vezes, você deseja transformar uma Response de uma dada operação em uma Response que pode ser baixada sem que os dados tenham sido escritos em disco. Você pode usar o método `streamDownload` neste cenário. Este método aceita um *callback*, um nome de arquivo e um array opcional com cabeçalhos.


```php
use App\Services\GitHub;
 
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');

```
### # Respostas como Arquivos

O método `file` pode ser utilizado para exibir um arquivo, como uma imagem ou PDF, diretamente no navegador do usuário em vez de iniciar um download. Este método aceita o caminho para o arquivo como o seu primeiro argumento e um conjunto de cabeçalhos como o seu segundo argumento:

```php
return response()->file($pathToFile);
 
return response()->file($pathToFile, $headers);

```
## # Macros de resposta

Se desejar definir uma Response personalizada que possa ser reutilizada nas suas rotas e controladores, você poderá utilizar o método `macro` do Facade `Response`. Normalmente, você aplica essa opção dentro do método `boot` a partir de um dos `Service Providers` de sua aplicação, por exemplo `App\Providers\AppServiceProvider`:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Response::macro('caps', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

A função `macro` aceita um nome como seu primeiro argumento e uma closure como seu segundo argumetno. O clojuse do macro será executado quando o nome da macro for executada a partir de uma implementação `ResponseFactory` ou através do helper `response`:

```php
return response()->caps('foo');
```