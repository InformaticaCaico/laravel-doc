# # Sessão HTTP

## # Introdução

Dado que aplicações orientadas a HTTP são sem estados, as sessões fornecem uma maneira de armazenar informações sobre o usuário através de múltiplas requisições. As informações do usuário geralmente são colocadas em um local de armazenamento/backend persistente que pode ser acessado a partir de requisições subsequentes.

O Laravel possui uma variedade de backends de sessão que são acessadas por uma API expressiva e unificada. Suporte para backends populares como `Memcached`, `Redis` e bancos de dados está incluso.

### # Configuração

O arquivo de configuração de sessão da sua aplicação é armazenado em `config/session.php`. Certifique-se de conferir as opções disponíveis para você nesse arquivo. Por padrão, o Laravel é configurado para utilizar o driver de sessão `file`, que terá um bom funcionamento na maioria das aplicações. Se sua aplicação realizar balanceamento de carga em muitos servidores web, você deve escolher um local de armazenamento centralizado que todos os servidores podem acessar, como Redis ou um banco de dados.

A opção de configuração `driver` define onde os dados da sessão vão ser armazenados a cada requisição. O Laravel acompanha vários drivers excelentes:

-   `file` - as sessões são armazenadas em `storage/framework/sessions`.
-   `cookie` - as sessões são armazenadas em cookies seguros e encriptados.
-   `database` - as sessões são armazenadas em um banco de dados relacional.
-   `memcached` / `redis` - as sessões são armazenadas em um desses repositórios ágeis, baseados em cache.
-   `dynamodb` - as sessões são armazenadas em um banco de dados DynamoDB da AWS.
-   `array` - as sessões são armazenadas em um array do PHP e não serão persistentes.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg) 
>
> O driver de array é usado principalmente durante testes e previne que os dados armazenados na sessão se tornem permanentes.

### # Pré-requisitos do Driver

#### # Banco de Dados

Quando utilizar o driver de sessão `database`, você precisará criar uma tabela para conter seus dados. Um exemplo de declaração `Schema` para a tabela se encontra abaixo:

```php
Schema::create('sessions', function ($table) {    
	$table->string('id')->primary();    
	$table->foreignId('user_id')->nullable()->index();    
	$table->string('ip_address', 45)->nullable();    
	$table->text('user_agent')->nullable();    
	$table->text('payload');    
	$table->integer('last_activity')->index();
});
```

Você pode usar o comando do Artisan `session:table` para gerar essa migração. Para saber mais sobre migrações de banco de dados, consulte a documentação de migração completa.

```php
php artisan session:table 

php artisan migrate
```

#### # Redis

Antes de usar as sessões de Redis com o Laravel, é preciso ou instalar a extensão do PHP, PhpRedis, via PECL, ou instalar o pacote `predis/predis` (~1.0) via Composer. Para mais informações sobre como configurar o Redis, consulte a documentação Redis do Laravel.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
> 
> No arquivo de configuração `session`, a opção `connection` pode ser usada para especificar qual conexão do Redis é utilizada pela sessão.

## # Interagindo com a Sessão

### # Recuperando Dados

Existem duas maneiras principais de trabalhar com os dados de uma sessão no Laravel: por meio do helper global `session` e de uma instância `Request`. Primeiramente, vamos ver como acessar a sessão a partir da instância `Request`, a qual pode ser especificado o seu tipo em um clojure de rota ou em um método do controlador. Lembre-se, as dependências do método controlador são automaticamente injetadas a partir do `service container`.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
	/**

	* Mostrar o perfil de determinado usuário.

	*

	* @param Request $request

	* @param int $id

	* @return Response

	*/

	public function show(Request $request, $id)

	{
		$value = $request->session()->get('key');

		//

	}
}
```

Quando recuperar um item da sessão, você também pode atribuir um valor padrão como o segundo argumento do método `get`. O valor padrão vai ser retornado se a chave especificada não existir na sessão. Caso atribua uma closuje como o valor padrão do método `get` e a chave requisitada não existir, a closuje vai ser executado e seu resultado retornado:

```php
$value = $request->session()->get('key', 'default'); 

$value = $request->session()->get('key', function () {    
	return 'default';
});
```

#### O helper Global Session

Você também pode utilizar a função global do PHP `session` para recuperar e armazenar dados em uma sessão. Quando o helper `session` é chamado com um único argumento de string, ele retornará o valor da chave daquela sessão. Quando o helper é chamado com um array de chaves/pares de valores, esses valores serão armazenados na sessão:

```php
Route::get('/home', function () {
	// Recupera uma porção dos dados da sessão...    
	$value = session('key');     

	// Especifica um valor padrão...    
	$value = session('key', 'default');     

	// Armazena uma porção de dados na sessão...    
	session(['key' => 'value']);
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
> 
> Existe uma pequena diferença prática entre acessar a sessão via instância de requisição HTTP e usando o helper global `session`. Ambos os métodos são testáveis a partir do método `assertSessionHas`, que está disponível em todos os seus casos de teste.

#### # Recuperando Todos os Dados da Sessão

Se quiser recuperar todos os dados na sessão, você pode usar o método `all`:

```php
$data = $request->session()->all();
```

#### # Determinando Se Um Item Existe em Uma Sessão

Para determinar se um item está presente na sessão, use o método `has`. O método `has` retorna `true` se o item está presente e não é `null`:

```php
if ($request->session()->has('users')) {    
	//
}
```

Para determinar se um item está presente na sessão, mesmo que seu valor seja `null`, você pode usar o método `exists`:

```php
if ($request->session()->exists('users')) {    
	//
}
```

Para determinar se um item não está presente na sessão, você pode usar o método `missing`. O método `missing` retorna `true` se o item é `null` ou se o item não está presente:

```php
if ($request->session()->missing('users')) {    
	//
}
```

### # Armazenando Dados

Para armazenar dados na sessão, você vai usar, normalmente, o método `put` da instância de requisição ou o helper global `session`:

```php
// A partir de uma instância de requisição…
$request->session()->put('key', 'value'); 

// A partir do auxiliar global de sessão…
session(['key' => 'value']);
```

#### # Acrescentando Valores Para o Array da Sessão

O método `push` pode ser usado para acrescentar um novo valor em algum valor da sessão que é um array. Por exemplo, se a chave `user.teams` contém um array de nomes de times, você pode acrescentar um novo valor no array assim:

```php
$request->session()->push('user.teams', 'developers');
```

#### # Recuperando e Excluindo Um Item

O método `pull` vai recuperar e excluir um item da sessão em uma única declaração: 

```php
$value = $request->session()->pull('key', 'default');
```

#### # Incrementando e Decrementando Valores da Sessão

Se os dados da sua sessão contêm um inteiro que você deseja incrementar ou decrementar, você pode usar os métodos `increment` e `decrement`:

```php
$request->session()->increment('count'); 

$request->session()->increment('count', $incrementBy = 2); 

$request->session()->decrement('count'); 

$request->session()->decrement('count', $decrementBy = 2);
```

### # Dados Flash

Às vezes, você talvez queira armazenar alguns itens na sessão para a próxima requisição. Pode fazê-lo usando o método `flash`. Dados armazenados na sessão utilizando esse método ficarão disponíveis imediatamente e durante a requisição HTTP posterior. Após a requisição HTTP seguinte, os dados flash serão excluídos. Dados flash são úteis principalmente para mensagens de status de curta duração:

```php
$request->session()->flash('status', 'Tarefa realizada com sucesso!');
```

Se precisar manter seus dados flash para várias requisições, você pode usar o método `reflash`, o qual vai manter todos os dados flash para uma requisição adicional. Se precisar apenas manter dados flash específicos, pode usar o método `keep`:

```php
$request->session()->reflash(); 

$request->session()->keep(['username', 'email']);
```

Para manter os dados flash somente para a requisição atual, você pode usar o método `now`:

```php
$request->session()->now('status', 'Tarefa realizada com sucesso!');
```

### # Excluindo Dados

O método `forget` vai remover uma porção dos dados de uma sessão. Se deseja remover todos os dados da sessão, você pode usar o método `flush`:

```php
// Remover uma única chave...
$request->session()->forget('name'); 

// Remover múltiplas chaves...
$request->session()->forget(['name', 'status']); 

$request->session()->flush();
```

### # Regenerando o ID da Sessão

A regeneração do ID da sessão é geralmente realizada para impedir que usuários mal-intencionados façam um ataque de [fixação de sessão](https://owasp.org/www-community/attacks/Session_fixation) na sua aplicação.

O Laravel automaticamente regenera o ID da sessão durante a autenticação se você estiver usando um dos kits iniciais de aplicação do Laravel ou o Laravel Fortify. No entanto, caso você precisar regenerar o ID da sessão manualmente, use o método `regenerate`:

```php
$request->session()->regenerate();
```

Caso necessitar regenerar o ID da sessão e remover todos os seus dados em uma única declaração, pode usar o método `invalidate`:

```php
$request->session()->invalidate();
```

## # Bloqueio de Sessão

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
> 
> Para utilizar o bloqueio de sessão, sua aplicação deve estar usando um driver de cache que suporte atomic locks. Atualmente, esses drivers de cache incluem os drivers `memcached`, `dynamodb`, `redis` e `database`. Além disso, você não pode utilizar o driver de sessão `cookie`.

Por padrão, o Laravel permite que requisições usando a mesma sessão sejam executadas simultaneamente. Então, por exemplo, se você utiliza uma biblioteca HTTP de JavaScript para fazer duas requisições HTTP para sua aplicação, ambas irão executar ao mesmo tempo. Para muitas aplicações, isso não é um problema. No entanto, podem ocorrer perdas de dados em pequenos subconjuntos de aplicações que fazem requisições simultâneas para dois endpoints diferentes da aplicação que gravam dados na sessão.

Para amenizar isso, o Laravel fornece uma funcionalidade que permite limitar requisições simultâneas para uma determinada sessão. Para começar, você pode simplesmente adicionar o método `block` na sua definição de rota. Neste exemplo, uma requisição para o endpoint `/profile` adquiriria um bloqueio de sessão. Enquanto o bloqueio segura, quaisquer requisições que vierem para os endpoints `/profile` ou `/order` que compartilham o mesmo ID de sessão irão esperar que a primeira requisição termine de executar antes de continuar a execução das outras:

```php
Route::post('/profile', function () {
	//
})->block($lockSeconds = 10, $waitSeconds = 10) 

Route::post('/order', function () {
	//
})->block($lockSeconds = 10, $waitSeconds = 10)
```

O método `block` aceita dois argumentos opcionais. O primeiro argumento aceito pelo método `block` é o número máximo de segundos que o bloqueio de sessão deve durar antes de ser liberado. É claro que, se a requisição terminar sua execução antes do tempo, o bloqueio será dispensado mais cedo.

O segundo argumento aceito pelo método `block` é o número de segundos que uma requisição deve esperar ao tentar obter um bloqueio de sessão. Um `Illuminate\Contracts\Cache\LockTimeoutException` será lançado se a requisição não for capaz de obter um bloqueio de sessão dentro do número de segundos dado.

Caso nenhum desses argumentos for passado, o bloqueio vai ser obtido por um máximo de 10 segundos e requisições vão esperar um máximo de 10 segundos enquanto tentam obter um bloqueio:

```php
Route::post('/profile', function () {
	//
})->block()
```

## # Adicionando Drivers de Sessão Personalizados

#### # Implementando o Driver

Se nenhum dos drivers de sessão existentes suprem as necessidades da sua aplicação, o Laravel permite que você importe o seu próprio manipulador de sessão. Seu driver de sessão personalizado deve implementar o `SessionHandlerInterface` integrado do PHP. Esta interface possui apenas alguns métodos simples. Uma implementação do MongoDB se parece com o seguinte:

```php
<?php 

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{    
	public function open($savePath, $sessionName) {}    
	public function close() {}    
	public function read($sessionId) {}    
	public function write($sessionId, $data) {}    
	public function destroy($sessionId) {}    
	public function gc($lifetime) {}
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
> 
> O Laravel não conta com um diretório para as suas extensões. Você é livre para colocá-las onde quiser. Nesse exemplo, criamos um diretório `Extensions` para guardar o `MongoSessionHandler`.

Como o propósito destes métodos não é facilmente compreensível, vamos abordar rapidamente o que cada um deles faz: 

-  O método `open` seria normalmente utilizado em sistemas de armazenamento baseado em arquivos. Já que o Laravel acompanha um driver de sessão `file`, raramente você vai precisar inserir algo nesse método. Pode deixá-lo vazio.
-  O método `close`, assim como o método `open`, também pode comumente ser desconsiderado. Para a maioria dos drivers, ele não é necessário.
-  O método `read` deve retornar a versão em string dos dados da sessão associados ao `$sessionId` fornecido. Não há necessidade de realizar nenhuma serialização ou outra codificação quando recuperar ou armazenar dados de sessão no seu driver, já que o Laravel fará a serialização para você.
-  O método `write` deve gravar a string `$data` fornecida associada ao `$sessionId` em algum sistema de armazenamento persistente, como MongoDB ou outro sistema de armazenamento de sua escolha. Novamente, você não deve executar nenhuma serialização - o Laravel já terá tratado isso para você.
-  O método `destroy` deve remover os dados associados ao `$sessionId` de um armazenamento persistente.
-  O método `gc` deve destruir todos os dados da sessão que são mais antigos que o `$lifetime` fornecido, que é a marca temporal (timestamp) do UNIX. Para sistemas com expiração automática como Memcached e Redis, esse método pode ser deixado vazio.

#### # Registrando o Driver

Uma vez que seu driver tenha sido implementado, você está pronto para registrá-lo com o Laravel. Para adicionar drivers adicionais para a sessão de backend do Laravel, você pode usar o método `extend` fornecido pelo facade `Session`. Chame o método `extend` a partir do método `boot` de um provedor de serviços. Pode-se fazê-lo pelo `App\Providers\AppServiceProvider` existente ou criando um provedor completamente novo:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider; 

class SessionServiceProvider extends ServiceProvider
{
	/**     
	* Registrar qualquer serviço da aplicação.     
	*
	* @return void
	*/    
	public function register()    
	{
		//    
	}
	     
	/**     
	* Iniciar qualquer serviço da aplicação.
	*     
	* @return void     
	*/    
	public function boot()    
	{        
		Session::extend('mongo', function ($app) {
		// Retorna uma implementação do SessionHandlerInterface…
		return new MongoSessionHandler;        
	});    
	}
}
```

Assim que seu driver tiver sido registrado, você poderá usar o driver `mongo` no seu arquivo de configuração `config/session.php`.
