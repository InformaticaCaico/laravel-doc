# # Controladores

## # Introdução

Em vez de definir toda a sua lógica de manipulação de solicitações como encerramentos em seus arquivos de rota, você pode organizar esse comportamento usando classes "controladoras". Os controladores podem agrupar a lógica de manipulação de solicitações relacionadas em uma única classe. Por exemplo, uma classe UserController pode lidar com todas as solicitações recebidas relacionadas a usuários, incluindo mostrar, criar, atualizar e excluir usuários. Por padrão, os controladores são armazenados no diretório app/Http/Controllers.


## # Escrevendo Controladores

### # Controladores básicos:
 
Vamos dar uma olhada em um exemplo de um controlador básico. Observe que o controlador estende a classe de controlador base incluída no Laravel: App\Http\Controllers\Controller:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Models\User;
 
class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     *
     * @param  int  $id
     * @return \Illuminate\View\View
     */
    public function show($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Você pode definir uma rota para este método do controlador assim:

```php
use App\Http\Controllers\UserController;
 
Route::get('/user/{id}', [UserController::class, 'show']);
```

Quando uma solicitação de entrada corresponde ao URI de rota especificado, o método show na classe App\Http\Controllers\UserController será invocado e os parâmetros de rota serão passados ​​para o método.

> Os controladores não são obrigados a estender uma classe base. No entanto, você não terá acesso a recursos convenientes, como middleware e métodos de autorização.

### # Controladores de acao única


## # Middleware do controlador

O middleware pode ser atribuído às rotas do controlador em seus arquivos de rota:

```php
    Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

Ou você pode achar conveniente especificar middleware dentro do construtor do seu controlador. Usando o método de middleware no construtor do seu controlador, você pode atribuir middleware às ações do controlador:

```php
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
}
```

Os controladores também permitem que você registre o middleware usando um encerramento. Isso fornece uma maneira conveniente de definir um middleware embutido para um único controlador sem definir uma classe de middleware inteira:

```php
$this->middleware(function ($request, $next) {
    return $next($request);
});
```

## # Controladores de recursos

Se você pensar em cada modelo criado com o Eloquent do Laravel em sua aplicação como um “recurso”, irá notar que é comum executar o mesmo conjunto de ações em cada um deles. Por exemplo, imagine que sua aplicação contém um modelo `Foto` e um modelo `Filme`. É provável que os usuários possam criar, ler, atualizar ou excluir esses recursos.

Devido a essa similaridade, o roteamento de recursos do Laravel atribui rotas típicas de criação, leitura, atualização e exclusão (“CRUD” - create, read, update e delete- em inglês) a um controlador com uma única linha de código. Inicialmente, para criá-lo, podemos usar a opção `make:controller` do comando Artisan `--resource` para criar rapidamente um controlador com essas ações.

```php
php artisan make:controller PhotoController --resource
```

Este comando irá gerar um controlador em `app/Http/Controllers/PhotoController.php`. O controlador conterá um método para cada uma das operações de recursos disponíveis. Em seguida, você pode registrar uma rota de recurso (no arquivo web.php) que aponta para o controlador.

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class);
```

Essa declaração de rota única cria outras várias rotas para lidar com uma variedade de ações no recurso. O controlador gerado já terá métodos stubs (ou esboços de métodos) para cada uma dessas ações. Lembre-se, você sempre pode obter uma visão geral rápida das rotas da sua aplicação executando o comando do Artisan `route:list`.

Você pode até registrar muitos controladores de recurso de uma só vez passando um array para o método `resources`.

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

#### # Ações tratadas pelo controlador de recursos

| Verb	| URI | Action | Route Name |
| ----- | --- | ------ | ---------- |
|GET |	/photos |	index |	photos.index
|GET |	/photos/create |	create |	photos.create
|POST |	/photos |	store |	photos.store
|GET |	/photos/{photo} |	show |	photos.show
|GET |	/photos/{photo}/edit |	edit |	photos.edit
|PUT/PATCH | /photos/{photo} |	update |	photos.update
|DELETE	|/photos/{photo} |	destroy |	photos.destroy

#### # Personalizando o comportamento do modelo ausente

Normalmente, uma resposta HTTP 404 será gerada se um modelo de recurso ligado implicitamente não for encontrado. No entanto, você pode personalizar esse comportamento chamando o método `missing` ao definir sua rota de recurso. O método `missing` aceita um encerramento que será invocado se um modelo vinculado implicitamente não puder ser encontrado para nenhuma das rotas do recurso.

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
Route::resource('photos', PhotoController::class)
        ->missing(function (Request $request) {
            return Redirect::route('photos.index');
        });
```

#### # Especificando o modelo de recursos

Se você estiver usando a [vinculação do modelo de rota](https://laravel.com/docs/9.x/routing#route-model-binding) e quiser que os métodos do controlador de recursos deem dicas de tipo para uma instância do modelo, você pode usar a opção `--model` ao gerar o controlador:

```php
php artisan make:controller PhotoController --model=Photo --resource
```

#### # Modelos soft deleted

Tipicamente, vincular modelo implicitamente não vai recuperar modelos do banco de dados que foram removidos no modo `soft deleted`, e isso, ao contrário, irá gerar uma respostas HTTP 404. Entretanto, você pode instruir o framework para permitir modelos removidos no modo soft delete invocando o método `withTrashed` quando definir a rota do recurso:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->withTrashed();
```

Charmar `withTrashed` sem arguementos permitirá permitirá modelos removido no modo soft delete serem usados nas rotas `show`, `edit` e `update`. Você pode especificar um subconjunto dessas rotas passando um array para o método `withTrashed`.


#### # Especificar o modelo resource

Se você estiver ustilizando [`route model binding`](https://laravel.com/docs/9.x/routing#route-model-binding) e gostaria de indicar o tipo de modelo nas funcões do controlador, você pode usar a opcão `--model` quando gerar um controlador.

```php
php artisan make:controller PhotoController --model=Photo --resource
```

#### # Gerando solicitações de formulário

Você pode fornecer a opção  `--requests` ao gerar um controlador de recursos para instruir o Artisan a gerar [classes de solicitação de formulário](https://laravel.com/docs/9.x/validation#form-request-validation) para os métodos de armazenamento e atualização do controlador.

```php
php artisan make:controller PhotoController --model=Photo --resource --requests
```

### # Rotas de recursos parciais

Ao declarar uma rota de recurso você pode especificar um subconjunto de ações que o controlador deve manipular em vez do conjunto completo de métodos/ações padrões.

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);
 
Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### # Rotas de recursos da API

Ao declarar rotas de recurso que serão consumidas por APIs você normalmente desejará excluir rotas que apresentem modelos HTML como `create` e `edit`. Por conveniência você pode usar o método `apiResource` para excluir automaticamente essas duas rotas:

```php
use App\Http\Controllers\PhotoController;
 
Route::apiResource('photos', PhotoController::class);
```

Você pode registrar vários controladores de recursos da API de uma só vez passando um array para o método `apiResources`: 

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;
 
Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Para gerar rapidamente um controlador de recursos de API que não inclua os métodos `create` ou `edit`, use a opção `--api` quando executar o comando `make:controller`:

```php
php artisan make:controller PhotoController --api
```

### # Recursos aninhados

Às vezes você pode precisar definir rotas para um recurso aninhado. Por exemplo, um recurso de foto pode ter vários comentários que podem ser anexados nela. Para aninhar os controlodores de recursos (comentários e fotos), você pode usar a declaração "ponto' em sua declaração de rotas:

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class);
```

Essa rota registrará um recurso aninhado que pode ser acessado com URIs como a seguinte:

```php
/photos/{photo}/comments/{comment}
```

#### # Escopo de recursos aninhados

[O recurso de associação de modelo implícito](https://laravel.com/docs/9.x/routing#implicit-model-binding-scoping) do Laravel pode definir automaticamente o escopo de associações aninhadas de modo que o modelo filho seja confirmado como pertencente ao modelo pai. Usando o método `scoped` ao definir seu recurso aninhado, você pode habilitar o escopo automaticamente, bem como instruir o Laravel para qual campo o recurso filho deve ser relacionado. Para obter mais informações sobre como fazer isso, consulte a documentação sobre como definir o [escopo de rotas de recursos](https://laravel.com/docs/9.x/controllers#restful-partial-resource-routes).

#### # Aninhamento raso

Muitas vezes não é totalmente necessário ter os IDs do pai e filho em uma URI, pois o ID do filho já é um identificador exclusivo. Ao usar identificadores exclusivos, como chaves primárias de incremento automático para identificar seus modelos em segmentos de URI, você pode optar por usar o "aninhamento raso".

```php
use App\Http\Controllers\CommentController;
 
Route::resource('photos.comments', CommentController::class)->shallow();
```

Essa definição de rota definirá as outras seguintes rotas:

| Verb | URI | Action | Route Name |
| ---- | --- | ------ | ---------- |
|GET |	/photos/{photo}/comments |	index |	photos.comments.index
|GET |	/photos/{photo}/comments/create |	create |	photos.comments.create
|POST |	/photos/{photo}/comments |	store |	photos.comments.store
|GET |	/comments/{comment} |	show |	comments.show
|GET |	/comments/{comment}/edit |	edit | comments.edit
|PUT/PATCH |	/comments/{comment} |	update |	comments.update
|DELETE |	/comments/{comment} |	destroy |	comments.destroy

### # Nomeando rotas de recursos

Por padrão, todas as ações do controlador de recursos têm um nome de rota; no entanto, você pode substituir esses nomes passando uma matriz de nomes com os nomes de rota desejados:

```php
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

### # Como nomear parâmetros de rotas de recursos

Por padrão, Route::resource criará os parâmetros de rota para suas rotas de recursos com base na versão "singularizada" do nome do recurso. Você pode facilmente substituir isso por recurso usando o método de parâmetros. A matriz passada para o método de parâmetros deve ser uma matriz associativa de nomes de recursos e nomes de parâmetros:

```php
use App\Http\Controllers\AdminUserController;
 
Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

O exemplo acima gera o seguinte URI para a rota de exibição do recurso:

```php
/users/{admin_user}
```

### # Rotas de recursos de escopo


O recurso de vinculação de modelo implícito com escopo definido do Laravel pode definir automaticamente o escopo de vinculações aninhadas de modo que o modelo filho resolvido seja confirmado como pertencente ao modelo pai. Usando o método com escopo definido ao definir seu recurso aninhado, você pode habilitar o escopo automático, bem como instruir o Laravel em qual campo o recurso filho deve ser recuperado por:

```php
use App\Http\Controllers\PhotoCommentController;
 
Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Essa rota registrará um recurso aninhado com escopo que pode ser acessado com URIs como o seguinte:

```php
/photos/{photo}/comments/{comment:slug}
```

Ao usar uma ligação implícita de chave personalizada como um parâmetro de rota aninhado, o Laravel automaticamente definirá o escopo da consulta para recuperar o modelo aninhado por seu pai usando convenções para adivinhar o nome do relacionamento do pai. Neste caso, será assumido que o modelo Photo possui um relacionamento denominado comentários (o plural do nome do parâmetro de rota) que pode ser usado para recuperar o modelo Comment.

### # Localização de URLs de recursos
Por predefinição, Route::resource criará URLs de recursos usando verbos em inglês e regras no plural. Se você precisar localizar os verbos de ação create e edit, você pode usar o método  Route::resourceVerbs. Isto pode ser feito no começo do método boot dentro de suas aplicações App\Providers\RouteServiceProvider:

```php
* Define your route model bindings, pattern filters, etc.
*
* @return void
*/
public function boot()
{
   Route::resourceVerbs([
       'create' => 'crear',
       'edit' => 'editar',
   ]);
 
   // ...
}
```

O pluralizador do Laravel suporta várias diferentes linguagens que você pode configurar baseado em suas necessidades. Uma vez que os verbos e a linguagem de pluralização foram personalizados, um registro de rota de recursos tal como Route::resource('publicacion', PublicacionController::class) produzirá as URLs a seguir:

```php
/publicacion/crear
 
/publicacion/{publicaciones}/editar
```



### # Complementando Controladores de Recursos

Se você precisar adicionar rotas adicionais ao controlador de recursos além do recurso de rota definido por padrão, você  deve definir essa rota depois você liga ao método Route::resource; outra, a rota definida pelo método resource  pode involuntariamente ter precedência sobre suas rotas suplementares:

```php
use App\Http\Controller\PhotoController;
 
Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```


> Lembre-se de  manter seus controladores focados. Se você encontrar-se rotineiramente precisando de métodos fora do conjunto típico de ações de  recursos, considere dividir seu controlador  em dois controladores menores.

## # Injeção de dependência e controladores

#### # Injeção de Construtor 

O contêiner de serviço Laravel é usado para resolver todos os controladores Laravel. Como resultado, você pode indicar qualquer dependência que seu controlador possa precisar em seu construtor. As dependências declaradas serão automaticamente resolvidas e injetadas na instância do controlador:

```php
<?php
namespace App\Http\Controllers;
use App\Repositories\UserRepository;
class UserController extends Controller
{
   /**
    * The user repository instance.
    */
   protected $users;
 
   /**
    * Create a new controller instance.
    *
    * @param  \App\Repositories\UserRepository  $users
    * @return void
    */
   public function __construct(UserRepository $users)
   {
          $this->users = $users;
   }
}
```

#### # Injeção de Método 

Além da injeção de construtor, você também pode digitar dependências nos métodos do seu controlador. Um caso de uso comum para injeção de método é injetar a `Illuminate\Http\Request` instância em seus métodos do controlador:

```php
?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
class UserController extends Controller
{
   /**
    * Store a new user.
    *
    * @param  \Illuminate\Http\Request  $request
    * @return \Illuminate\Http\Response
    */
   public function store(Request $request)
   {
       $name = $request->name;
       //
   }
}
```

Se o método do controlador também estiver esperando a entrada de um parâmetro de rota, liste seus argumentos de rota após suas outras dependências. Por exemplo, se sua rota for definida assim:

```php
use App\Http\Controllers\UserController;
Route::put('/user/{id}', [UserController::class, 'update']);
```

Você ainda pode dar dicas de tipo `Illuminate\Http\Request` acessar seu id parâmetro definindo seu método de controlador da seguinte forma:

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
class UserController extends Controller
{
   /**
    * Update the given user.
    *
    * @param  \Illuminate\Http\Request  $request
    * @param  string  $id
    * @return \Illuminate\Http\Response
    */
   public function update(Request $request, $id)
   {
       //
   }
}
```



