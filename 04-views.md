# # Views

#### Introdução

#### Criando e renderizando visualizações
- Diretórios de visualização instalada
- Criando a primeira visualização disponível
- Determinando se uma visualização existe

#### Passando dados para visualização
- Compartilhando dados com todas as visualizações 

#### Ver compositores 
- Ver criadores

#### Otimizando visualizações 



## # Introdução

Obviamente, não é prático retornar Documentos HTML como strings a partir de suas rotas e controladores. Felizmente, as *views* fornecem uma maneira prática de colocar todo o nosso HTML em arquivos separados. As *views* separam a lógica do controlador/aplicativo da lógica de apresentação e são armazenadas no diretório `resources/views`. Uma visualização simples pode ser algo assim:

```html
<!-- View stored in resources/views/greeting.blade.php -->
<html>
<body>
        <h1>Hello, {{ $name }}</h1>
    </body>

</html>
``` 

Como essa visualização é armazenada em `resources/views/greeting.blade.php`, podemos retorná-la usando o *helper* global `view` da seguinte forma:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});

```

## # Criando e renderizando visualizações

Você pode criar uma *view* colocando um arquivo com a extensão `.blade.php`  no diretório `resources/views` da sua aplicação. A extensão `.blade.php`  informa ao framework que o arquivo contém um template Blade. Os templates blade contêm HTML e diretivas Blade que permitem ecoar valores facilmente, criar instruções "if", iterar(loops) sobre dados e muito mais.

Depois de criar uma view, você pode retorná-la de uma das rotas ou controladores de sua aplicação usando o helper global `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

As views também podem ser retornadas usando o Facade `View` :

```php
use Illuminate\Support\Facades\View;
return View::make('greeting', ['name' => 'James']);
``` 

Como é possível notar, o primeiro argumento passado para o heloper `view` corresponde ao nome do arquivo da view no diretório `resources/views`. O segundo argumento é um array de dados que deve ser disponibilizado para a exibição. Neste caso, estamos passando a variável name , que é exibida na view usando a sintaxe do Blade. 

### # Subdiretórios dentro do diretório padrão

As views também podem ser aninhadas em subdiretórios do diretório `resources/views`. A notação "ponto" pode ser usada para fazer referência as views nos subdiretórios. Por exemplo, se sua view estiver armazenada em `resources/views/admin/profile.blade.php`, você pode retorná-la de uma das rotas/controladores do seu aplicativo da seguinte forma:

```php
return view('admin.profile', $data);
```

> Nomes de diretórios dentro da pasta view não podem conterm o caracter `.`.

### # Criando a primeira visualização disponível

Usando o método `first` do facade `View`, você pode criar a primeira view que existe em um dado array de views. Este recurso é util se a sua aplicação ou pacote permite que as views sejam customizadas ou sobrescritas:

```php
use Illuminate\Support\Facades\View;
    return View::first(['custom.admin', 'admin'], $data);
```

### # Determinando se uma visualização existe

Se você precisar determinar se existe uma view, você pode usar o facade  `View` . O método `exists` retornará `true` se a view existir:

```php
use Illuminate\Support\Facades\View;
if (View::exists('emails.customer')) {
    //
}
```

## # Passando dados para visualização

Como você viu nos exemplos anteriores, um array de dados pode ser passado para views afim de  disponibilizar esses dados para a views:

```php
return view('greetings', ['name' => 'Victoria']);
```

Ao passar informações dessa maneira, os dados devem ser um array com pares chave/valor. Depois de fornecer dados a uma view, você pode acessar cada valor em sua view usando as chaves dos dados, como `<?php echo $name; ?>`.

Como alternativa para passar um array completo de dados para a função auxiliar `view`, você pode usar o método `with` para adicionar partes individuais de dados à exibição. Esse método  `with` retorna uma instância do objeto view para que você possa continuar encadeando métodos antes de retornar a view:

```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

### # Compartilhando dados com todas as visualizações 

Às vezes, você precisa compartilhar dados com todas as views de sua aplicação. Você pode fazer isso usando método `share` do Facade `View`. Normalmente, você deve usar a função `share` dentro do método `boot` de algum `ServiceProvider`. Você pode adicioná-los à classe `App\Providers\AppServiceProvider`ou gerar um provedor de serviços separado para abrigá-los:


```php
<?php
namespace App\Providers;
use Illuminate\Support\Facades\View;
class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
 
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }
}
```

##  # Compositores de View

Compositores de view são retornos de chamada ou métodos de classe que são chamados quando uma view é renderizada. Se você tiver dados que deseja vincular a uma view sempre que ela for renderizada, um compositor de view pode ajudá-lo a organizar essa lógica em um único local. Os compositores de view podem ser particularmente úteis se a mesma view for retornada por várias rotas ou controladores em seu aplicativo e sempre precisar de um dado específico.

Normalmente, os compositores de view serão registrados em um dos `service providers` do seu aplicativo . Neste exemplo, vamos supor que criamos um novo `App\Providers\ViewServiceProvider` para abrigar essa lógica.

Usaremos o método `composer` do Facade `View` para registrar o compositor de view . O Laravel não inclui um diretório padrão para compositores de view baseados em classes, então você pode organizá-los da maneira que preferir. Você pode, por exemplo, criar um diretório `app/View/Composers`  para hospedar todos os compositores de view do seu aplicativo:

```php
<?php
 
namespace App\Providers;
 
use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;
 
class ViewServiceProvider extends ServiceProvider
{
    /**
     * Registre qualqer serviço de aplicação
     *
     * @return void
     */
    public function register()
    {
        //
    }
 
    /**
     * Inicializa qualquer serviço de aplicação
     *
     * @return void
     */
    public function boot()
    {
        // Usando composer baseados em classes
        View::composer('profile', ProfileComposer::class);
 
        // Usando composer baseados em closures
        View::composer('dashboard', function ($view) {
            //
        });
    }
}
```

> Lembre-se, se você criar um novo service provider para adicionar seus registro de compositores de view, você precisará adicionar o service provider para o array `providers` no arquivo de configuração `config/app.php`.

Agora que registramos o compositor, o método `compose`  da classe `App\View\Composers\ProfileComposer` será executado toda vez que a view `profile` for renderizada. Vamos dar uma olhada em um exemplo da classe compositora:

```php
<?php
 
namespace App\View\Composers;
 
use App\Repositories\UserRepository;
use Illuminate\View\View;
 
class ProfileComposer
{
    /**
     * Implementa repositório do usuário
     *
     * @var \App\Repositories\UserRepository
     */
    protected $users;
 
    /**
     * Cria um novo compositor de profile
     *
     * @param  \App\Repositories\UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
 
    /**
     * Vincula dados para a view
     *
     * @param  \Illuminate\View\View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

Como você pode ver, todas os compositores de view são resolvidos no `service container`, portanto, você pode indicar quaisquer dependências necessárias no construtor de um compositor.

#### # Anexando um compositor a várias visualizações 

Você pode anexar um compositor de views a várias views ao mesmo tempo, passando um array de views como o primeiro argumento para o método `composer`:

```php
use App\Views\Composers\MultiComposer;
 
View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

O método `composer` também aceita o caractere como curinga, permitindo anexar um compositor a todas as views:

```php
View::composer('*', function ($view) {
    //
});
```

### # Criadores de View

Criadores de views são similares aos composidores de views; Entretanto, eles são executados imediatamente após a view é instanciada ao invés de esperar até a view estar prestes a ser renderizada. Para registrar um criador de view, use o método `creator`:


```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

## # Otimizando Views

Por padrão, as views do template Blade são compiladas sob demanda. Quando é executada uma Request que renderiza uma view, o Laravel determinará se existe uma versão compilada da view. Se o arquivo existir, o Laravel determinará se a view não compilada foi modificada mais recentemente do que a view compilada. Se a view compilada não existir, ou a view não compilada for modificada, o Laravel irá recompilar a vew.

Compilar views durante a Request pode ter um pequeno impacto negativo no desempenho, então o Laravel fornece o comando do Artisan `view:cache` para pré-compilar todas as view utilizadas por sua aplicação. Para aumentar o desempenho, você pode executar este comando como parte de seu processo de implantação:

```php
php artisan view:cache
``` 

Você pode usar o comando  `view:clear` para limpar o cache de views:

```php
php artisan view:clear
``` 
