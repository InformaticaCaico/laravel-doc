# **Visualizações** #


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



##### > Introdução

Obviamente, não é prático retornar strings inteiras de documentos HTML diretamente de suas rotas e controladores. Felizmente, as visualizações fornecem uma maneira prática de colocar todo o nosso HTML em arquivos separados. As visualizações separam a lógica do controlador/aplicativo da lógica da apresentação e são armazenadas no diretório `resources/views`. Uma visualização simples pode ser algo assim:
```
` <!-- View stored in resources/views/greeting.blade.php --> `

<html>

`   `<body>

`       `<h1>Hello, {{ $name }}</h1>

`   `</body>

</html>

``` 

Como essa visualização é armazenada em `resources/views/greeting.blade.php`, podemos retorná-la usando o view auxiliar global da seguinte forma:

```
Route::get('/', function () {

return view('greeting', ['name' => 'James']);

});

```

##### > Criando e renderizando visualizações

Você pode criar uma view colocando um arquivo com a extensão `.blade.php`  no diretório `resources/views` da sua aplicação. A extensão `.blade.php`  informa ao framework que o arquivo contém um template Blade. Os modelos blade contêm HTML e diretivas Blade que permitem ecoar valores facilmente, criar instruções "if", reiterar sobre dados e muito mais.

Depois de criar uma view, você pode retorná-la de uma das rotas ou controladores de sua aplicação usando o `view`auxiliar global:
``` 
Route::get('/', function () {

`   `return view('greeting', ['name' => 'James']);

});
```


As views também podem ser retornadas usando a fachada `View` :

```
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
``` 

Como é possível notar, o primeiro argumento passado para o auxiliar `view` corresponde ao nome do arquivo de visualização no diretório `resources/views`. O segundo argumento é uma matriz de dados que deve ser disponibilizada para a exibição. Neste caso, estamos passando a variável name , que é exibida na view usando a sintaxe do Blade. 

- #####  Diretórios de visualização instalada

As visualizações também podem ser aninhadas em subdiretórios do diretório `resources/views`. A notação "ponto" pode ser usada para fazer referência a visualizações instaladas. Por exemplo, se sua visualização estiver armazenada em `resources/views/admin/profile.blade.php`, você pode retorná-la de uma das rotas/controladores do seu aplicativo da seguinte forma:

```
return view('admin.profile', $data);
```
- ##### Criando a primeira visualização disponível

Usando o método `View` da fachada `first`, é possível criar a  primeira visualização que existe em uma determinada matriz de visualizações. Isso pode ser útil se seu aplicativo ou pacote permitir que as visualizações sejam personalizadas ou substituídas:
```
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
``` 
- ##### Determinando se uma visualização existe

Se você precisar determinar se existe uma view, você pode usar a fachada `View` . O método `exists` retornará `true` se a view existir:
```

use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {

`   `//

}
```

#### > Passando dados para visualização

Como você viu nos exemplos anteriores, uma matriz de dados pode ser passada para  visualizações afim de  disponibilizar esses dados para a visualização:
```
return view('greetings', ['name' => 'Victoria']);
```

Ao passar informações dessa maneira, os dados devem ser um array com pares chave/valor. Depois de fornecer dados a uma exibição, você pode acessar cada valor em sua exibição usando as chaves dos dados, como `<?php echo $name; ?>`.

Como alternativa para passar uma matriz completa de dados para a função auxiliar `view` , você pode usar o método `with` para adicionar partes individuais de dados à exibição. Esse método  `with` retorna uma instância do objeto view para que você possa continuar encadeando métodos antes de retornar a view:

```
return view('greeting')

->with('name', 'Victoria')

->with('occupation', 'Astronaut');
```

- ##### Compartilhando dados com todas as visualizações 

Pode ocorrer de você precisar compartilhar dados com todas as exibições renderizadas pelo seu aplicativo, e você pode fazer isso usando o método `View` da fachada `share`. Normalmente, você deve fazer chamadas para o método  `share` dentro do método de um provedor de serviços `boot`. Você pode adicioná-los à classe `App\Providers\AppServiceProvider`ou gerar um provedor de serviços separado para abrigá-los:

```

<?php



namespace App\Providers;



use Illuminate\Support\Facades\View;



class AppServiceProvider extends ServiceProvider

{

`   `/\*\*

`    `\* Register any application services.

`    `\*

`    `\* @return void

`    `\*/

`   `public function register()

`   `{

`       `//

`   `}



`   `/\*\*

`    `\* Bootstrap any application services.

`    `\*

`    `\* @return void

`    `\*/

`   `public function boot()

`   `{

`       `View::share('key', 'value');

`   `}

}
```

#### > Ver compositores 

Compositores de exibição são retornos de chamada ou métodos de classe que são chamados quando uma exibição é renderizada. Se você tiver dados que deseja vincular a uma exibição sempre que essa exibição for renderizada, um compositor de exibição poderá ajudá-lo a organizar essa lógica em um único local. Os compositores de exibição podem ser particularmente úteis se a mesma exibição for retornada por várias rotas ou controladores em seu aplicativo e sempre precisar de um dado específico.

Normalmente, os compositores de visualização serão registrados em um dos provedores de serviço do seu aplicativo . Neste exemplo, vamos supor que criamos um novo `App\Providers\ViewServiceProvider` para abrigar essa lógica.

Usaremos o método `View` da fachada `composer` para registrar a composer view . O Laravel não inclui um diretório padrão para compositores de visualização baseados em classes, então você pode organizá-los da maneira que preferir. Você pode, por exemplo, criar um diretório `app/View/Composers`  para hospedar todos os compositores de visualização do seu aplicativo:
```
<?php



namespace App\Providers;



use App\View\Composers\ProfileComposer;

use Illuminate\Support\Facades\View;

use Illuminate\Support\ServiceProvider;



class ViewServiceProvider extends ServiceProvider

{

`   `/\*\*

`    `\* Register any application services.

`    `\*

`    `\* @return void

`    `\*/

`   `public function register()

`   `{

`       `//

`   `}



`   `/\*\*

`    `\* Bootstrap any application services.

`    `\*

`    `\* @return void

`    `\*/

`   `public function boot()

`   `{

`       `// Using class based composers...

`       `View::composer('profile', ProfileComposer::class);



`       `// Using closure based composers...

`       `View::composer('dashboard', function ($view) {

`           `//

`       `});

`   `}

}
```

Agora que registramos o compositor, o método `compose`  da classe `App\View\Composers\ProfileComposer` será executado toda vez que a view profile  for renderizada. Vamos dar uma olhada em um exemplo da classe composer:
```
<?php 

namespace App\View\Composers;

use App\Repositories\UserRepository;

use Illuminate\View\View; 

class ProfileComposer

{

`   `/\*\*

`    `\* The user repository implementation.

`    `\*

`    `\* @var \App\Repositories\UserRepository

`    `\*/

`   `protected $users;

`   `/\*\*

`    `\* Create a new profile composer.

`    `\*

`    `\* @param  \App\Repositories\UserRepository  $users

`    `\* @return void

`    `\*/

`   `public function \_\_construct(UserRepository $users)

`   `{

`       `$this->users = $users;

`   `}

`   `/\*\*

`    `\* Bind data to the view.

`    `\*

`    `\* @param  \Illuminate\View\View  $view

`    `\* @return void

`    `\*/

`   `public function compose(View $view)

`   `{

`       `$view->with('count', $this->users->count());

`   `}

}
```

Todos os compositores de exibição são resolvidos por meio do contêiner de serviço, portanto, você pode indicar quaisquer dependências necessárias no construtor de um compositor.

- ##### Anexando um compositor a várias visualizações 

Você pode anexar um compositor de visualizações a várias visualizações ao mesmo tempo, passando uma matriz de visualizações como o primeiro argumento para o método `composer`:
```
use App\Views\Composers\MultiComposer;



View::composer(

`   `['profile', 'dashboard'],

`   `MultiComposer::class

);
```

O método `composer` também aceita o caractere como curinga, permitindo anexar um compositor a todas as visualizações:
```
` `View::composer('\*', function ($view) {

`   `//

});
```

- ##### Ver criadores

Apesar dos “criadores” de visualização apresentarem bastante semelhanças com os compositores de visualização, a diferença está que os primeiros são executados imediatamente após a visualização ser instanciada, em vez de esperar até que a visualização esteja prestes a ser renderizada. Então, para registrar um criador de visualizações, use o método `creator`:
```
use App\View\Creators\ProfileCreator;

use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

#### > Otimizando visualizações

Por padrão, as visualizações do modelo Blade são compiladas sob demanda. Quando é executada uma requisição que renderiza uma visão, o Laravel determinará se existe uma versão compilada da visão. Se o arquivo existir, o Laravel determinará se a visualização não compilada foi modificada mais recentemente do que a visualização compilada. Se a visão compilada não existir, ou a visão não compilada for modificada, o Laravel irá recompilar a visão.

Compilar visualizações durante a solicitação pode ter um pequeno impacto negativo no desempenho, então o Laravel fornece o `view:cache` comando Artisan para pré-compilar todas as visualizações utilizadas por sua aplicação. Para aumentar o desempenho, você pode executar este comando como parte de seu processo de implantação:
```
php artisan view:cache
``` 

Você pode usar o comando  `view:clear` para limpar o cache de visualização:
```
php artisan view:clear
``` 
