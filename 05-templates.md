# Introdução

Blade é o mecanismo de modelagem simples incluído no laravel, todos os modelos do Blade são compilados em código PHP simples e armazenados até serem modificados, ele não sobrecarrega o seu aplicativo. Os arquivos de template blade usam `.blade.php` e são armazenados no diretório `resources/views`.

As views podem retornar de rotas ou controladores utilizando `view` que é o helper global, os dados podem ser passados para o helper como segundo argumento:

``` php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});

```

> Quer elevar seus templates blades para o próximo nível e construri aplicações com interfaces dinâmicas? Teste o [Laravel Livewire](https://laravel-livewire.com/).

# # Exibindo dados

Você pode mostrar os dados passados para o template através do uso de duas chaves. Por exemplo, dada a seguinte rota:

``` php
Route::get('/', function () {
   return view('welcome', ['name' => 'Samantha']);
});
```

A exibição da variável `name` é dada por:

``` php
Hello, {{ $name }}.
``` 

> As declarações usando chaves são enviadas automaticamente para a função PHP `htmlspecialchars` com o objtivo de previnir XSS ataques.

Você não está limitado a mostrar o conteúdo de variáveis passadas para as view. Você também pode ecoar os resultados de qualquer função PHP. De fato, você pode por qualquer código PHP que você desejar dentro de uma declaração echo do Blade.


``` php 
The current UNIX timestamp is {{ time() }}.
```

## # Codificação de Entidade HTML

O Blade codificará duplamente as entidades HTML, caso queira desabilitar a dupla codificação, use o métod `Blade::withoutDoubleEncoding` a partir do método `boot` do `AppServiceProvider`:

``` php
namespace App\Providers;
 
use Illuminate\Support\Facades\Blade;
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
        Blade::withoutDoubleEncoding();
    }
}
```

### # Exibindo dados sem escape

Por padrão, as  instruções Blade `{{ }}` são enviadas automaticamente através da função do PHP `htmlspecialcharspara` evitar ataques do tipo XSS. Se você não quiser que seus dados sejam escapados, você pode usar a seguinte sintaxe:

``` php
Hello, {!! $name !!}.
```

> Seja muito cuidadoso quando ecoar conteúdo que é fornecido pelos usuários da sua aplicação. VocÊ deve sempre as chaves duplas para previnir ataques quando mostrar dados enviados por usuários.

## # Blade e Frameworks JavaScript

Dado que muitos frameworks JavaScripts também usam chaves duplas para indicar uma dada expressão seja aplicada no browser, você deve usar o símbolo `@` para informar ao Blade que renderize uma expressão que permanessa intocado. Por exemplo:


```html
<h1>Laravel</h1> 
Hello, @{{ name }}.
```

No exemplo acima, o símbolo `@` será removido, porém a expressão `{{ name }}` permanecerá inalterada pela engine Blade, permitindo que seja renderizada pelo JavaScript.

O símbolo `@` também pode ser usado para escapar das diretivas Blade:

``` php
{{-- Blade template --}}
@@if()
 
<!-- HTML output -->
@if()
```

#### # Renderizando JSON

Algumas vezes ao passar um array para a sua view afim de renderizá-lo como JSON para iniciar uma variável JavaScript. Por exemplo:

``` php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Porém, em vez de chamar manualmente `json_encode`, utilize `Illuminate\Support\Js::from`. O método `from` aceita os mesmos argumentos da função `json_encode`, mas isso garantirá  que o JSON resultante seja escapado adequadamente para inclusão entre aspas HTML. O método `from` retorná uma string da declaração JavaScript `JSON.parse` que erá convertida em um dado objeto ou array dentro de um objeto JavaScript válido:

```php
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
``` 

As versões recentes dos esqueletos de aplicação Laravel incluem uma Facade `Js` , que fornece acesso conveniente a essa funcionalidade dentro dos templates Blade:

``` php
<script>
    var app = {{ Js::from($array) }};
</script> 

> Você deve utilizar somente o método `Js::from` para renderizar as variáveis com JSON. O template blade e baseado em expressões regulares e tentar passar um expressão coplexa para a diretiva pode causar falhas inesperadas.

``` 
#### # A Diretiva _@verbatim_

Se você está mostrando variáveis em uma larga porção dos templates, você pode envolver a declaração HTMl dentro da diretiva `@verbatim` de modo que você não precisa prefixar cada declaração blade echo com o símbolo `@`:

``` php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
``` 

## # Diretrizes de Blade 

Além da herança de modelos e exibição de dados, o Blade também fornece atalhos convenientes para estruturas de controle PHP comuns, como instruções condicionais e loops. Esses atalhos fornecem uma maneira muito limpa e concisa de trabalhar com estruturas de controle PHP, ao mesmo tempo em que permanecem familiares aos seus equivalentes PHP.

### # Declarações If

A criação das instruções **if** podem ser utilizadas com as diretivas **@if**, **@elseif**, **@else** . **@endif**. Essas diretivas agem de forma idêntica às contrapartes PHP:

``` php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Por conveniência, o Blade também fornece uma diretiva @unless:

``` php
@unless (Auth::check())
    You are not signed in.
@endunless

```

As diretivas @isset, @empty podem ser utilizadas como atalhos para suas funções PHP:

``` php
@isset($records)
    // $records is defined and is not null...
@endisset
 
@empty($records)
    // $records is "empty"...
@endempty

```

#### # Diretivas de autenticação

As diretivas **@auth** e **@guest** podem ser utilizadas para determinar rapidamente se o usuário atual é autenticado ou convidado:

```php 
@auth
    // The user is authenticated...
@endauth
 
@guest
    // The user is not authenticated...
@endguest
``` 
Caso necessário,você pode especificar a proteção da autenticação que deverá ser verificada ao usar as diretivas **@auth** e **@guest**: 

```php
@auth('admin')
    // The user is authenticated...
@endauth
 
@guest('admin')
    // The user is not authenticated...
@endguest
```

#### # Diretivas de ambiente

Pode-se verificar se a aplicação está funcionando no local de produção, para isso utiliza-se a diretiva **@production**:

``` php 
@production
    // Production specific content...
@endproduction
``` 

Ou pode-se determinar se o aplicativo está funcionando em um local específico utilizando a diretiva **@env**: 

```php
@env('staging')
    // The application is running in "staging"...
@endenv
 
@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
``` 
#### # Diretrizes da Seção

Pode-se determinar se existe uma seção de herança com conteúdo utilizando a diretiva **@hasSection**: 

``` php
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>
 
    <div class="clearfix"></div>
@endif
``` 

Utiliza-se a diretiva **sectionMissing** para determinar se uma seção não possui conteúdo: 

``` php 
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif 
```

### # Alternar declarações

As instruções switch podem ser criadas utilizando as diretivas, **@switch**, **@case** e **@break: @default** **@endswitch**:

``` php
@switch($i)
    @case(1)
        First case...
        @break
 
    @case(2)
        Second case...
        @break
 
    @default
        Default case...
@endswitch
``` 

### # Laços

Além das declarações condicionais, o Blade sugere diretivas simples para utilizar nas estruturas de loop do PHP, cada diretiva funciona de maneira idêntica às suas contrapartes PHP: 

``` php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor
 
@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach
 
@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse
 
@while (true)
    <p>I'm looping forever.</p>
@endwhile
``` 

Ao repetir em um loop **foreach**, pode-se utilizar a variável loop para obter informações sobre o loop. 

Na utilização de loops, pode-se pular a iteração atual ou finalizar o loop usando as diretivas **@continue** e **@break**

``` php 
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif
 
    <li>{{ $user->name }}</li>
 
    @if ($user->number == 5)
        @break
    @endif
@endforeach
``` 
### Pode-se inserir a condição de continuação ou corte da declaração da diretiva: 

``` php 
@foreach ($users as $user)
    @continue($user->type == 1)
 
    <li>{{ $user->name }}</li>
 
    @break($user->number == 5)
@endforeach
``` 

### # A variável de loop

No decorrer da iteração através de um loop **foreach**, uma variável **$loop** estará disponível dentro do seu loop. Esta variável oferece acesso a algumas informações, como o índice atual do loop e se é a primeira ou a última interação: 

``` php 
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif
 
    @if ($loop->last)
        This is the last iteration.
    @endif
 
    <p>This is user {{ $user->id }}</p>
@endforeach
```

O loop aninhado pode acessar a variável pai através da propriedade **parent**:

``` php
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
``` 

A variável **$loop** possui diversas propriedades úteis:

| Propriedade | Descrição|
|-|-|
|**$loop->index**| O índice da iteração de loop atual (começa em 0).|
|**$loop->iteration**| A iteração do loop atual (começa em 1).|
|**$loop->remaining**| As iterações restantes no loop.|
|**$loop->count**| O número total de itens na matriz sendo iterados.|
|**$loop->first**|	Se esta é a primeira iteração através do loop.|
|**$loop->last**| Se esta é a última iteração através do loop.|
|**$loop->even**| Se esta é uma iteração uniforme através do loop.|
|**$loop->odd**|	Se esta é uma iteração estranha através do loop.|
|**$loop->depth**|	O nível de aninhamento do loop atual.|
|**$loop->parent**| Quando em um loop aninhado, a variável de loop do pai.|

### # Classes condicionais

A diretiva  **@class** compila uma string do CSS e aceita um array de classes no qual a chave possui a classe ou classes para adicionar, ao mesmo tempo que o valor é uma expressão booleana. Caso o elemento tenha uma chave numérica será inserida nas classes renderizadas: 

``` php
@php
    $isActive = false;
    $hasError = true;
@endphp
 
<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>
 
<span class="p-4 text-gray-500 bg-red"></span>
```

### # Atributos Adicionais

Pode-se utilizar a diretiva **@checked** para sinalizar se uma determinada caixa de seleção HTML está selecionada. Esta diretiva será validada se a condição for avaliada como *-*true**:

``` php
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

A diretiva **@selected** pode ser utilizada para estabelecer se uma opção deve ser “selecionada”:

``` php
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

A diretiva **@disabled** pode ser utilizada para mostrar se um elemento deve ser desativado: 

``` php
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

A diretiva **@readonly** pode ser utilizada para indicar se um elemento deve ser somente leitura: 

``` php
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
``` 

A diretiva **@required** pode ser utilizada para indicar se um determinado elemento será obrigatório: 

```php
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

### # Incluindo subvisualizações

Os componentes **@include** blade fornece funcionalidade semelhante e também vários benefícios sobre a diretiva juntamente com a vinculação de dados e atributos.

A diretiva do **Blade  @include** permite inserir uma visualização dentro de outra visualização. Todas as variáveis disponíveis para a visualização pai serão oferecidas para a visualização incluída: 

``` php
<div>
    @include('shared.errors')
 
    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Embora que a visualização incluída herde todos os dados presentes na visualização pai, pode-se também passar uma matriz de dados adicionais que devem ser apresentados para a visualização incluída: 

``` php
@include('view.name', ['status' => 'complete'])
```


Ao tentar uma visão **@include** não existente, o Laravel mostrará um erro. Caso queira inserir uma visão que pode estar ou não presente, usa-se a diretiva **@includeIf**:

``` php
@includeIf('view.name', ['status' => 'complete'])
```

Se quiser verificar se uma expressão booleana é validada como true ou  false, pode-se usar as diretivas **@includeWhen** e **@includeUnless**: 

``` php
@includeWhen($boolean, 'view.name', ['status' => 'complete'])
 
@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

Para inserir a primeira visão de um array de visualizações, pode ser usado a diretiva **includeFirst**:

``` php
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete']
``` 

Deve-se evitar usar as constantes **__DIR__** e **__FILE__** nas visualizações do Blade, pois elas se referem ao local da visualização compilada.

#### # Renderizando visualizações para coleções

Para combinar loops e includes em uma linha usa-se a diretiva **@each**

```php
@each('view.name', $jobs, 'job')
```

O primeiro argumento **@each** da diretiva é a exibição que será renderizada para cada um dos elementos presentes na matriz ou coleção. O segundo argumento é a matriz ou coleção que se deseja inteirar, já o terceiro argumento  é o nome da variável que será atribuída à iteração atual na exibição.  Assim, por exemplo, se você estiver iterando sobre um array de **jobs** normalmente você deseja acessar cada trabalho como uma variável **job** dentro da visualização. A chave de matriz para a iteração atual estará disponível como a **key** variável na exibição.

Você também pode passar um quarto argumento para a diretiva **@each**. Este argumento determina a visão que será renderizada se o array fornecido estiver vazio.

``` php
@each('view.name', $jobs, 'job', 'view.empty')
```

> As visualizações renderizadas via **@each** não herdam as variáveis ​​da visualização pai. Se a exibição filha requer essas variáveis, você deve usar as diretivas **@foreach** e em vez disso **@include**.

### # A diretiva @once

A diretiva **@once** permite definir uma parte do template que será avaliada apenas uma vez por ciclo de renderização. Isso pode ser útil para enviar uma determinada parte do JavaScript para o cabeçalho da página usando __stacks__. Por exemplo, se você estiver renderizando um determinado componente dentro de um loop, talvez queira apenas enviar o JavaScript para o cabeçalho na primeira vez que o componente for renderizado:

``` php
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Como a diretiva  **@once** é frequentemente usada em conjunto com as diretivas **@pushou ou **@prepend**, as diretivas **@pushOnce** e **@prependOnce** estão disponíveis para sua conveniência:

``` php
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

### # PHP bruto

Em algumas situações, é útil incorporar código PHP em suas visualizações. Você pode usar a diretiva Blade **@php** para executar um bloco de PHP simples dentro do seu template:

``` php
@php
    $counter = 1;
@endphp
```

### # Comentários 

O Blade também permite definir comentários em suas visualizações. No entanto, diferentemente dos comentários HTML, os comentários do Blade não são incluídos no HTML retornado pelo seu aplicativo:

```
{{-- This comment will not be present in the rendered HTML --}} 
```

## # Componentes

Componentes e slots oferecem benefícios semelhantes a seções, layouts e inclusões; no entanto, alguns podem achar o modelo mental de componentes e slots mais fácil de entender. Existem duas abordagens para escrever componentes: componentes baseados em classes e componentes anônimos.

Para criar um componente baseado em classe, você pode usar o comando Artisan **make:component**. Para ilustrar como usar componentes, vamos criar um **Alert** componente simples. O comando make:component colocará o componente no diretório: **app/View/Componentes**

```
php artisan make:component Alert
```

O comando make:component também criará um modelo de visualização para o componente. A visualização será colocada no iretório **resources/views/components**. Ao escrever componentes para seu próprio aplicativo, os componentes são descobertos automaticamente no diretório app/View/Components e no diretório **resources/views/components**, portanto, normalmente, nenhum registro de componente adicional é necessário.

Você também pode criar componentes dentro de subdiretórios:

```php
php artisan make:component Forms/Input
```

O comando acima criará um componente **Input**  no diretório **app/View/Components/Forms** e a visualização será colocada no diretório **resources/views/components/forms**.

Se você deseja criar um componente anônimo (um componente com apenas um modelo Blade e sem classe), você pode usar o sinalizador **--view** ao invocar o comando **make:component:

```php
php artisan make:component forms.input --view
```

O comando acima criará um arquivo Blade no **resources/views/components/forms/input.blade.php** que pode ser renderizado como um componente via **<x-forms.input />**.


#### # Registrando manualmente os componentes do pacote

Ao escrever componentes para seu próprio aplicativo, os componentes são descobertos automaticamente no diretório **app/View/Components** e no diretório **resources/views/components**.

No entanto, se você estiver construindo um pacote que utiliza componentes Blade, você precisará registrar manualmente sua classe de componente e seu alias de tag HTML. Normalmente, você deve registrar seus componentes no  método boot do provedor de serviços do seu pacote:

``` php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap your package's services.
 */
public function boot()
{
    Blade::component('package-alert', Alert::class);

``` 

Depois que seu componente for registrado, ele poderá ser renderizado usando seu alias de tag:

```php 
<x-package-alert/>

``` 

Como alternativa, você pode usar o método **componentNamespace** para carregar automaticamente as classes de componentes por convenção. Por exemplo, um pacote **Nightshade** pode ter **Calendare** e **ColorPicker** que são componentes que residem no namespace: **Package\Views\Components**

``` php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Isso permitirá o uso dos componentes do pacote pelo namespace do fornecedor usando a sintaxe: package-name::

``` php
<x-nightshade::calendar />
<x-nightshade::color-picker />

```

O Blade detectará automaticamente a classe que está vinculada a este componente colocando em pascal o nome do componente. Os subdiretórios também são suportados usando a notação "ponto".

### # Componentes de renderização

Para exibir um componente, você pode usar uma tag de componente Blade em um de seus modelos Blade. As tags do componente blade começam com a string **x-** seguida pelo nome do caso kebab da classe do componente:

```php
<x-alert/>
 
<x-user-profile/>
```

Se a classe do componente estiver aninhada mais profundamente no diretório **app/View/Components**, você poderá usar o caractere **.** para indicar o aninhamento do diretório. Por exemplo, se assumirmos que um componente está localizado em **app/View/Components/Inputs/Button.php**, podemos renderizá-lo assim:

``` php
<x-inputs.button/>
```

### # Passando dados para componentes

Você pode passar dados para componentes Blade usando atributos HTML. Valores primitivos codificados podem ser passados ​​para o componente usando strings de atributo HTML simples. Expressões e variáveis ​​PHP devem ser passadas para o componente por meio de atributos que usam o caractere como prefixo: **:**

```php
<x-alert type="error" :message="$message"/>
``` 

Você deve definir todos os atributos de dados do componente em seu construtor de classe. Todas as propriedades públicas em um componente serão disponibilizadas automaticamente para a visualização do componente. Não é necessário passar os dados para a view do método do componente: **render**

``` php
<?php
 
namespace App\View\Components;
 
use Illuminate\View\Component;
 
class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;
 
    /**
     * The alert message.
     *
     * @var string
     */
    public $message;
 
    /**
     * Create the component instance.
     *
     * @param  string  $type
     * @param  string  $message
     * @return void
     */
    public function __construct($type, $message)
    {
        $this->type = $type;
        $this->message = $message;
    }
 
    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return view('components.alert');
    }
}
```

Quando seu componente é renderizado, você pode exibir o conteúdo das variáveis ​​públicas do seu componente ecoando as variáveis ​​pelo nome:

```php
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```
#### # Revestimento

Os argumentos do construtor de componentes devem ser especificados usando **camelCase**, enquanto **kebab-case** devem ser usados ​​ao fazer referência aos nomes dos argumentos em seus atributos HTML. Por exemplo, dado o seguinte construtor de componentes:

``` php
/**
 * Create the component instance.
 *
 * @param  string  $alertType
 * @return void
 */
public function __construct($alertType)
{
    $this->alertType = $alertType;
}
``` 

O argumento **$alertType** pode ser fornecido ao componente assim:

``` php
<x-alert alert-type="danger" />

``` 

#### # Sintaxe de atributo simples

Ao passar atributos para componentes, você deve usar uma sintaxe curta de atributo. Isto é frequentemente conveniente dado que os nomes de atributo frequentemente correspondem aos nomes de variáveis:

```php
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />
 
{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

#### # Escape da renderização de atributos

Como alguns frameworks JavaScript, como Alpine.js, também usam atributos prefixados com dois pontos, você pode usar um  prefixo com dois pontos **::** para informar ao Blade que o atributo não é uma expressão PHP. Por exemplo, dado o seguinte componente:

```php
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

O seguinte HTML será renderizado pelo Blade:

```php
<button :class="{ danger: isDeleting }">
    Submit
</button>
``` 

#### # Métodos de componentes

Além das variáveis ​​públicas estarem disponíveis para seu modelo de componente, quaisquer métodos públicos no componente podem ser invocados. Por exemplo, imagine um componente que tenha um dmétodo: **isSelecte**

```php
/**
 * Determine if the given option is the currently selected option.
 *
 * @param  string  $option
 * @return bool
 */
public function isSelected($option)
{
    return $option === $this->selected;
} 
```

Você pode executar este método a partir do seu modelo de componente invocando a variável que corresponde ao nome do método:

``` php
<option {{ $isSelected($value) ? 'selected="selected"' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

#### # Acessando atributos e slots em classes de componentes

Os componentes blade também permitem que você acesse o nome do componente, atributos e slot dentro do método de renderização da classe. No entanto, para acessar esses dados, você deve retornar um encerramento do método **render** do seu componente. A closure receberá um array **$data** como seu único argumento. Esta matriz conterá vários elementos que fornecem informações sobre o componente:

``` php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];
 
        return '<div>Components content</div>';
    };
}
```

O **componentName** é igual ao nome usado na tag HTML após o prefixo **x-**. Assim será **<x-alert />** O elemento que conterá todos os atributos que estavam presentes na tag HTML. O elemento é uma instância com o conteúdo do slot do componente.**componentNamealertattributesslotIlluminate\Support\HtmlString**

O encerramento deve retornar uma string. Se a string retornada corresponder a uma visualização existente, essa visualização será renderizada; caso contrário, a string retornada será avaliada como uma visualização Blade inline.

#### # Dependências Adicionais

Se seu componente requer dependências do container de serviço do Laravel , você pode listá-las antes de qualquer atributo de dados do componente e elas serão automaticamente injetadas pelo container:

``` php
use App\Services\AlertCreator;
 
/**
 * Create the component instance.
 *
 * @param  \App\Services\AlertCreator  $creator
 * @param  string  $type
 * @param  string  $message
 * @return void
 */
public function __construct(AlertCreator $creator, $type, $message)
{
    $this->creator = $creator;
    $this->type = $type;
    $this->message = $message;
}
```

#### # Ocultando Atributos/Métodos

Se você quiser evitar que alguns métodos ou propriedades públicas sejam expostos como variáveis ​​para seu modelo de componente, você pode adicioná-los a uma propriedade **$except** de matriz em seu componente:

```php
<?php
 
namespace App\View\Components;
 
use Illuminate\View\Component;
 
class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;
 
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];
}
``` 

### # Atributos do Componente

Já examinamos como passar atributos de dados para um componente; no entanto, às vezes você pode precisar especificar atributos HTML adicionais, como **class**, que não fazem parte dos dados necessários para que um componente funcione. Normalmente, você deseja passar esses atributos adicionais para o elemento raiz do modelo de componente. Por exemplo, imagine que queremos renderizar um componente **alert** assim:

``` php
<x-alert type="error" :message="$message" class="mt-4"/>
```

Todos os atributos que não fazem parte do construtor do componente serão automaticamente adicionados ao "attribute bag" do componente. Este pacote de atributos é disponibilizado automaticamente para o componente por meio da variável $attributes. Todos os atributos podem ser renderizados dentro do componente ecoando esta variável:

```php

<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

#### # Atributos padrão/mesclados

Às vezes, você pode precisar especificar valores padrão para atributos ou mesclar valores adicionais em alguns dos atributos do componente. Para fazer isso, você pode usar o método do atributo bag **merge**. Este método é particularmente útil para definir um conjunto de classes CSS padrão que sempre devem ser aplicadas a um componente:

```php
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Se assumirmos que este componente é utilizado assim:

``` php
<x-alert type="error" :message="$message" class="mb-4"/>
```

O HTML final e o renderizado do componente terá a seguinte aparência:

```php
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

#### # Mesclar classes condicionalmente

Às vezes você pode querer mesclar classes se uma determinada condição for **true**. Você pode fazer isso através do método **class**, que aceita um array de classes onde a chave do array contém a classe ou classes que você deseja adicionar, enquanto o valor é uma expressão booleana. Se o elemento array tiver uma chave numérica, ele sempre será incluído na lista de classes renderizadas:

```php
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

Se você precisar mesclar outros atributos em seu componente, você pode encadear o método **merge** no método **class**:

``` php
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
``` 

Se você precisar compilar condicionalmente classes em outros elementos HTML que não devem receber atributos mesclados, você pode usar a diretiva.

#### # Mesclagem de atributos não-classe

Ao mesclar atributos que não são atributos **class**, os valores fornecidos ao método merge serão considerados os valores "padrão" do atributo. No entanto, ao contrário do  **class**, esses atributos não serão mesclados com valores de atributo injetados. Em vez disso, eles serão substituídos. Por exemplo, a implementação button de um componente pode ter a seguinte aparência:

``` php
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

Para renderizar o componente de botão com um custom **type**, ele pode ser especificado ao consumir o componente. Se nenhum tipo for especificado, o  button será usado:

``` php
<x-button type="submit">
    Submit
</x-button>
```

O HTML renderizado do componente button neste exemplo seria:

``` php
<button type="submit">
    Submit
</button>
``` 

Se você quiser que um atributo diferente de **class** ter seu valor padrão e valores injetados juntos, você pode usar o métodop repends. Neste exemplo, o atributo **data-controller** sempre começará com **profile-controllere** quaisquer valores injetados adicionais  serão colocados após este valor padrão:

``` php
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
``` 

#### # Recuperando e Filtrando Atributos

Você pode filtrar atributos usando o  **filter**. Este método aceita um encerramento que deve retornar **true** se você deseja manter o atributo no pacote de atributos:

```php
{{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}
```

Por conveniência, você pode usar o método whereStartsWith para recuperar todos os atributos cujas chaves começam com uma determinada string:

``` php
{{ $attributes->whereStartsWith('wire:model') }}
``` 

Por outro lado, o método **whereDoesntStartWith** pode ser usado para excluir todos os atributos cujas chaves começam com uma determinada string:

``` php
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Usando o método **first**, você pode renderizar o primeiro atributo em um determinado pacote de atributos:

```php 
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Se você quiser verificar se um atributo está presente no componente, você pode usar o método **has**. Este método aceita o nome do atributo como seu único argumento e retorna um booleano indicando se o atributo está presente ou não:

``` php
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
``` 

Você pode recuperar o valor de um atributo específico usando o método **get**:

```php
{{ $attributes->get('class') }}
```

### # Palavras-chave reservadas

Por padrão, algumas palavras-chave são reservadas para uso interno do Blade para renderizar componentes. As seguintes palavras-chave não podem ser definidas como propriedades públicas ou nomes de métodos em seus componentes:

* data
* render
* resolveView
* shouldRender
* view
* withAttributes
* withName


### # Slots

Muitas vezes, você precisará passar conteúdo adicional para seu componente por meio de "slots". Os slots de componentes são renderizados ecoando a  **$slot** . Para explorar esse conceito, vamos imaginar que um componente **alert** tenha a seguinte marcação:

```php
<!-- /resources/views/components/alert.blade.php -->
 
<div class="alert alert-danger">
    {{ $slot }}
</div>
``` 

Podemos passar conteúdo para o **slot** injetando conteúdo no componente:

```php
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
``` 

Às vezes, um componente pode precisar renderizar vários slots diferentes em locais diferentes dentro do componente. Vamos modificar nosso componente de alerta para permitir a injeção de um slot "título":

```php
<!-- /resources/views/components/alert.blade.php -->
 
<span class="alert-title">{{ $title }}</span>
 
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Você pode definir o conteúdo do slot nomeado usando a tag **x-slot**. Qualquer conteúdo que não esteja dentro de uma tag explícita será **x-slot** passado para o componente na variável **$slot**:

```php
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>
 
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
``` 

#### # Slots com escopo

Se você usou uma estrutura JavaScript como Vue, pode estar familiarizado com "slots com escopo", que permitem acessar dados ou métodos do componente em seu slot. Você pode obter um comportamento semelhante no Laravel definindo métodos ou propriedades públicas em seu componente e acessando o componente em seu slot por meio da variável $component. Neste exemplo, vamos supor que o componente **x-alert** tenha um método **formatAlert** público definido em sua classe de componente:

```php
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>
 
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
``` 

#### # Atributos de slot

Assim como os componentes Blade, você pode atribuir atributos adicionais a slots, como nomes de classes CSS:

``` php
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>
 
    Content
 
    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
``` 

Para interagir com os atributos do slot, você pode acessar a attributespropriedade da variável do slot. Para mais informações sobre como interagir com atributos, consulte a documentação sobre atributos de componentes:

``` php
@props([
    'heading',
    'footer',
])
 
<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>
 
    {{ $slot }}
 
    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
``` 

### # Visualizações de componentes embutidos

Para componentes muito pequenos, pode parecer complicado gerenciar tanto a classe do componente quanto o modelo de visualização do componente. Por esse motivo, você pode retornar a marcação do componente diretamente do método **render**:

``` php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

#### # Gerando Componentes de Visualização Inline

Para criar um componente que renderiza uma visualização em linha, você pode usar a opção  inline ao executar o comando: make:component.

``` php
php artisan make:component Alert --inline
``` 

### # Componentes dinâmicos

Às vezes, você pode precisar renderizar um componente, mas não sabe qual componente deve ser renderizado até o tempo de execução. Nesta situação, você pode usar o componente **dynamic-component** interno do Laravel para renderizar o componente com base em um valor ou variável de tempo de execução:

``` php
<x-dynamic-component :component="$componentName" class="mt-4" />
``` 

### # Registrando manualmente os componentes

Ao escrever componentes para seu próprio aplicativo, os componentes são descobertos automaticamente no diretório **app/View/Components** e no diretório **resources/views/components**.

No entanto, se você estiver construindo um pacote que utiliza componentes Blade ou colocando componentes em diretórios não convencionais, você precisará registrar manualmente sua classe de componente e seu alias de tag HTML para que o Laravel saiba onde encontrar o componente. Normalmente, você deve registrar seus componentes no método **boot** do provedor de serviços do seu pacote:

``` php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;
 
/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Depois que seu componente for registrado, ele poderá ser renderizado usando seu alias de tag:

``` php
<x-package-alert/>
```

##### # Componentes do pacote de carregamento automático

Como alternativa, você pode usar o método **componentNamespace** para carregar automaticamente as classes de componentes por convenção. Por exemplo, um pacote **Nightshade** pode ter  **Calendar** e  **ColorPickercomponentes** que residem no namespace: **Package\Views\Component** 

``` php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Isso permitirá o uso dos componentes do pacote pelo namespace do fornecedor usando a sintaxe: **package-name::**

``` php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

O Blade detectará automaticamente a classe que está vinculada a este componente colocando em pascal o nome do componente. Os subdiretórios também são suportados usando a notação "ponto".

## # Componentes anônimos

Semelhante aos componentes inline, os componentes anônimos fornecem um mecanismo para gerenciar um componente por meio de um único arquivo. No entanto, componentes anônimos utilizam um único arquivo de visualização e não possuem classe associada. Para definir um componente anônimo, você só precisa colocar um modelo Blade em seu diretório **resources/views/components**. Por exemplo, supondo que você tenha definido um componente em **resources/views/components/alert.blade.php**, você pode simplesmente renderizá-lo assim:

```php
<x-alert/>
```

Você pode usar o caractere *.* para indicar se um componente está aninhado mais profundamente dentro do diretório components. Por exemplo, supondo que o componente esteja definido em **resources/views/components/inputs/button.blade.php**, você pode renderizá-lo assim:

``` php
<x-inputs.button/>
``` 

#### # Componentes de índice anônimo

Às vezes, quando um componente é composto de muitos modelos Blade, você pode querer agrupar os modelos do componente em um único diretório. Por exemplo, imagine um componente "acordeão" com a seguinte estrutura de diretórios:

``` php
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
``` 

Esta estrutura de diretórios permite renderizar o componente de acordeão e seu item da seguinte forma:

```php
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

No entanto, para renderizar o componente de acordeão via **x-accordion**, fomos forçados a colocar o modelo de componente de acordeão "índice" no diretório **resources/views/components** em vez de aninhar dentro do  **accordion**  com os outros modelos relacionados ao acordeão.

Felizmente, o Blade permite que você coloque um arquivo **index.blade.php** no diretório de modelos de um componente. Quando **index.blade.php** existe um modelo para o componente, ele será renderizado como o nó "raiz" do componente. Assim, podemos continuar usando a mesma sintaxe do Blade dada no exemplo acima; no entanto, ajustaremos nossa estrutura de diretórios assim:

``` php
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

### # Propriedades/atributos de dados

Como os componentes anônimos não têm nenhuma classe associada, você pode se perguntar como diferenciar quais dados devem ser passados ​​para o componente como variáveis ​​e quais atributos devem ser colocados no pacote de atributos do componente .

Você pode especificar quais atributos devem ser considerados variáveis ​​de dados usando a diretiva **@props** na parte superior do template Blade do seu componente. Todos os outros atributos do componente estarão disponíveis por meio da sacola de atributos do componente. Se você deseja dar a uma variável de dados um valor padrão, você pode especificar o nome da variável como a chave do array e o valor padrão como o valor do array:

```php
<!-- /resources/views/components/alert.blade.php -->
 
@props(['type' => 'info', 'message'])
 
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Dada a definição do componente acima, podemos renderizar o componente assim:

```php
<x-alert type="error" :message="$message" class="mb-4"/>
```

### # Acessando dados dos pais

Às vezes você pode querer acessar dados de um componente pai dentro de um componente filho. Nesses casos, você pode usar a diretiva **@aware**. Por exemplo, imagine que estamos construindo um componente de menu complexo que consiste em um pai **<x-menu>** e um filho **<x-menu.item>**:

```php
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

O componente **<x-menu>** pode ter uma implementação como a seguinte:

```php
<!-- /resources/views/components/menu/index.blade.php -->
 
@props(['color' => 'gray'])
 
<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

Como o prop **color** foi passado apenas para o pai **( <x-menu>)**, ele não estará disponível dentro do **<x-menu.item>**. No entanto, se usarmos a diretiva @aware, podemos disponibilizá-la dentro **<x-menu**.item> e também:

``` php
<!-- /resources/views/components/menu/item.blade.php -->
 
@aware(['color' => 'gray'])
 
<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
``` 

A diretiva **@aware** não pode acessar dados pai que não sejam explicitamente passados ​​para o componente pai por meio de atributos HTML. Os valores padrão **@props** que não são explicitamente passados ​​para o componente pai não podem ser acessados ​​pela diretiva @aware.

### # Namespaces de componentes anônimos

Conforme discutido anteriormente, os componentes anônimos geralmente são definidos colocando um modelo Blade em seu diretório **resources/views/components**. No entanto, você pode ocasionalmente querer registrar outros caminhos de componentes anônimos com o Laravel além do caminho padrão.

Por exemplo, ao criar um aplicativo de reserva de férias, você pode colocar componentes anônimos relacionados à reserva de voo em um diretório **resources/views/flights/bookings/components**. Para informar ao Laravel sobre a localização desse componente anônimo, você pode usar o método **anonymousComponentNamespace** fornecido pela fachada Blade.

O método **anonymousComponentNamespace**  aceita o "caminho" para o local do componente anônimo como seu primeiro argumento e o "namespace" em que os componentes devem ser colocados como seu segundo argumento. Como você verá no exemplo abaixo, o "namespace" será prefixado ao nome do componente quando o componente for renderizado. Normalmente, esse método deve ser chamado a partir do método **boot**  de um dos provedores de serviço do seu aplicativo :

```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::anonymousComponentNamespace('flights.bookings.components', 'flights');
}
```

Dado o exemplo acima, você pode renderizar um panelcomponente que existe dentro do diretório de componentes recém-registrado assim:

``` php
<x-flights::panel :flight="$flight" />
```

## # Layouts de construção

### # Layouts usando componentes

A maioria dos aplicativos da Web mantém o mesmo layout geral em várias páginas. Seria incrivelmente complicado e difícil manter nosso aplicativo se tivéssemos que repetir todo o layout HTML em todas as visualizações que criamos. Felizmente, é conveniente definir esse layout como um único componente Blade e usá-lo em todo o nosso aplicativo.

#### # Definindo o componente de layout

Por exemplo, imagine que estamos construindo um aplicativo de lista de tarefas. Podemos definir um componente **layout** que se pareça com o seguinte:

```php
<!-- resources/views/components/layout.blade.php -->
 
<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

#### # Aplicando o componente de layout

Uma vez que o componente **layout** tenha sido definido, podemos criar uma visão Blade que utiliza o componente. Neste exemplo, definiremos uma visualização simples que exibe nossa lista de tarefas:

```php
<!-- resources/views/tasks.blade.php -->
 
<x-layout>
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Lembre-se, o conteúdo que é injetado em um componente será fornecido à variável **$slot** padrão em nosso componente **layout**. Como você deve ter notado, o nosso **layout** também respeita um slot **$title**  se for fornecido; caso contrário, um título padrão é mostrado. Podemos injetar um título personalizado de nossa visualização de lista de tarefas usando a sintaxe de slot padrão discutida na documentação do componente :

```php
<!-- resources/views/tasks.blade.php -->
 
<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>
 
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Agora que definimos nossas visualizações de layout e lista de tarefas, precisamos apenas retornar a visualização **task** de uma rota:

```php
use App\Models\Task;
 
Route::get('/tasks', function () {
    return view('tasks', ['tasks' => Task::all()]);
});
```

### # Layouts usando herança de modelo

#### # Definindo um layout

Os layouts também podem ser criados via "herança de modelo". Essa era a principal maneira de construir aplicativos antes da introdução dos componentes .

Para começar, vamos dar uma olhada em um exemplo simples. Primeiro, examinaremos um layout de página. Como a maioria dos aplicativos da Web mantém o mesmo layout geral em várias páginas, é conveniente definir esse layout como uma única visualização do Blade:

```php
<!-- resources/views/layouts/app.blade.php -->
 
<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show
 
        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Como você pode ver, este arquivo contém marcação HTML típica. No entanto, tome nota das diretivas **@sectione**. @yield A **@sectiondiretiva**, como o nome indica, define uma seção de conteúdo, enquanto a diretiva  **@yield** é usada para exibir o conteúdo de uma determinada seção.

Agora que definimos um layout para nosso aplicativo, vamos definir uma página filha que herda o layout.

#### # Estendendo um layout

Ao definir uma visualização filha, use a diretiva **@extends** Blade para especificar qual layout a visualização filha deve "herdar". As visualizações que estendem um layout Blade podem injetar conteúdo nas seções do layout usando diretivas **@section**. Lembre-se, como visto no exemplo acima, o conteúdo dessas seções será exibido no layout usando **@yield**:

``` php
<!-- resources/views/child.blade.php -->
 
@extends('layouts.app')
 
@section('title', 'Page Title')
 
@section('sidebar')
    @parent
 
    <p>This is appended to the master sidebar.</p>
@endsection
 
@section('content')
    <p>This is my body content.</p>
@endsection
```
Neste exemplo, a seção **sidebar** está utilizando a diretiva **@parent** para anexar (em vez de sobrescrever) o conteúdo à barra lateral do layout. A diretiva **@parent** será substituída pelo conteúdo do layout quando a visualização for renderizada.

Ao contrário do exemplo anterior, esta sidebarseção termina com **@endsection** em vez de **@show**. A diretiva **@endsection** definirá apenas uma seção enquanto finirá **@showd** e produzirá imediatamente a seção.


A diretiva **@yield** também aceita um valor padrão como seu segundo parâmetro. Este valor será renderizado se a seção que está sendo gerada for indefinida:

```php
@yield('content', 'Default content')
```

## # Formulários

### # Campo CSRF

Sempre que você definir um formulário HTML em seu aplicativo, deverá incluir um campo de token CSRF oculto no formulário para que o middleware de proteção CSRF possa validar a solicitação. Você pode usar a @csrfdiretiva Blade para gerar o campo de token:

```php
<form method="POST" action="/profile">
    @csrf
 
    ...
</form>
```
### # Campo do método

Como os formulários HTML não podem fazer solicitações **PUT**, **PATCH**, ou solicitações **DELETE**, você precisará adicionar um campo **_method** oculto para falsificar esses verbos HTTP. A diretiva Blade **@method** pode criar este campo para você:

```php
<form action="/foo/bar" method="POST">
    @method('PUT')
 
    ...
</form>
```

### # Erros de validação

A diretiva **@error** pode ser usada para verificar rapidamente se existem mensagens de erro de validação para um determinado atributo. Dentro de uma @errordiretiva, você pode ecoar a variável **$message** para exibir a mensagem de erro:

```php
<!-- /resources/views/post/create.blade.php -->
 
<label for="title">Post Title</label>
 
<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">
 
@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Como a diretiva **@error**  compila para uma instrução "if", você pode usar a diretiva **@else**  para renderizar conteúdo quando não houver um erro para um atributo:

```php
<!-- /resources/views/auth.blade.php -->
 
<label for="email">Email address</label>
 
<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```

Você pode passar o nome de um pacote de erro específico como o segundo parâmetro para a diretiva **@error** para recuperar mensagens de erro de validação em páginas contendo vários formulários:

```php
<!-- /resources/views/auth.blade.php -->
 
<label for="email">Email address</label>
 
<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">
 
@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

# # Pilhas
O Blade permite que você envie para pilhas nomeadas que podem ser renderizadas em outro lugar em outra visualização ou layout. Isso pode ser particularmente útil para especificar quaisquer bibliotecas JavaScript exigidas por suas visualizações filhas:

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

### Se você quiser **@push** contentar se uma determinada expressão booleana for avaliada como **true**, você pode usar a diretiva **@pushIf**:

``` php
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

Você pode fazer push para uma pilha quantas vezes forem necessárias. Para renderizar o conteúdo completo da pilha, passe o nome da pilha para a diretiva **@stack**:

``` php
<head>
    <!-- Head Contents -->
 
    @stack('scripts')
</head>
```

Se você quiser colocar o conteúdo no início de uma pilha, você deve usar a diretiva **@prepend**:

``` php 
@push('scripts')
    This will be second...
@endpush
 
// Later...
 
@prepend('scripts')
    This will be first...
@endprepend
```

# # Injeção de serviço

A diretiva pode ser usada para recuperar um serviço do contêiner de serviço **@inject** Laravel. O primeiro argumento passado é o nome da variável na qual o serviço será colocado, enquanto o segundo argumento é o nome da classe ou interface do serviço que você deseja resolver **@inject**:

`` php 
@inject('metrics', 'App\Services\MetricsService')
 
<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

# # Renderizando Modelos Blade Inline

### Às vezes, pode ser necessário transformar uma string de modelo Blade bruta em HTML válido. Você pode fazer isso usando o método **render** fornecido pela **Blade** fachada. O método **render** aceita a string do modelo Blade e uma matriz opcional de dados para fornecer ao modelo:

 ``` php
use Illuminate\Support\Facades\Blade;
 
return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

### O Laravel renderiza templates Blade inline gravando-os no diretório **storage/framework/views**. Se você quiser que o Laravel remova esses arquivos temporários após renderizar o template Blade, você pode fornecer oargumento **deleteCachedView** para o método:

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);

```

# # Extensão Blade

Blade permite que você defina suas próprias diretivas personalizadas usando o emétodo **directiv**. Quando o compilador Blade encontrar a diretiva customizada, ele chamará o retorno de chamada fornecido com a expressão que a diretiva contém.

O exemplo a seguir cria uma diretiva **@datetime($var)**  que formata um determinado **$var**, que deve ser uma instância de **DateTime**:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;
 
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
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}

```

 Como você pode ver, vamos encadear o método **format**  em qualquer expressão que seja passada na diretiva. Então, neste exemplo, o PHP final gerado por esta diretiva será:

``` php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

Após atualizar a lógica de uma diretiva Blade, você precisará excluir todas as visualizações Blade armazenadas em cache. As visualizações do Blade em cache podem ser removidas usando o comando Artisan **view:clear**.


## # Manipuladores de echo personalizados

Se você tentar "ecoar" um objeto usando Blade, o método **__toString** do objeto será invocado. O método **__toString** é um dos "métodos mágicos" embutidos do PHP. No entanto, às vezes você pode não ter controle sobre o método **__toString** de uma determinada classe, como quando a classe com a qual você está interagindo pertence a uma biblioteca de terceiros.

Nesses casos, o Blade permite que você registre um manipulador de eco personalizado para esse tipo específico de objeto. Para fazer isso, você deve invocar o método **stringable** do Blade. O método **stringable** aceita um encerramento. Esse encerramento deve indicar o tipo de objeto que é responsável pela renderização. Normalmente, o método **stringable** deve ser invocado dentro do bootmétodo da classe do seu aplicativo **AppServiceProvider**:

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;
 
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
``` 

Depois que seu manipulador de eco personalizado for definido, você pode simplesmente ecoar o objeto em seu modelo Blade:

```php
Cost: {{ $money }}
```

## # Declarações If personalizadas

A programação de uma diretiva personalizada às vezes é mais complexa do que o necessário ao definir instruções condicionais simples e personalizadas. Por esse motivo, o Blade fornece um método **Blade::if** que permite definir rapidamente diretivas condicionais personalizadas usando closures. Por exemplo, vamos definir uma condicional personalizada que verifica o "disco" padrão configurado para o aplicativo. Podemos fazer isso no método **boot** do nosso **AppServiceProvider**:


``` php
use Illuminate\Support\Facades\Blade;
 
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('disk', function ($value) {
        return config('filesystems.default') === $value;
    });
}

```

Depois que a condicional personalizada for definida, você poderá usá-la em seus modelos:

``` php
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk
 
@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```

