# #CRIANDO RESPOSTAS
- #Arrays e Strings
### Todas as rotas e controladores deveriam retornar uma resposta para ser enviado de volta para o navegador do usuário. Laravel fornece várias maneiras diferentes de retornar respostas. A resposta mais básica é retornando uma String de uma rota ou controlador. O framework irá automaticamente converter uma String para uma resposta HTTP completa.
```php
Route::get('/', function () {
    return 'Hello World';
});

```


### Além de retornar strings de suas rotas e controladores, você também pode retornar arrays. O framework irá automaticamente converter o array em uma resposta JSON.
```php
Route::get('/', function () {
    return [1, 2, 3];
});

```

- #Objetos de respostas
### Geralmente, você não estará retornando apenas strings ou arrays simples de suas ações de rotas. Em vez disso, você estará retornando Illuminate\Http\Response instâncias completas ou views. Retornando uma instância de resposta completa permite que você personalize um código de status e cabeçalhos da resposta. Uma instância da resposta herda da classe Symfony\Component\HttpFoundation\Response, que fornece uma variedade de métodos para construção de respostas HTTP.
```php
Route::get('/home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```
- #Modelos e coleções eloquente
### Você também pode retornar modelos de ORM eloquente  e coleções diretamente de suas rotas e controladores. Quando você retornar, Laravel irá automaticamente converter os modelos e coleções para respostas JSON respeitando os atributos escondidos do modelo.
```php
use App\Models\User;
 
Route::get('/user/{user}', function (User $user) {
    return $user;
});

```
## #Anexando cabeçalhos em repostas
### Mantenha em mente que a maioria dos métodos de respostas estão encadeados, permitindo a construção fluente de instâncias de respostas. Por exemplo, você pode usar o método de cabeçalho para adicionar uma série de cabeçalhos para a resposta antes o mandando de volta para o usuário:
```php
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');

```
### Ou você pode usar o método “withHeaders” para especificar uma série de cabeçalhos para serem adicionados na resposta.
```php
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);

```
- #Middleware de controle de cache
### Laravel possui um middleware “cache.headers”, que pode ser usado para rapidamente definir um cabeçalho de controle de cache para um grupo de rotas. Diretivas devem ser fornecidas usando o equivalente “snake case” da diretiva de controle de cache correspondente e deve ser separado por um ponto e vírgula. Se etag está especificado na lista de diretivas, um hash MD5 do conteúdo irá automaticamente ser definido como o identificador ETag:
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
## #Anexando Cookies ás respostas
### Você pode anexar um cookie a uma instância de saída Illuminate\Http\Response usando o método do cookie. Você deve dar o nome, valor, e o número de minutos o cookie deve ser considerado válido para esse método.:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);

```

###  O método cookie também aceita um pouco mais de argumentos que são usados com menos frequência. Geralmente, esses argumentos tem o mesmo propósito e significado como o argumento que seria dado para o método nativo setcookie do PHP:
```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```
### Se você gostaria de garantir que um cookie está enviado com a resposta de saida mas você ainda não tem uma instância no qual responda, você pode usar o cookie facade para "enfilerar" os cookies para o anexamento para a resposta quando ele é mandado. O método que recebe o argumento preciso para criar uma instância do cookie. Esses cookies serão anexados para a resposta de saida antes dele ser mandado para o navegador
```php
use Illuminate\Support\Facades\Cookie;
 
Cookie::queue('name', 'value', $minutes);
```
- #Gerando instâncias do cookie
### Se você gosta de gerar uma instância  Symfony\Component\HttpFoundation\Cookie que pode ser anexada em uma instância de resposta mais tarde, você deve usar o ajudante global do cookie. Esse cookie não será mandado de volta para o cliente a nao ser que ele esteja anexado em uma instância de resposta:
```php
$cookie = cookie('name', 'value', $minutes);
 
return response('Hello World')->cookie($cookie);
```
- #Expirando cookies antecipadamente
### Você pode remover um cookie o expirando via o método withoutCookie de uma reposta de saída.
```php 
return response('Hello World')->withoutCookie('name');
```
###  Se você ainda não tem uma instância de saída, você pode usar  método cookie expire do facade para expirar o cookie:
```php
Cookie::expire('name');
```
## #Cookies e Criptografia
### Por defeito, todos os cookies gerados por Laravel são codificados e assinados para que não possam ser modificados ou lidos pelo cliente.Se desejar desativar a encriptação para um subconjunto de cookies gerados pela sua aplicação, pode utilizar a propriedade $except do App\Http\Middleware\EncryptCookies middleware, que se encontra no app/Http/Middleware diretório:
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
#  #Redirecionamentos
### redirecionamentos responsivos são instâncias da classe illuminate\http\Redirect Response, e contém cabeçalhos adequados necessários para redirecionar o usuário para outra URL. Existem várias maneiras de gerar a instância RedirectResponse. O método mais simples é usar um ajudante global redirect:
```php
Route::get('/dashboard', function () {
    return redirect('home/dashboard');
});
```
### Às vezes você pode querer redirecionar o usuário para uma localização anterior, como quando um formulário inválido é enviado. Você pode fazer isso usando uma função auxiliar global back. Uma vez que esse recurso utiliza a sessão, certifique-se de que a rota chamando a função back está usando o grupo middleware web:
```php
Route::post('/user/profile', function () {
    // Validate the request...
 
    return back()->withInput();
});
```
## #Redirecionando para rotas nomeadas
### Quando você chama o ajudante redirect sem parâmetro, uma instância Illuminate\Routing\ Rediretor é retornada, permitindo que você chame qualquer método na instância Redirector. Por exemplo, para gerar para uma rota nomeada directResponse, você pode usar o método de rota, você pode usar o método de route.
```php
return redirect()->route('login');
```
### Se a sua rota tem parâmetros, você pode passar eles como um argumento secundário para o método route
```php
// para uma rota com o seguinte URI: /profile/{id}
 
return redirect()->route('profile', ['id' => 1]);

```
- #Preenchendo parâmetros através de modelos Eloquentes
### Se você estiver se direcionando para uma rota com um parâmetro "ID" que está sendo preenchido a partir de um modelo Eloquente, você pode passar o próprio modelo. O ID será extraído automaticamente:
```php
// Para uma rota com o seguinte URI: /profile/{id}
 
return redirect()->route('profile', [$user]);
```
### Se você quiser personalizar o valor que está colocado no parâmetro de rota, você pode especificar a coluna na definição do parâmetro de rota (/profile/{id:slug}) ou você pode substituir o método getRouteKey no modelo Eloquente:
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
## #Redirecionamento para ações do controlador
### Você também pode gerar redirecionamentos para ações do controlador. Para isso, passe o controlador e o nome de ação para o método action:
```php
use App\Http\Controllers\UserController;
 
return redirect()->action([UserController::class, 'index']);
```
### Se a rota do controlador exigir parâmetros, você pode passá-los como segundo argumento para o método action:
```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```
## #Redirecionando para domínios externos
### Às vezes, você pode precisar redirecionar para um domínio fora da sua aplicação. Você pode fazer isso chamando o método away , que cria um RedirectResponse sem qualquer codificação validação ou verificação de URL adicional:
```php
return redirect()->away('https://www.google.com');
```
## #Redirecionando com dados de sessão flashed
### O redirecionamento para uma nova URL e o flash de dados para a sessão geralmente são feitos ao mesmo tempo Normalmente, isso é feito após a execução bem-sucedida de uma ação quando você exibe uma mensagem de sucesso na sessão. Por conveniência, você pode criar uma instância do RedirectResponse e enviar dados para a sessão em uma única cadeia de métodos fluente:
```php
Route::post('/user/profile', function () {
    // ...
 
    return redirect('dashboard')->with('status', 'Profile updated!');
});
```
### Depois que o usuário for redirecionado, você poderá exibir a mensagem piscada da sessão . Por exemplo, usando a sintaxe do Blade:
```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
- #Redirecionamento com input
### Pode utilizar o método withInput fornecido pela instância RedirectResponse para piscar os dados de entrada do pedido do input para a sessão antes de redirecionar o utilizador para um novo local. Isto é normalmente feito se o utilizador tiver encontrado um erro de validação. Uma vez que a entrada tenha sido assinalada na sessão, poderá facilmente recuperá-la durante o próximo pedido de repovoamento do formulário:
```php
return back()->withInput();
```
# #Outros tipos de resposta
### O ajudante de resposta pode ser utilizado para gerar outros tipos de instâncias. Quando o ajudante de resposta é chamado sem argumentos, uma implementação do Illuminate\Contracts\Routing\ResponseFactory O contrato é retornado. Este contrato fornece vários métodos úteis para gerar respostas. 
## #Ver respostas
### Se você precisar controlar o status e os cabeçalhos mas também precisa retornar uma **exibição** como conteúdo da resposta, você deve usar o método **view**
```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```
### Claro, se você não precisar passar um código de status HTTP personalizado ou cabeçalhos personalizados, você pode a função de ajudante global **view**
## #Respostas JSON
### O método json definirá automaticamente o cabeçalho tipo-conteúdo para a aplicação/json bem como converterá o array dado para o JSON usando a função PHP json_encode:
```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```
### Se você quiser criar uma resposta JSONP você pode usar o método json em combinação com o método withCallback:
```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```
## #descarregamento de ficheiros 
### O método de descarga pode ser utilizado para gerar uma resposta que força o navegador do utilizador a descarregar o ficheiro no caminho dado. O método de descarregamento aceita um nome de ficheiro como segundo argumento para o método, que determinará o nome de ficheiro que é visto pelo utilizador que descarrega o ficheiro. Finalmente, pode passar um conjunto de cabeçalhos HTTP como terceiro argumento para o método:
```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);

```
- #Descarregamentos em fluxo contínuo
### Por vezes pode desejar transformar a resposta em cadeia de uma dada operação numa resposta descarregável sem ter de escrever o conteúdo da operação em disco. Pode utilizar o método streamDownload neste cenário. Este método aceita como argumentos uma chamada de retorno, nome de ficheiro, e um conjunto opcional de cabeçalhos:

```php
use App\Services\GitHub;
 
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');

```
## #Respostas de ficheiros
### O método de ficheiro pode ser utilizado para exibir um arquivo, como uma imagem ou PDF, diretamente no navegador do utilizador em vez de iniciar um descarregamento. Este método aceita o caminho para o arquivo como o seu primeiro argumento e um conjunto de cabeçalhos como o seu segundo argumento:

```php
return response()->file($pathToFile);
 
return response()->file($pathToFile, $headers);

```
# #Macros de resposta
### Se desejar definir uma resposta personalizada que possa ser reutilizada nas suas rotas e controladores, poderá utilizar o método macro na fachada de respostas. Normalmente, deve chamar a este método a partir do método de **boot** de uma das suas aplicações **service providers**, tais como o **App\Providers\AppServiceProvider** prestação de serviços:
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