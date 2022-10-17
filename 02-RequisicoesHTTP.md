# #Requisições HTTP <h1>

## #Introdução <h2>
A classe **Illuminate\Http\Request** do Laravel fornece uma maneira orientada a objetos de interagir com a solicitação HTTP atual que está sendo tratada pelo sua aplicação, bem como recuperar o input (entrada de dados), os cookies e os arquivos que foram enviados com a solicitação.

## #Interagindo com a solicitação <h2>

### #Acessando a solicitação <h3>

Para obter uma instância da solicitação HTTP atual por meio de injeção de dependência, você deve digitar a classe **Illuminate\Http\Request** em na `clojure` da rota ou na ação do controlador. A instância `Request` recebida será automaticamente injetada pelo `Service Container` Laravel:

~~~php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Armazenar um novo usuário.
     * Store a new user.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');
 
        //
    }
}
~~~

Como mencionado, você também pode digitar a classe **Illuminate\Http\Request** em um `clojure` de rota. O `Service Container` injetará automaticamente a Request recebida na clojure quando for executado:

~~~php
use Illuminate\Http\Request;
 
Route::get('/', function (Request $request) {
    //
});
~~~

### #Injeção de Dependência e Parâmetros de Rotas 

Se o método do seu controlador também estiver esperando a entrada de um parâmetro de rota, você deve listar seus parâmetros de rota após suas outras dependências. Por exemplo, se sua rota for definida assim:

~~~php
use App\Http\Controllers\UserController;
 
Route::put('/user/{id}', [UserController::class, 'update']);
~~~

Você ainda pode digitar o **Illuminate\Http\Request** e acessar seu ID parâmetro de rota definindo seu método controlador da seguinte maneira:

~~~php
<?php
 
namespace App\Http\Controllers;
 
use Illuminate\Http\Request;
 
class UserController extends Controller
{
    /**
     * Atualize o usuário especificado.
     * Update the specified user.
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
~~~

### # Request Path (caminho), Host e Método <h3>

A instância **Illuminate\Http\Request** fornece uma variedade de métodos para examinar a solicitação HTTP recebida e estende a classe
**Symfony\Component\HttpFoundation\Request**. Vamos discutir alguns dos métodos mais importantes abaixo.

### #Recuperando o Path (Caminho) da Solicitação

O método **path** retorna as informações do caminho da solicitação. Portanto, se a solicitação recebida for direcionada a **http://example.com/foo/bar**, o método **path** retornará **foo/bar**:

~~~php
$uri = $request->path();
~~~

### #Inspecionando o caminho/rota da solicitação

O método **is** permite verificar se o caminho da Request recebida corresponde a um determinado padrão. Você pode usar o caractere * como “coringa” ao utilizar este método:

~~~php
if ($request->is('admin/*')) {
    //
}
~~~

Usando o método **routeIs**, você pode determinar se a Request recebida corresponde a uma **rota nomeada**:

~~~php
if ($request->routeIs('admin.*')) {
    //
}
~~~

### #Recuperando o URL de solicitação

Para recuperar a URL completa da solicitação recebida, você pode usar os métodos **url** ou **fullUrl**. O método **url** retornará a URL sem a string de consulta, enquanto o método **fullUrl** inclui a string de consulta:

~~~php
$url = $request->url();
 
$urlWithQueryString = $request->fullUrl();
~~~

Se você quiser anexar dados de string de consulta à URL atual, você pode chamar o método **fullUrlWithQuery**. Este método mescla o array fornecido de variáveis de string de consulta com a string de consulta atual:

~~~php
$request->fullUrlWithQuery(['type' => 'phone']);
~~~

### #Recuperando o Host de Solicitação

Você pode recuperar o "host" da solicitação recebida por meio dos métodos **host**, **httpHost** e **schemeAndHttpHost**:

~~~php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
~~~

### #Recuperando o método de solicitação

O método **method** retornará o verbo HTTP da Request. Você pode usar o método **isMethod** para verificar se o verbo HTTP corresponde a uma determinada string:

~~~php
$method = $request->method();
 
if ($request->isMethod('post')) {
    //
}
~~~

### # Cabeçalhos da Request <h3>

Você pode recuperar um cabeçalho de Request da instância **Illuminate\Http\Request** usando o método **header**. Se o cabeçalho não estiver presente na solicitação, será retornado **null**. No entanto, o método **header** aceita um segundo argumento opcional que será retornado se o cabeçalho não estiver presente na Request:

~~~php
$value = $request->header('X-Header-Name');
 
$value = $request->header('X-Header-Name', 'default');
~~~

O método **hasHeader** pode ser usado para determinar se a Request contém um determinado cabeçalho:

~~~php
if ($request->hasHeader('X-Header-Name')) {
    //
}
~~~

Por conveniência, o método `bearerToken` pode ser usado para recuperar um token do cabeçalho `Authorization`. Se nenhum cabeçalho estiver presente, uma string vazia será retornada:

~~~php
$token = $request->bearerToken();
~~~

### # Solicitar endereço IP <h3>

O método **ip** pode ser usado para recuperar o endereço IP do cliente que fez a solicitação a sua aplicação:

~~~php
$ipAddress = $request->ip();
~~~

### # Negação de conteúdo <h3>

O Laravel fornece vários métodos para inspecionar os tipos de conteúdo solicitados da Request recebida por meio do cabeçalho **Accept**. Primeiro, o método **getAcceptableContentTypes** retornará um array contendo todos os tipos de conteúdo aceitos pela Request:

~~~php
$contentTypes = $request->getAcceptableContentTypes();
~~~

O método **accepts** aceita um array de tipos de conteúdo e retorna **true** se algum dos tipos de conteúdo for aceito pela Request. Caso contrário, **false** será retornado:

~~~php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
~~~

Você pode usar o método **prefers** para determinar qual tipo de conteúdo de um determinado array de tipos de conteúdo é preferido pela Request. Se nenhum dos tipos de conteúdo fornecidos for aceito pela Request, **null** será retornado:

~~~php
$preferred = $request->prefers(['text/html', 'application/json']);
~~~

Como muitos aplicativos servem apenas HTML ou JSON, você pode usar o método **expectsJson** para determinar rapidamente se a Request recebida espera uma resposta JSON:

~~~php
if ($request->expectsJson()) {
    // ...
}
~~~

### #Solicitações PSR-7 <h3>

O **padrão PSR-7** especifica interfaces para mensagens HTTP, incluindo Requests e Responses. Se você deseja obter uma instância de uma Request PSR-7 em vez de uma Request Laravel, primeiro você precisará instalar algumas bibliotecas. O Laravel usa o componente *Symfony HTTP Message Bridge* para converter Requests e Responses típicas do Laravel em implementações compatíveis com PSR-7:

~~~php
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
~~~

Depois de instalar essas bibliotecas, você pode obter uma Request PSR-7 indicando a interface da Request na clojure da rota ou controlador:

~~~php
use Psr\Http\Message\ServerRequestInterface;
 
Route::get('/', function (ServerRequestInterface $request) {
    //
});
~~~


> Se você retornar uma instância de resposta PSR-7 de uma rota ou controlador, ela será automaticamente convertida de volta para uma instância de resposta Laravel e exibida pela estrutura.


## # Input (Entrada) <h2>

### # Recuperando entrada <h3>

#### # Recuperando todos os dados de entrada

Você pode recuperar todos os dados de entrada da Request recebida como um `array` (matriz) usando o método `all`. Este método pode ser usado independentemente de a Request recebida ser de um formulário HTML ou uma Request XHR:

~~~php
$input = $request->all();
~~~

Usando o método `collect`, você pode recuperar todos os dados de entrada da Request recebida como uma `Collection`:

~~~php
$input = $request->collect();
~~~

O método `collect` também permite recuperar um subconjunto da entrada da Request recebida como uma coleção:

~~~php
$request->collect('users')->each(function ($user) {
    // ...
});
~~~

### #Recuperando valores de entrada

Usando alguns métodos simples, você pode acessar todas as entradas do usuário de sua instância `Illuminate\Http\Request` sem se preocupar com qual verbo HTTP foi usado para a Request. Independentemente do verbo HTTP, o método `input` pode ser usado para recuperar a entrada do usuário:

~~~php
$name = $request->input('name');
~~~

Você pode passar um valor padrão como segundo argumento para o método `input`. Este valor será retornado se o valor de entrada solicitado não estiver presente na solicitação:

~~~php
$name = $request->input('name', 'Sally');
~~~

Ao trabalhar com formulários que contêm entradas de array, use a notação "ponto" para acessar os arrays:

~~~php
$name = $request->input('products.0.name');
 
$names = $request->input('products.*.name');
~~~

Você pode chamar o método `input` sem nenhum argumento para recuperar todos os valores de entrada como um array associativo:

~~~php
$input = $request->input();
~~~

### #Recuperando a entrada da string de consulta

Embora o método `input` recupere valores de toda a carga útil da solicitação (incluindo a query string), o método `query` recuperará apenas valores da query string:

~~~php
$name = $request->query('name');
~~~

Se os dados do valor da query string solicitada não estiverem presentes, o segundo argumento para este método será retornado:

~~~php
$name = $request->query('name', 'Helen');
~~~

Você pode chamar o método `query` sem nenhum argumento para recuperar todos os valores da query string como um array (matriz) associativo:

~~~php
$query = $request->query();
~~~

### #Recuperando valores de entrada JSON

Ao enviar solicitações JSON para seu aplicativo, você pode acessar os dados JSON por meio do método `input`, desde que o cabeçalho `Content-Type` da Request esteja definido corretamente como `application/json`. Você pode até usar a sintaxe "ponto" para recuperar valores aninhados em arrays JSON:

~~~php
$name = $request->input('user.name');
~~~

### # Recuperando Valores de Entrada Stringable (encadeados)

Em vez de recuperar os dados de entrada da Request como uma `string` primitiva, você pode usar o método `string` para recuperar os dados da Request como uma instância de `Illuminate\Support\Stringable`:

~~~php
$name = $request->string('name')->trim();
~~~

### # Recuperando Valores Booleanos de Entrada

Ao lidar com elementos HTML como caixas de seleção, seu aplicativo pode receber valores "verdadeiros" que na verdade são strings. Por exemplo, "true" ou "on". Por conveniência, você pode usar o método `boolean` para recuperar esses valores como booleanos. O método `boolean` retorna `true` para 1, "1", true, "true", "on" e "yes". Todos os outros valores retornarão `false`:

~~~php
$archived = $request->boolean('archived');
~~~

### # Recuperando Valores de Entrada Date (data)

Por conveniência, os valores de entrada contendo datas/horas podem ser recuperados como instâncias Carbon usando o método `date`. Se a Request não contiver um valor de entrada com o nome fornecido, será retornado `null`:

~~~php
$birthday = $request->date('birthday');
~~~

O segundo e terceiro argumentos aceitos pelo método `date` podem ser usados para especificar o formato da data e o fuso horário, respectivamente:

~~~php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
~~~

Se o valor de entrada estiver presente, mas tiver um formato inválido, uma `InvalidArgumentException` será lançada; portanto, é recomendável validar a entrada antes de chamar o método `date`.

### # Recuperando Valores de Entrada de Enum (enumeração)

Os valores de entrada que correspondem às `enumerações do PHP` também podem ser recuperados da Request. Se a Request não contiver um valor de entrada com o nome fornecido ou a enumeração não tiver um valor de definida na enum que corresponda ao valor de entrada, será retornado `null`. O método `enum` aceita o nome do valor de entrada e a classe enum como seu primeiro e segundo argumentos:

~~~php
use App\Enums\Status;
 
$status = $request->enum('status', Status::class);
~~~

### # Recuperando entrada por meio de propriedades dinâmicas

Você também pode acessar a entrada do usuário usando propriedades dinâmicas na instância *`Illuminate\Http\Request`. Por exemplo, se um dos formulários do seu aplicativo contém um campo `name`, você pode acessar o valor do campo da seguinte forma:

~~~php
$name = $request->name;
~~~

Ao usar propriedades dinâmicas, o Laravel procurará primeiro o valor do parâmetro no *payload* da Request. Se não estiver presente, o Laravel irá procurar o campo nos parâmetros da rota correspondente.

### # Recuperando uma parte dos dados de entrada

Se você precisar recuperar um subconjunto dos dados de entrada, poderá usar os métodos `only` e `except`. Ambos os métodos aceitam um único `array` ou uma lista dinâmica de argumentos:

~~~php
$input = $request->only(['username', 'password']);
 
$input = $request->only('username', 'password');
 
$input = $request->except(['credit_card']);
 
$input = $request->except('credit_card');
~~~

> O método only retorna todos os pares chave/valor solicitados; no entanto, ele não retornará pares de chave/valor que não estão presentes na solicitação.

### # Determinando se a entrada está presente 

Você pode usar o método `has` para determinar se um valor está presente na Request. O método `has` retorna `true` se o valor estiver presente na Request:

~~~php
if ($request->has('name')) {
    //
}
~~~

Ao receber um array, o método `has` determinará se todos os valores especificados estão presentes:

~~~php
if ($request->has(['name', 'email'])) {
    //
}
~~~

O método `whenHas` executará o encerramento fornecido se um valor estiver presente na Request:

~~~php
$request->whenHas('name', function ($input) {
    //
});
~~~

Um segundo clojure pode ser passado para o método `whenHas` que será executado se o valor especificado não estiver presente na Request:

~~~php
$request->whenHas('name', function ($input) {
    //O valor "name" está presente...
    // The "name" value is present...
}, function () {
    //O valor "name" não está presente...
    // The "name" value is not present...
});
~~~

O método `hasAny` retorna `true` se algum dos valores especificados estiver presente:

~~~php
if ($request->hasAny(['name', 'email'])) {
    //
}
~~~

Se você deseja determinar se um valor está presente na Request e não está vazio, você pode usar o método `filled`:

~~~php
if ($request->filled('name')) {
    //
}
~~~

O método `whenFilled` executará a clojure fornecido se um valor estiver presente na Request e não estiver vazio:

~~~php
$request->whenFilled('name', function ($input) {
    //
});
~~~

Uma segunda clojure pode ser passado para o método `whenFilled` que será executado se o valor especificado não for "preenchido":

~~~php
$request->whenFilled('name', function ($input) {
    //O valor "name" está preenchido...
    // The "name" value is filled...
}, function () {
    //O valor "name" não está preenchido...
    // The "name" value is not filled...
});
~~~

Para determinar se uma determinada chave está ausente da Request, você pode usar o método `missing`:

~~~php
if ($request->missing('name')) {
    //
}
~~~

### # Mesclando Entrada Adicional<h3>

Às vezes, pode ser necessário mesclar manualmente entradas adicionais nos dados de input existentes da Request. Para fazer isso, você pode usar o método `merge`:

~~~php
$request->merge(['votes' => 0]);
~~~

O método `mergeIfMissing` pode ser usado para mesclar dados na Request se as chaves correspondentes ainda não existirem nos dados de entrada da Request:

~~~php
$request->mergeIfMissing(['votes' => 0]);
~~~

### # Entrada antiga <h3>

O Laravel permite que você mantenha a entrada de uma Request durante a próxima Request. Esse recurso é particularmente útil para preencher novamente os formulários após detectar erros de validação. No entanto, se você estiver usando os recursos de validação incluídos no Laravel, é possível que você não precise usar manualmente esses métodos de entrada flashing de sessão diretamente, pois alguns dos recursos de validação internos do Laravel os chamarão automaticamente.

### # Entrada flashing para a sessão

O método `flash` na classe `Illuminate\Http\Request` indicará os dados da request atual para a sessão de modo que fiquem disponíveis durante a próxima Request do usuário ao aplicativo:

~~~php
$request->flash();
~~~

Você também pode usar os métodos `flashOnly` e `flashExcept` para atualizar um subconjunto dos dados da Request para a sessão. Esses métodos são úteis para manter informações confidenciais, como senhas, fora da sessão:

~~~php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
~~~

### # Indicando (flashing) a entrada e redirecionando 

Como muitas vezes você vai querer fazer flash de entrada para a sessão e depois redirecionar para a página anterior, você pode facilmente encadear a entrada em um redirecionamento usando o método `withInput`:

~~~php
return redirect('form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
~~~

### # Recuperando Entrada Antiga

Para recuperar a entrada atualizada da Request anterior, invoque o método `old` em uma instância de `Illuminate\Http\Request`. O método `old` puxará os dados de entrada anteriormente exibidos da sessão:

~~~php
$username = $request->old('username');
~~~

O Laravel também fornece um *helper* global `old`. Se você estiver exibindo uma entrada antiga em um `template Blade`, é mais conveniente usar o `old` auxiliar para preencher novamente o formulário. Se não existir nenhuma entrada antiga para o campo fornecido, será retornado `null`:

~~~php
<input type="text" name="username" value="{{ old('username') }}">
~~~

### # Cookies <h3>

### # Recuperando cookies de Requests

Todos os cookies criados pelo framework Laravel são criptografados e assinados com um código de autenticação, ou seja, serão considerados inválidos caso tenham sido alterados pelo cliente. Para recuperar um valor de cookie da solicitação, use o método `cookie` em uma instância `Illuminate\Http\Request`:

~~~php
$value = $request->cookie('name');
~~~

## # Trimming de Entrada e Normalização <h2>

Por padrão, o Laravel inclui o middleware `App\Http\Middleware\TrimStrings` e `App\Http\Middleware\ConvertEmptyStringsToNull` na pilha de *middlewares* global do seu aplicativo. Esses *middlewares* são listados na pilha de middleware global pela classe `App\Http\Kernel`. Esses *middlewares* cortarão automaticamente todos os campos de string de entrada na solicitação, bem como converterão qualquer campo de string vazio em `null` (nulo). Isso faz com que você não precise se preocupar com normalização em suas rotas e controladores.

### Desativando a entrada Normalization (normalização)

Se você quiser desabilitar esse comportamento para todas as Request, você pode remover os dois middlewares da pilha de middleware do seu aplicativo removendo-os da propriedade `$middleware` de sua classe `App\Http\Kernel`.

Se você deseja desabilitar o corte de string e a conversão de string vazia para um subconjunto de Request da sua aplicação, você pode usar o método `skipWhen` oferecido por ambos os middlewares. Este método aceita uma Clojure que deve retornar `true` (verdadeiro) ou `false` (falso) para indicar se a normalização de entrada deve ser ignorada. Normalmente, o método `skipWhen` deve ser invocado no método `boot` do `AppServiceProvider` da sua aplicação.

~~~php
use App\Http\Middleware\ConvertEmptyStringsToNull;
use App\Http\Middleware\TrimStrings;
 
/**
 * Qualquer serviço de aplicações Bootstrap
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    TrimStrings::skipWhen(function ($request) {
        return $request->is('admin/*');
    });
 
    ConvertEmptyStringsToNull::skipWhen(function ($request) {
        // ...
    });
}
~~~

## # Arquivos 

### # Recuperando arquivos enviados 

Você pode recuperar arquivos carregados de uma instância `Illuminate\Http\Request` usando o método `file` ou usando propriedades dinâmicas. O método `file` retorna uma instância da classe `Illuminate\Http\UploadedFile`, que estende a classe PHP `SplFileInfo` e fornece vários métodos para interagir com o arquivo:

~~~php
$file = $request->file('photo');
 
$file = $request->photo;
~~~

Você pode determinar se um arquivo está presente na solicitação usando o método `hasFile`

~~~php
if ($request->hasFile('photo')) {
    //
}
~~~

### # Validando uploads bem-sucedidos 

Além de verificar se o arquivo está presente, você pode verificar se não houve problemas ao fazer o upload do arquivo pelo método `isValid`:

~~~php
if ($request->file('photo')->isValid()) {
    //
}
~~~

### # Caminhos de arquivo e extensões 

A classe `UploadedFile` também contém métodos para acessar o caminho totalmente qualificado do arquivo e sua extensão. O método `extension` tentará adivinhar a extensão do arquivo com base em seu conteúdo. Esta extensão pode ser diferente da extensão que foi fornecida pelo cliente:

~~~php
$path = $request->photo->path();
 
$extension = $request->photo->extension();
~~~

### # Outros métodos de arquivo 

Há uma variedade de outros métodos disponíveis em instâncias `UploadedFile`. Confira a [documentação da API](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php) da classe para obter mais informações sobre esses métodos.

### # Armazenando arquivos enviados <h3>

Para armazenar um arquivo carregado, você normalmente usará um de seus sistemas de arquivos configurados (`filesystems`) . A classe `UploadedFile` tem um método `store` que moverá um arquivo carregado para um de seus discos, que pode ser um local em seu sistema de arquivos local ou um local de armazenamento em nuvem como o Amazon S3.

O método `store` aceita o caminho onde o arquivo deve ser armazenado em relação ao diretório raiz configurado no sistema de arquivos. Esse caminho não deve conter um nome de arquivo, pois um ID exclusivo será gerado automaticamente para servir como nome de arquivo.

O método `store` também aceita um segundo argumento opcional para o nome do disco que deve ser usado para armazenar o arquivo. O método retornará o caminho do arquivo relativo à raiz do disco:

~~~php
$path = $request->photo->store('images');
 
$path = $request->photo->store('images', 's3');
~~~

Se você não quiser que um nome de arquivo seja gerado automaticamente, você pode usar o método `storeAs` que aceita o caminho, o nome do arquivo e o nome do disco como seus argumentos:

~~~php
$path = $request->photo->storeAs('images', 'filename.jpg');
 
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
~~~


> Para obter mais informações sobre armazenamento de arquivos no Laravel, confira a documentação completa de armazenamento de arquivos.


## # Configurando proxies (procurador) confiáveis

Ao executar seus aplicativos atrás de um balanceador de carga que delimita certificados TLS/SSL, você pode notar que sua aplicação às vezes não gera links HTTPS ao usar o *helper* `url`. Normalmente, isso ocorre porque seu aplicativo está encaminhando tráfego do balanceador de carga na porta 80 e não sabe que deve gerar links seguros.

Para resolver isso, você pode usar o `App\Http\Middleware\TrustProxies` middleware incluído em sua aplicação Laravel, que permite personalizar rapidamente os balanceadores de carga ou proxies nos quais seu aplicativo deve confiar. Seus proxies confiáveis devem ser listados como um array na propriedade `$proxies` deste middleware. Além de configurar os proxies confiáveis, você pode configurar o proxy `$headers` que deve ser confiável:

~~~php
<?php
namespace App\Http\Middleware;
 
use Illuminate\Http\Middleware\TrustProxies as Middleware;
use Illuminate\Http\Request;
 
class TrustProxies extends Middleware
{
    /**
     * O proxies(procurador) confiável para essa aplicação
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];
 
    /**
     * Cabeçalhos que devem ser usados para detectar proxies.
     * The headers that should be used to detect proxies.
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
}
~~~

### # Confiando em todos os proxies 

Se você estiver usando a Amazon AWS ou outro provedor de balanceador de carga de "nuvem", talvez não saiba os endereços IP de seus balanceadores reais. Nesse caso, você pode usar * para confiar em todos os proxies:

~~~php
/**
 * Proxies confiáveis para essa aplicação.
 * The trusted proxies for this application.
 *
 * @var string|array
 */
protected $proxies = '*';
~~~

## #Configurando Hosts confiáveis <h2>

Por padrão, o Laravel responderá a todas as Requests que receber,
independentemente do conteúdo do cabeçalho `Host` da solicitação HTTP. Além disso, o valor do cabeçalho `Host` será usado ao gerar URLs absolutos para seu aplicativo durante uma Request da web.

Normalmente, você deve configurar seu servidor web, como Nginx ou Apache, para enviar apenas Requests para seu aplicativo que correspondam a um determinado nome de host. No entanto, se você não tiver a capacidade de personalizar seu servidor web diretamente e precisar instruir o Laravel a responder apenas a determinados nomes de host, você pode fazê-lo habilitando o `App\Http\Middleware\TrustHosts` middleware para sua aplicação.

O middleware `TrustHosts` já está incluído na pilha `$middleware` de seu aplicativo; No entanto, você deve descomentá-lo para que fique ativo. Dentro do método `hosts` deste middleware , você pode especificar os nomes de host aos quais seu aplicativo deve responder. As Requests recebidas com outros valores de cabeçalhos `Host` serão rejeitadas:

~~~php
/**
 * Obter o padrão host que deveria ser confiável.
 * Get the host patterns that should be trusted.
 *
 * @return array
 */
public function hosts()
{
    return [
        'laravel.test',
        $this->allSubdomainsOfApplicationUrl(),
    ];
}
~~~

O método auxiliar `allSubdomainsOfApplicationUrl` retornará uma expressão regular correspondente a todos os subdomínios do valor de configuração `app.url` do seu aplicativo. Esse método auxiliar fornece uma maneira conveniente de permitir todos os subdomínios do seu aplicativo ao criar um aplicativo que utiliza subdomínios coringa.
