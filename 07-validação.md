# # Validação

    Introdução
    Primeiros passos na validação
        Definindo Rotas
        Criando um Controladorr
        Escrevendo a logica de Validação
        Exibindo os erros de Validação
        Repopulando Formulários
        Uma nota sobre campos Opcionais
        Formato de resposta de erro de validação
    Validação de solicitação de formulário
        Criando solicitação de formulário
        Autorizando solicitação de Formulários
        Customizando mensagens de erro
        Preparando dados de entrada para validação
    Criando validadores manualmente
        Redirecionamento Automático
        Sacos de erro nomeados
        Customizando mensagens de erro
        Após validação
    Trabalhando com a validação dos dados entrados
    Trabalhando com mensagens de Erro
        Especificando mensagens personalizadas em arquivos de idioma
        Especificando atributos em arquivos de idioma
        Especificando valores em arquivos de idioma
    Regras de validação disponíveis
    Adicionando Regras Condicionalmente
    Validando Matrizes
        Validando a entrada de matriz aninhada
        Índices e posições de mensagens de erro
    Validando Arquivos
    Validando Senhas
    Customizando regras de Validação
        Usando regras de objetos
        Usando clausuras
        Regras implícitas
        
        
## # Introdução

O Laravel fornece diversas abordagens diferentes para validar os dados de entrada de seu aplicativo. É o mais comum usar método de validação disponível em todas as solicitações HTTP recebidas. No entanto, falaremos também de outras abordagens para realizar a validação
O Laravel inclui uma ampla variedade de regras de validação convenientes que você pode aplicar aos dados, até mesmo fornecendo a capacidade de validar se os valores são exclusivos em uma determinada tabela de banco de dados. Cobriremos cada uma dessas regras de validação em detalhes para que você esteja familiarizado com todos os recursos de validação do Laravel.


## # Início rápido de validação

Para aprender sobre os poderosos recursos de validação do Laravel, vejamos um exemplo completo de validação de um formulário e exibição das mensagens de erro de volta para o usuário. Ao ler esta visão geral de alto nível, você poderá obter uma boa compreensão geral de como validar dados de solicitações recebidas usando o Laravel:


### # Definindo as Rotas

Primeiro, vamos supor que temos as seguintes rotas definidas em nosso arquivo routes/web.php:

```php
use App\Http\Controllers\PostController;
 
Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

A rota GET exibirá um formulário para o usuário criar uma nova postagem no blog, enquanto a rota POST armazenará a nova postagem no banco de dados.

### # Criando o controlador

Em seguida, vamos dar uma olhada em um controlador simples que lida com solicitações de entrada para essas rotas. Vamos deixar o método store vazio por enquanto:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
 
class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return \Illuminate\View\View
     */
    public function create()
    {
        return view('post.create');
    }
 
    /**
     * Store a new blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### # Escrevendo a lógica de validação    

Agora estamos prontos para preencher nosso método store com a lógica para validar a nova postagem do blog. Para isso, usaremos o método validate fornecido pelo objeto Illuminate\Http\Request. Se as regras de validação forem aprovadas, seu código continuará executando normalmente; no entanto, se a validação falhar, uma exceção Illuminate\Validation\ValidationException será lançada e a resposta de erro apropriada será automaticamente enviada de volta ao usuário.

Se a validação falhar durante uma solicitação HTTP tradicional, uma resposta de redirecionamento para a URL anterior será gerada. Se a solicitação recebida for uma solicitação XHR, uma resposta JSON contendo as mensagens de erro de validação será retornada.

Para entender melhor o método validate, vamos voltar para o método store:pty por enquanto:

```php
/**
 * Armazene uma nova postagem no blog.
 *
 * @param \Illuminate\Http\Request $request
 * @return \Illuminate\Http\Response
 */
public function store (Request $request)
{
    $validated = $required->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);
 
    // A postagem do blog é válida...
}
```

Como você pode ver, as regras de validação são passadas para o método validate. Não se preocupe - todas as regras de validação disponíveis estão documentadas. Novamente, se a validação falhar, a resposta adequada será gerada automaticamente. Se a validação for aprovada, nosso controlador continuará executando normalmente.

Alternativamente, as regras de validação podem ser especificadas como matrizes de regras em vez de um único | cadeia delimitada:

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

Além disso, você pode usar o método validateWithBag para validar uma solicitação e armazenar todas as mensagens de erro em um pacote de erro nomeado:

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### # Parando na primeira falha de validação

Às vezes, você pode querer parar de executar as regras de validação em um atributo após a primeira falha de validação. Para isso, atribua a regra de fiança ao atributo:

```php
$request->validate([
    'title' => 'fiança|obrigatório|único:postas|max:255',
    'body' => 'required',
]);
```

Neste exemplo, se a regra exclusiva no atributo title falhar, a regra max não será verificada. As regras serão validadas na ordem em que forem atribuídas.

#### # Uma nota sobre atributos aninhados

Se a solicitação HTTP recebida contiver dados de campo "aninhados", você poderá especificar esses campos em suas regras de validação usando a sintaxe "ponto":

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

Por outro lado, se o nome do seu campo contiver um ponto literal, você pode impedir explicitamente que isso seja interpretado como sintaxe de "ponto" escapando o ponto com uma barra invertida:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### # Exibindo os erros de validação

Então, e se os campos de solicitação de entrada não passarem pelas regras de validação fornecidas? Como mencionado anteriormente, o Laravel redirecionará automaticamente o usuário de volta para sua localização anterior. Além disso, todos os erros de validação e entrada de solicitação serão automaticamente exibidos na sessão.

Uma variável $errors é compartilhada com todas as visualizações do seu aplicativo pelo middleware Illuminate\View\Middleware\ShareErrorsFromSession, que é fornecido pelo grupo de middleware da web. Quando esse middleware é aplicado, uma variável $errors sempre estará disponível em suas visualizações, permitindo que você assuma convenientemente que a variável $errors está sempre definida e pode ser usada com segurança. A variável $errors será uma instância de Illuminate\Support\MessageBag. Para obter mais informações sobre como trabalhar com este objeto, confira sua documentação.

Assim, em nosso exemplo, o usuário será redirecionado para o método create do nosso controller quando a validação falhar, permitindo-nos exibir as mensagens de erro na view:

```php
<!-- /resources/views/post/create.blade.php -->
 
<h1>Create Post</h1>
 
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@fim se
<!-- Criar formulário de postagem -->
```

#### # Personalizando as mensagens de erro

Cada uma das regras de validação integradas do Laravel tem uma mensagem de erro localizada no arquivo lang/en/validation.php da sua aplicação. Nesse arquivo, você encontrará uma entrada de tradução para cada regra de validação. Você é livre para alterar ou modificar essas mensagens com base nas necessidades do seu aplicativo.

Além disso, você pode copiar este arquivo para outro diretório de idioma de tradução para traduzir as mensagens para o idioma do seu aplicativo. Para saber mais sobre a localização do Laravel, confira a documentação completa de localização.

#### # Solicitações e validação de XHR

Neste exemplo, usamos um formulário tradicional para enviar dados para o aplicativo. No entanto, muitos aplicativos recebem solicitações XHR de um front-end com JavaScript. Ao usar o método de validação durante uma solicitação XHR, o Laravel não gerará uma resposta de redirecionamento. Em vez disso, o Laravel gera uma resposta JSON contendo todos os erros de validação. Essa resposta JSON será enviada com um código de status HTTP 422.

#### # A diretiva @error

Você pode usar a diretiva @error Blade para determinar rapidamente se existem mensagens de erro de validação para um determinado atributo. Dentro de uma diretiva @error, você pode ecoar a variável $message para exibir a mensagem de erro:

```php
<!-- /resources/views/post/create.blade.php -->
 
<label for="title">Título do Post</label>
 
<input id="title"
    tipo="text"
    nome="title"
    class="@error('title') is-invalid @enderror">
 
@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Se você estiver usando sacos de erro nomeados, você pode passar o nome do saco de erro como o segundo argumento para a diretiva @error:

```php
<input ... class="@error('title', 'post') é inválido @enderror">
```

### # Repopulação de formulários

Quando o Laravel gera uma resposta de redirecionamento devido a um erro de validação, o framework irá automaticamente fazer o flash de todas as entradas da requisição para a sessão. Isso é feito para que você possa acessar convenientemente a entrada durante a próxima solicitação e preencher novamente o formulário que o usuário tentou enviar.

Para recuperar a entrada atualizada da solicitação anterior, invoque o método antigo em uma instância de Illuminate\Http\Request. O método antigo puxará os dados de entrada anteriormente exibidos da sessão:

```php
$title = $request->old('title');
```

O Laravel também fornece um antigo ajudante global. Se você estiver exibindo uma entrada antiga em um modelo Blade, é mais conveniente usar o antigo auxiliar para preencher novamente o formulário. Se não existir nenhuma entrada antiga para o campo fornecido, será retornado null:

```php
<input type="text" name="title" value="{{ old('title') }}">
```

### # Uma Nota Sobre Campos Opcionais

Por padrão, o Laravel inclui o middleware TrimStrings e ConvertEmptyStringsToNull na pilha de middleware global do seu aplicativo. Esses middlewares são listados na pilha pela classe App\Http\Kernel. Por isso, muitas vezes você precisará marcar seus campos de solicitação "opcionais" como anuláveis ​​se não quiser que o validador considere valores nulos como inválidos. Por exemplo:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

Neste exemplo, estamos especificando que o campo publish_at pode ser nulo ou uma representação de data válida. Se o modificador anulável não for adicionado à definição da regra, o validador considerará nula uma data inválida.

### # Formato de resposta de erro de validação

Quando seu aplicativo lança uma exceção Illuminate\Validation\ValidationException e a solicitação HTTP recebida está esperando uma resposta JSON, o Laravel formata automaticamente as mensagens de erro para você e retorna uma resposta HTTP 422 Unprocessable Entity.

Abaixo, você pode revisar um exemplo do formato de resposta JSON para erros de validação. Observe que as chaves de erro aninhadas são achatadas no formato de notação "ponto":

```php
{
    "message": "O nome da equipe deve ser uma string. (e mais 4 erros)",
    "erros": {
        "name_team": [
            "O nome da equipe deve ser uma string.",
            "O nome da equipe deve ter pelo menos 1 caractere."
        ],
        "authorization.role": [
            "A autorização.role selecionada é inválida."
        ],
        "users.0.email": [
            "O campo users.0.email é obrigatório."
        ],
        "users.2.email": [
            "O users.2.email deve ser um endereço de e-mail válido."
        ]
    }
}
```

## # Validação de solicitação de formulário

### # Criando solicitações de formulário

Para cenários de validação mais complexos, você pode criar uma "solicitação de formulário". Solicitações de formulário são classes de solicitação personalizadas que encapsulam sua própria lógica de validação e autorização. Para criar uma classe de solicitação de formulário, você pode usar o comando make:request Artisan CLI:

```php
php artesão make:request StorePostRequest
```

A classe de solicitação de formulário gerada será colocada no diretório app/Http/Requests. Se este diretório não existir, ele será criado quando você executar o comando make:request. Cada solicitação de formulário gerada pelo Laravel possui dois métodos: autorizar e regras.

Como você deve ter adivinhado, o método authorize é responsável por determinar se o usuário atualmente autenticado pode realizar a ação representada pela solicitação, enquanto o método rules retorna as regras de validação que devem ser aplicadas aos dados da solicitação:

```php
/**
 * Obtenha as regras de validação que se aplicam à solicitação.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

Então, como as regras de validação são avaliadas? Tudo o que você precisa fazer é digitar a solicitação no método do controlador. A solicitação de formulário recebida é validada antes que o método do controlador seja chamado, o que significa que você não precisa sobrecarregar seu controlador com nenhuma lógica de validação:

```php
/**
 * Armazene uma nova postagem no blog.
 *
 * @param \App\Http\Requests\StorePostRequest $request
 * @return Iluminar\Http\Resposta
 */
public function store (StorePostRequest $request)
{
    // A requisição recebida é válida...
 
    // Recupera os dados de entrada validados...
    $validated = $request->validated();
 
    // Recupera uma parte dos dados de entrada validados...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);
}
```

Se a validação falhar, uma resposta de redirecionamento será gerada para enviar o usuário de volta ao local anterior. Os erros também serão exibidos na sessão para que fiquem disponíveis para exibição. Se a solicitação for uma solicitação XHR, uma resposta HTTP com um código de status 422 será retornada ao usuário, incluindo uma representação JSON dos erros de validação.


#### # Adicionando ganchos posteriores a solicitações de formulário

Se você quiser adicionar um gancho de validação "depois" a uma solicitação de formulário, você pode usar o método withValidator. Este método recebe o validador totalmente construído, permitindo que você chame qualquer um de seus métodos antes que as regras de validação sejam realmente avaliadas:

```php
/**
 * Configure a instância do validador.
 *
 * @param \Illuminate\Validation\Validator $validator
 * @return nulo
 */
public function withValidator($validator)
{
    $validador->after(function ($validador) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Algo está errado com este campo!');
        }
    });
}
```

#### # Parando no primeiro atributo de falha de validação

Ao adicionar uma propriedade stopOnFirstFailure à sua classe de solicitação, você pode informar ao validador que ele deve parar de validar todos os atributos assim que ocorrer uma única falha de validação:

```php
/**
 * Indica se o validador deve parar na primeira falha de regra.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

#### # Personalizando o local de redirecionamento

Conforme discutido anteriormente, uma resposta de redirecionamento será gerada para enviar o usuário de volta ao local anterior quando a validação da solicitação de formulário falhar. No entanto, você pode personalizar esse comportamento. Para fazer isso, defina uma propriedade $redirect em sua solicitação de formulário:

```php
/**
 * O URI para o qual os usuários devem ser redirecionados se a validação falhar.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

Ou, se quiser redirecionar os usuários para uma rota nomeada, você pode definir uma propriedade $redirectRoute:

```php
/**
 * A rota para a qual os usuários devem ser redirecionados se a validação falhar.
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```

### # Autorização de solicitações de formulário

A classe de solicitação de formulário também contém um método de autorização. Nesse método, você pode determinar se o usuário autenticado realmente tem autoridade para atualizar um determinado recurso. Por exemplo, você pode determinar se um usuário realmente possui um comentário de blog que está tentando atualizar. Muito provavelmente, você interagirá com seus portões e políticas de autorização neste método:

```php
use App\Models\Comment;
 
/**
 * Determine se o usuário está autorizado a fazer esta solicitação.
 *
 * @return bool
 */
function public autorizar()
{
    $comment = Comment::find($this->route('comment'));
 
    return $comment && $this->user()->can('update', $comment);
}
```

Como todas as solicitações de formulário estendem a classe de solicitação básica do Laravel, podemos usar o método user para acessar o usuário atualmente autenticado. Além disso, observe a chamada para o método de rota no exemplo acima. Esse método concede acesso aos parâmetros de URI definidos na rota que está sendo chamada, como o parâmetro {comment} no exemplo abaixo:

```php
Rote::post('/comment/{comment}');
```

Portanto, se sua aplicação está aproveitando a vinculação do modelo de rota, seu código pode ficar ainda mais sucinto acessando o modelo resolvido como propriedade da requisição:

```php
return $this->user()->can('update', $this->comment);
```

Se o método authorize retornar false, uma resposta HTTP com um código de status 403 será retornada automaticamente e o método do controlador não será executado.

Se você planeja manipular a lógica de autorização para a solicitação em outra parte do seu aplicativo, você pode simplesmente retornar true do método authorize:

```php
/**
 * Determine se o usuário está autorizado a fazer esta solicitação.
 *
 * @return bool
 */
function public authorize()
{
    return true;
}
```

### # Personalizando as mensagens de erro

Você pode personalizar as mensagens de erro usadas pela solicitação de formulário substituindo o método messages. Este método deve retornar um array de pares atributo/regra e suas mensagens de erro correspondentes:

```php
/**
 * Obtenha as mensagens de erro para as regras de validação definidas.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'Um título é obrigatório',
        'body.required' => 'Uma mensagem é obrigatória',
    ];
}
```

#### # Personalizando os atributos de validação

Muitas das mensagens de erro de regra de validação internas do Laravel contêm um espaço reservado :attribute. Se você quiser que o espaço reservado :attribute de sua mensagem de validação seja substituído por um nome de atributo personalizado, você pode especificar os nomes personalizados substituindo o método de atributos. Este método deve retornar um array de pares atributo/nome:

```php
/**
 * Obtenha atributos personalizados para erros do validador.
 *
 * @return array
 */
public function attributes()
{
    Retorna [
        'e-mail' => 'email address',
    ];
}
```

### # Preparando a entrada para validação
Se você precisar preparar ou higienizar quaisquer dados da solicitação antes de aplicar suas regras de validação, poderá usar o método prepareForValidation:

```php
use Illuminate\Support\Str;
 
/**
 * Preparar os dados para validação.
 *
 * @return nulo
 */
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

## # Criando validadores manualmente

Se você não quiser usar o método validate na solicitação, poderá criar uma instância do validador manualmente usando a fachada Validator. O método make na fachada gera uma nova instância do validador:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
 
class PostController extends Controller
{
    /**
     * Armazene uma nova postagem no blog.
     *
     * @param Request $request
     * @return Response
     */
    public function store (Request $ request)
    {
        $validador = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);
 
        if ($validador->falha()) {
            return redirect('post/create')
                        ->withErrors($validador)
                        ->comEntrada();
        }
 
        // Recupera a entrada validada...
        $validado = $validador->validado();
 
        // Recupera uma parte da entrada validada...
        $validado = $validador->seguro()->somente(['name', 'email']);
        $validado = $validador->seguro()->exceto(['name', 'email']);
 
        // Armazenar a postagem do blog...
    }
}
```


O primeiro argumento passado para o método make são os dados em validação. O segundo argumento é uma matriz das regras de validação que devem ser aplicadas aos dados.

Depois de determinar se a validação da solicitação falhou, você pode usar o método withErrors para exibir as mensagens de erro na sessão. Ao usar esse método, a variável $errors será compartilhada automaticamente com suas visualizações após o redirecionamento, permitindo que você as exiba facilmente para o usuário. O método withErrors aceita um validador, um MessageBag ou um array PHP.

**Parando na primeira falha de validação**

O método stopOnFirstFailure informará ao validador que ele deve parar de validar todos os atributos assim que ocorrer uma única falha de validação:

```php
if ($validador->stopOnFirstFailure()->fails()) {
    // ...
}
```

### #Redirecionamento automático

Se você deseja criar uma instância do validador manualmente, mas ainda aproveita o redirecionamento automático oferecido pelo método de validação da solicitação HTTP, pode chamar o método de validação em uma instância de validador existente. Se a validação falhar, o usuário será redirecionado automaticamente ou, no caso de uma solicitação XHR, uma resposta JSON será retornada:

```php
Validador::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

Você pode usar o método validateWithBag para armazenar as mensagens de erro em uma bolsa de erro nomeada se a validação falhar:

```php
Validador::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### # Sacos de erro nomeados

Se você tiver vários formulários em uma única página, poderá nomear o MessageBag que contém os erros de validação, permitindo que você recupere as mensagens de erro de um formulário específico. Para conseguir isso, passe um nome como segundo argumento para withErrors:

```php
return redirect('register')->withErrors($validator, 'login');
```

Você pode então acessar a instância MessageBag nomeada da variável $errors:

```php
{{ $errors->login->first('email') }}
```

### # Personalizando as mensagens de erro

Se necessário, você pode fornecer mensagens de erro personalizadas que uma instância do validador deve usar em vez das mensagens de erro padrão fornecidas pelo Laravel. Existem várias maneiras de especificar mensagens personalizadas. Primeiro, você pode passar as mensagens personalizadas como o terceiro argumento para o método Validator::make:

```php
$validador = Validador::make($entrada, $regras, $mensagens = [
    'required' => 'O campo :attribute é obrigatório.',
]);
```

Neste exemplo, o espaço reservado :attribute será substituído pelo nome real do campo em validação. Você também pode utilizar outros marcadores de posição nas mensagens de validação. Por exemplo:

```php
$messages = [
    'same' => 'O :attribute e :other devem corresponder.',
    'size' => 'O :attribute deve ser exatamente :size.',
    'between' => 'O :attribute value :input não está entre :min - :max.',
    'in' => 'O :attribute deve ser um dos seguintes tipos: :values',
];
```

### # Especificando uma mensagem personalizada para um determinado atributo

Às vezes, você pode querer especificar uma mensagem de erro personalizada apenas para um atributo específico. Você pode fazer isso usando a notação "ponto". Especifique primeiro o nome do atributo, seguido pela regra:

```php
$messages = [
    'email.required' => 'Precisamos saber seu endereço de e-mail!',
];
```

### # Especificando valores de atributos personalizados

Muitas das mensagens de erro internas do Laravel incluem um espaço reservado :attribute que é substituído pelo nome do campo ou atributo em validação. Para personalizar os valores usados ​​para substituir esses espaços reservados para campos específicos, você pode passar uma matriz de atributos personalizados como o quarto argumento para o método Validator::make:

```php
$validador = Validador::make($input, $rules, $messages, [
    'email' => 'endereço de e-mail',
]);
```


### #  Após Gancho de Validação

Você também pode anexar retornos de chamada para serem executados após a conclusão da validação. Isso permite realizar validações adicionais com facilidade e até mesmo adicionar mais mensagens de erro à coleção de mensagens. Para começar, chame o método after em uma instância do validador:

```php
$Validator = Validator::make(/* ... */);
 
$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $$validator->errors()->add(
            'field', 'Algo está errado com este campo!'
        );
    }
});

if ($validator->fails()) {
    //
}
```

### # Trabalhando com entrada validada

Depois de validar os dados da solicitação de entrada usando uma solicitação de formulário ou uma instância do validador criada manualmente, talvez você queira recuperar os dados da solicitação de entrada que realmente foram validados. Isso pode ser feito de várias maneiras. Primeiro, você pode chamar o método validado em uma solicitação de formulário ou instância do validador. Este método retorna uma matriz dos dados que foram validados:

```php
$validated = $request->validated();
 
$validated = $validator->validated();
```

Como alternativa, você pode chamar o método seguro em uma solicitação de formulário ou instância do validador. Este método retorna uma instância de Illuminate\Support\ValidatedInput. Este objeto expõe apenas, exceto, e todos os métodos para recuperar um subconjunto dos dados validados ou toda a matriz de dados validados:

```php
$validated = $request->safe()->only(['name', 'email']);
 
$validated = $request->safe()->except(['name', 'email']);
 
$validated = $request->safe()->all();
```

Além disso, a instância Illuminate\Support\ValidatedInput pode ser iterada e acessada como uma matriz:

```php
// Os dados validados podem ser iterados...
foreach ($request->safe() as $key => $value) {
    //
}
 
// Os dados validados podem ser acessados ​​como um array...
$validated = $request->safe();
 
$email = $validated['email'];
```

Se você quiser adicionar campos adicionais aos dados validados, você pode chamar o método de mesclagem:

```php
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);
```

Se você deseja recuperar os dados validados como uma instância de coleta, pode chamar o método collect:

```php
$collection = $request->safe()->collect();
```

### # Trabalhando com mensagens de erro

Após chamar o método errors em uma instância do Validator, você receberá uma instância Illuminate\Support\MessageBag, que possui vários métodos convenientes para trabalhar com mensagens de erro. A variável $errors que é disponibilizada automaticamente para todas as exibições também é uma instância da classe MessageBag.


### # Recuperando a primeira mensagem de erro para um campo

Para recuperar a primeira mensagem de erro de um determinado campo, use o primeiro método:

```php
$errors = $validated->errors();
 
echo $errors->first('email');
```
    
### # Recuperando todas as mensagens de erro para um campo

Se você precisar recuperar uma matriz de todas as mensagens de um determinado campo, use o método get:

```php
foreach ($errors->get('email') as $message) {
    //
}
```

Se você estiver validando um campo de formulário de matriz, poderá recuperar todas as mensagens para cada um dos elementos da matriz usando o caractere *:

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

#### # Recuperando todas as mensagens de erro para todos os campos

Para recuperar uma matriz de todas as mensagens de todos os campos, use o método all:

```php
foreach ($errors->all() as $message) {
    //
}
```

#### # Determinando se existem mensagens para um campo

O método has pode ser usado para determinar se existem mensagens de erro para um determinado campo:

```php
if ($errors->has('email')) {
    //
}
```

### # Especificando mensagens personalizadas em arquivos de idioma

Cada uma das regras de validação integradas do Laravel tem uma mensagem de erro localizada no arquivo lang/en/validation.php da sua aplicação. Nesse arquivo, você encontrará uma entrada de tradução para cada regra de validação. Você é livre para alterar ou modificar essas mensagens com base nas necessidades do seu aplicativo.

Além disso, você pode copiar este arquivo para outro diretório de idioma de tradução para traduzir as mensagens para o idioma do seu aplicativo. Para saber mais sobre a localização do Laravel, confira a documentação completa de localização.

#### # Mensagens personalizadas para atributos específicos

Você pode personalizar as mensagens de erro usadas para as combinações de atributos e regras especificadas nos arquivos de linguagem de validação de seu aplicativo. Para fazer isso, adicione suas personalizações de mensagem à matriz personalizada do arquivo de idioma lang/xx/validation.php do seu aplicativo:

```php
'custom' => [
    'e-mail' => [
        'required' => 'Precisamos saber seu endereço de e-mail!',
        'max' => 'Seu endereço de e-mail é muito longo!'
    ],
],
```

### # Especificando atributos em arquivos de idioma

Muitas das mensagens de erro internas do Laravel incluem um espaço reservado :attribute que é substituído pelo nome do campo ou atributo em validação. Se você quiser que a parte :attribute de sua mensagem de validação seja substituída por um valor personalizado, você pode especificar o nome do atributo personalizado na matriz de atributos do seu arquivo de idioma lang/xx/validation.php:

```php
'atributos' => [
    'email' => 'endereço de e-mail',
],
```

### # Especificando valores em arquivos de idioma

Algumas das mensagens de erro de regra de validação internas do Laravel contêm um espaço reservado :value que é substituído pelo valor atual do atributo request. No entanto, você pode ocasionalmente precisar que a parte :value de sua mensagem de validação seja substituída por uma representação personalizada do valor. Por exemplo, considere a seguinte regra que especifica que um número de cartão de crédito é obrigatório se o payment_type tiver um valor de cc:

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

Se essa regra de validação falhar, ela produzirá a seguinte mensagem de erro:


> O campo do número do cartão de crédito é obrigatório quando o tipo de pagamento é cc.


Em vez de exibir cc como o valor do tipo de pagamento, você pode especificar uma representação de valor mais amigável em seu arquivo de idioma lang/xx/validation.php definindo uma matriz de valores:

```php
'values' => [
    'payment_type' => [
        'cc' => 'cartão de crédito'
    ],
],
```

Após definir esse valor, a regra de validação produzirá a seguinte mensagem de erro:

> O campo número do cartão de crédito é obrigatório quando o tipo de pagamento é cartão de crédito.

## # Regras de Validação Disponíveis

Abaixo está uma lista de todas as regras de validação disponíveis e suas funções:

[Accepted](#accepted)

[Accepted If](#acceptedif)

[Active URL](#activeurl)

[After (Date)](#afterdate)

[After Or Equal(Date)](#afterequal)

[Alpha](#alpha)

[Alpha Dash](#alpha_dash)

[Alpha Numeric](#alpha_num)

[Array](#array)

[Bail](#bail)

[Before (Dates)](#before)

[Before Or Equal](#beforeequal)

[Between](#between)

[Boolean](#boolean)

[Confirmed](#confirmed)

[Current Password](#current_password)

[Date](#date)

[Date Equals](#dataequals)

[Date Format](#date_format)

[Declined](#declined)

[Declined If](#declined_if)

[Diffent](#different)

[Digits](#digits)

[Digits Between](#digits_between)

[Dimensions (Image Files)](#dimensions)

[Distinct](#distinct)

[Doesnt Start With](#doesnt_start)

[Doesnt End With](#doesnt_end)

[Email](#email)

[Ends With](#ends_with)

[Enum](#enum)

[Exclude](#exclude)

[Exclude If](#exclude_if)

[Exclude Unless](#exclude_unless)

[Exclude With](#exlucde_with)

[Exclude Without](#exclude_without)

[Exits](#exits)

[File](#file)

[Filled](#filled)

[Greater Than](#greater_than)

[Greater Than Or Equal](#greater_than_equal)

[Image (File)](#image_file)

[In](#in)

[In Array](#in_array)

[Integer](#integer)

[Ip Address](#ip)

[Json](#json)

[Less Than](#less_than)

[Less Than Or Equal](#less_than_equal)

[MAC Address](#mac)

[Max](#max)

[MIME Types](#mime)

[MIME Types By File Extension](#mime_ext)

[Min](#min)

[Min Digits](#mim_digits)

[Multiple Of](#multiple_of)

[Not In](#not_in)



#### # accepted<a id="accepted"></a> 

O campo em validação deve ser `yes`, `on`, `1` ou `true`. Isso é útil para validar a aceitação de “Termos de Serviço” ou campos semelhantes.

#### # accept_if:anotherfield,value,…<a id="acceptedif"></a> 

O campo em validação deve ser `yes`, `on`, `1` ou `true` se outro campo em validação for igual a um valor especificado. Isso é útil para validar a aceitação de “Termos de Serviço” ou campos semelhantes.

#### # active_url<a id="activeurl"></a>

O campo em validação deve ter um registro A ou AAAA válido de acordo com a função PHP `dns_get_record`. O nome do host da URL fornecida é extraído usando a função PHP `parse_url` antes de ser passado para `dns_get_record`.

#### # after:date <a id="#afterdate"></a>

O campo em validação deve ser um valor após uma determinada data. As datas serão passadas para a função PHP `strtotime` para serem convertidas em uma instância `DateTime` válida:

```php
'data_inicial' => 'obrigatório|data|após:amanhã'
```

Em vez de passar uma string de data para ser avaliada por `strtotime`, você pode especificar outro campo para comparar com a data:

```php
'finish_date' => 'required|date|after:start_date'
```

#### # after_or_equal:date <a id="#afterequal"></a>

O campo em validação deve ser um valor posterior ou igual à data informada. Para obter mais informações, veja a regra [`after`](#afterdate).

#### # alpha<a id="#alpha"></a>

O campo em validação deve ser inteiramente de caracteres alfabéticos.

#### # alpha_dash<a id="#alpha_dash"></a>

O campo em validação pode conter caracteres alfanuméricos, bem como travessões e sublinhados.

#### # alpha_num<a id="#alpha_num"></a>

O campo sob validação deve ser inteiramente de caracteres alfanuméricos.

#### # array<a id="#array"></a>

O campo sob validação deve ser um `array` PHP.

Quando valores adicionais são fornecidos à regra `array`, cada chave na matriz de entrada deve estar presente na lista de valores fornecida à regra. No exemplo a seguir, a chave `admin` na matriz de entrada é inválida, pois não está contida na lista de valores fornecida à regra `array`:

```php
use Illuminate\Support\Facades\Validator;
 $input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];
 
Validator::make($input, [
    'user' => 'array:name,username',
]);
```

Em geral, você deve sempre especificar as chaves do array que devem estar presentes no seu array.

#### # bail <a id="bail"></a>

Pare de executar regras de validação para o campo após a primeira falha de validação.

Embora a regra de `bail` só pare de validar um campo específico quando encontrar uma falha de validação, o método `stopOnFirstFailure` informará ao validador que ele deve parar de validar todos os atributos assim que ocorrer uma única falha de validação:

```php 
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

#### # before:date <a id="before"></a>

O campo em validação deve ser um valor anterior à data informada. As datas serão passadas para a função PHP `strtotime` para serem convertidas em uma instância válida de `DateTime`. Além disso, como a regra `after`, o nome de outro campo em validação pode ser fornecido como o valor de `data`.

#### # before_or_equal:date <a id="beforeequal"></a>

O campo em validação deve ser um valor anterior ou igual à data informada. As datas serão passadas para a função PHP `strtotime` para serem convertidas em uma instância válida de `DateTime`. Além disso, como a regra `after`, o nome de outro campo em validação pode ser fornecido como o valor de `data`.

#### # between:min,max <a id="between"></a>

O campo em validação deve ter um tamanho entre o mínimo e o máximo especificados (inclusive). Strings, numéricos, arrays e arquivos são avaliados da mesma forma que a regra de `size`.

#### # boolean <a id="boolean"></a>

O campo em validação deve poder ser convertido como um booleano. As entradas aceitas são `true`, `false`, `1`, `0`, `“1”` e `“0”`.


#### # confirmed <a id="confirmed"></a>

O campo em validação deve ter um campo correspondente `{field}_confirmation`. Por exemplo, se o campo sob validação for `password`, um campo correspondente `password_confirmation` deverá estar presente na entrada.

#### # current_password <a id="current_password"></a>

O campo em validação deve corresponder à senha do usuário autenticado. Você pode especificar um protetor de autenticação (`authenticatin guard`) usando o primeiro parâmetro da regra:

```php
'senha' => 'senha_atual:api'
```

#### # date <a id="date"> </a>

O campo sob validação deve ser uma data válida e não relativa de acordo com a função strtotime do PHP.

#### # date_equals:date <a id="dateequals"></a>

O campo em validação deve ser igual à data informada. As datas serão passadas para a função PHP `strtotime` para serem convertidas em uma instância válida de `DateTime`.

#### # date_format:format <a id="date_format"></a>

O campo em validação deve corresponder ao formato fornecido. Você deve usar date ou `date_format` ao validar um campo, não ambos. Esta regra de validação suporta todos os formatos suportados pela classe `DateTime` do PHP.

#### # declined <a id="declined"></a>

O campo em validação deve ser `“no”`, `“off”`, `0` ou `false`.

#### # declined_if:anotherfield,value,… <a id="declined_if"></a>

O campo em validação deve ser `“no”`, `“off”`, `0`, ou `false` se outro campo em validação for igual a um valor especificado.

#### # different:field <a id="different"></a>

O campo em validação deve ter um valor diferente de *field*.

#### # digits:value <a id="digits"> </a>

O inteiro sob validação deve ter um comprimento exato de *value*.

#### # digits_between:min,max <a id="digits_between"></a>

A validação de inteiro deve ter um comprimento entre o mínimo(*min*) e o máximo(*max*) fornecidos.

#### # dimensions <a id="dimensions"></a>

O arquivo em validação deve ser uma imagem que atenda às restrições de dimensão conforme especificado pelos parâmetros da regra:

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

As restrições disponíveis são: *min_width*, *max_width*, *min_height*, *max_height*, *width*, *height*, *ratio*.

Uma restrição de proporção deve ser representada como largura dividida pela altura. Isso pode ser especificado por uma fração como `3/2` ou um float como `1.5`:

```php
'avatar' => 'dimensões:proporção=3/2'
```

Como esta regra requer vários argumentos, você pode usar o método `Rule::dimensions` para construir a regra fluentemente:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

#### # distinct <a id="distinct"></a>

Ao validar arrays, o campo em validação não deve ter valores duplicados:

```php
'foo.*.id' => 'distinto'
```

Distinct usa comparações de variáveis soltas por padrão. Para usar comparações estritas, você pode adicionar o parâmetro `strict` à sua definição de regra de validação:

```php
'foo.*.id' => 'distinto:strict'
```

Você pode adicionar `ignore_case` aos argumentos da regra de validação para fazer com que a regra ignore as diferenças de capitalização:

```php 
'foo.*.id' => 'distinto:ignore_case'
```

#### # doesnt_start_with:foo,bar,… <a id="doesnt_start"></a>

O campo em validação não deve iniciar com um dos valores fornecidos.

#### # doesnt_end_with:foo,bar,… <a id="doesnt_end"></a>

O campo em validação não deve terminar com um dos valores fornecidos.

#### # email <a id="email"></a>

O campo em validação deve ser formatado como endereço de email. Esta regra de validação utiliza o pacote `egulias/email-validator` para validar o endereço de email. Por padrão, o validador `RFCValidation` é aplicado, mas você também pode aplicar outros estilos de validação:

```php
'e-mail' => 'e-mail:rfc,dns'
```

O exemplo acima aplicará as validações `RFCValidation` e `DNSCheckValidation`. Aqui está uma lista completa de estilos de validação que você pode aplicar:

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`
- `filter_unicode`: `FilterEmailValidatio::unicode()`

O validador `filter`, que usa a função `filter_var` do PHP, vem com o Laravel e era o comportamento padrão de validação de e-mail do Laravel antes do Laravel versão 5.8.

> Os validadores dns e spoof requerem a extensão intl do PHP.

#### # ends_with:foo,bar,… <a id="ends_with"></a>

O campo em validação deve terminar com um dos valores fornecidos.

#### # enum <a id="exclude"></a>

A regra `Enum` é uma regra baseada em classe que valida se o campo sob validação contém um valor enum válido. A regra `Enum` aceita o nome do enum como seu único argumento construtor:

```php
use App\Enums\ServerStatus;
use Illuminate\Validation\Rules\Enum;
 
$request->validate([
    'status' => [new Enum(ServerStatus::class)],
]);
```

Enums estão disponíveis apenas no PHP 8.1+.

#### # exclude <a id="exclude"></a>

O campo em validação será excluído dos dados da solicitação retornados pelos métodos `validate` e `validated`.

#### # exclude_if:anotherfield,value <a id="exclude_if"></a>

O campo em validação será excluído dos dados da solicitação retornados pelos métodos `validate` e `validated` se o campo otherfield for igual a value.

Se a logia de exclusão for complexa, você poderá utilizar o método `Rule::excludeIf`. Este método aceita um booleano ou uma *clojure*. Ao receber uma *clojure*, a *clojure* deve retornar `true` ou `false` para indicar se o campo em validação deve ser excluído:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
Validator::make($request->all(), [
    'role_id' => Rule::excludeIf($request->user()->is_admin),
]);
 
Validator::make($request->all(), [
    'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
]);
```


#### # exclude_unless:anotherfield,value <a id="exclude_unless"></a>

O campo em validação será excluído dos dados da solicitação retornados pelos métodos `validate` e `validated`, a menos que o campo *anotherfield* seja igual a *value*. Se o valor for `null` (`exclude_unless:name,null`), o campo em validação será excluído, a menos que o campo de comparação seja `null` ou o campo de comparação esteja ausente dos dados da solicitação.

#### # exclude_with:anotherfield <a id="exclude_with"></a>

O campo em validação será excluído dos dados da solicitação retornados pelos métodos `validate` e `validated` se o campo *anotherfield* estiver presente.

#### # exclude_without:anotherfield <a id="exclude_without"></a>

O campo em validação será excluído dos dados da requisição retornados pelos métodos `validate` e `validated` se o campo *anotherfield* não estiver presente.

#### # existe:table,column

O campo sob validação deve existir em uma determinada tabela do banco de dados.

#### # Uso Básico da Regra Exists

```php
'state' => 'exists:states'
```

Se a opção `column` não for especificada, o nome do campo será usado. Portanto, nesse caso, a regra validará que a tabela do banco de dados de `states` contém um registro com um valor de coluna `state`correspondente ao valor do atributo `state` da solicitação.

#### # Especificando um nome de coluna personalizado

Você pode especificar explicitamente o nome da coluna do banco de dados que deve ser usado pela regra de validação, colocando-o após o nome da tabela do banco de dados:

```php
'state' => 'exists:states,abbreviation'
```

Ocasionalmente, pode ser necessário especificar uma conexão de banco de dados específica a ser usada para a consulta existente. Você pode fazer isso acrescentando o nome da conexão ao nome da tabela:

```php
'email' => 'exists:connection.staff,email'
```

Em vez de especificar o nome da tabela diretamente, você pode especificar o modelo Eloquent que deve ser usado para determinar o nome da tabela:

```php
'user_id' => 'exists:App\Models\User,id'
```

Se você quiser personalizar a consulta executada pela regra de validação, poderá usar a classe `Rule` para definir a regra com fluência. Neste exemplo, também especificaremos as regras de validação como um array em vez de usar o caractere `|` para delimitá-los:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function ($query) {
            return $query->where('account_id', 1);
        }),
    ],
]);
```

Você pode especificar explicitamente o nome da coluna do banco de dados que deve ser usado pela regra `exists` gerada pelo método `Rule::exists` fornecendo o nome da coluna como o segundo argumento para o método `exists`:

```php
'state' => Regra::exists('states', 'abbreviation'),
```

#### # file <a id="file"></a>

O campo validado deve ser um campo um arquivo que tenha sido enviado com sucesso.

#### # filled <a id="filled"></a>

O campo em validação não deve estar vazio quando estiver presente.

#### # gt:field <a id="greater_than"></a>

O campo em validação deve ser maior que o campo fornecido (*field*). Os dois campos devem ser do mesmo tipo. Strings, números, arrays e arquivos são avaliados usando as mesmas convenções da regra de `size`.

#### # gte:field <a id="greater_than_equal"></a>

O campo em validação deve ser maior ou igual ao campo fornecido (*field*). Os dois campos devem ser do mesmo tipo. Strings, números, arrays e arquivos são avaliados usando as mesmas convenções da regra `size`.

#### # image <a id="image"></a>

O arquivo em validação deve ser uma imagem (jpg, jpeg, png, bmp, gif, svg ou webp).

#### # in:foo,bar,… <a id="in"> </a>

O campo sob validação deve ser incluído na lista de valores fornecida. Como essa regra geralmente exige que você imploda (`implode`) um array, o método `Rule::in` pode ser usado para construir a regra com fluência:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

Quando a regra `in` é combinada com a regra `array`, cada valor no array de entrada deve estar presente na lista de valores fornecida à regra `in`. No exemplo a seguir, o código do aeroporto `LAS` no array de entrada é inválido, pois não está contido na lista de aeroportos fornecida à regra `in`:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
 
$input = [
    'airports' => ['NYC', 'LAS'],
];
 
Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
``` 

#### # in_array:anotherfield.* <a id="in_array"></a>

O campo sob validação deve existir nos valores *anotherfield*.

#### # integer <a id="integer"> </a>

O campo em validação deve ser um número inteiro.

> Esta regra de validação não verifica se a entrada é do tipo de variável “integer”, apenas se a entrada é de um tipo aceito pela regra `FILTER_VALIDATE_INT` do PHP. Se você precisar validar a entrada como sendo um número, use esta regra em combinação com a regra de validação numérica.

#### # ip<a id="ip"></a>

O campo em validação deve ser um endereço IP.

#### # ipv4

O campo em validação deve ser um endereço IPv4.

#### # ipv6

O campo em validação deve ser um endereço IPv6.

#### # json <a id="json"></a>

O campo em validação deve ser uma string JSON válida.

#### # lt:field <a id="less_than"></a>

O campo em validação deve ser menor que o campo fornecido. Os dois campos devem ser do mesmo tipo. Strings, números, arrays e arquivos são avaliados usando as mesmas convenções da regra `size`.

#### # lte:field <a id="less_than_equal"></a>

O campo em validação deve ser menor ou igual ao campo fornecido (*field*). Os dois campos devem ser do mesmo tipo. Strings, números, arrays e arquivos são avaliados usando as mesmas convenções da regra `rule`.

#### # mac_address <a id="mac"></a>

O campo em validação deve ser um endereço MAC.

#### # max:value <a id="max"></a>

O campo em validação deve ser menor ou igual a um valor máximo. Strings, numéricos, arrays e arquivos são avaliados da mesma forma que a regra `size`.

# max_digits:value

O inteiro sob validação deve ter um comprimento máximo *value*.

# mimetypes:text/plain,… <a id="mime"></a>

O arquivo sob validação deve corresponder a um dos tipos MIME fornecidos:

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

Para determinar o tipo MIME do arquivo carregado, o conteúdo do arquivo será lido e a estrutura tentará adivinhar o tipo MIME, que pode ser diferente do tipo MIME fornecido pelo cliente.

#### # mimes:foo,bar,…

O arquivo em validação deve ter um tipo MIME correspondente a uma das extensões listadas.

#### # Uso Básico da Regra MIME

```php
'photo' => 'mimes:jpg,bmp,png'
```

Mesmo que você só precise especificar as extensões, esta regra realmente valida o tipo MIME do arquivo lendo o conteúdo do arquivo e adivinhando seu tipo MIME. Uma lista completa de tipos MIME e suas extensões correspondentes pode ser encontrada no seguinte local: [Mime Types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types).

# min:value <a id="min"></a>

O campo em validação deve ter um valor mínimo. Strings, numéricos, arrays e arquivos são avaliados da mesma forma que a regra `size`.

# min_digits:value <a id="min_digits"></a>

O inteiro sob validação deve ter um comprimento mínimo de valor.

# multiple_of:value

O campo sob validação deve ser um múltiplo de value.
! A extensão PHP bcmath é necessária para usar a regra multiple_of.
# not_in:foo,bar,…
O campo sob validação não deve ser incluído na lista de valores fornecida. O método Rule::notIn pode ser usado para construir fluentemente a regra:
use Iluminar\Validação\Regra;
Validator::make($data, [
'toppings' => [
'required',
Rule::notIn(['sprinkles', 'cherries']),
],
]);
# not_regex:pattern
O campo sob validação não deve corresponder à expressão regular fornecida.
Internamente, esta regra usa a função PHP preg_match. O padrão especificado deve obedecer à mesma formatação exigida por preg_match e, portanto, também incluir delimitadores válidos. Por exemplo: 'email' => 'not_regex:/^.+$/i'.
Ao usar os padrões regex / not_regex, pode ser necessário especificar suas regras de validação usando uma matriz em vez de usar | delimitadores, especialmente se a expressão regular contiver um | personagem.
# nullable
O campo em validação pode ser nulo.
# numérico
O campo em validação deve ser numérico.
# senha
O campo em validação deve corresponder à senha do usuário autenticado.
! Esta regra foi renomeada para current_password com a intenção de removê-la no Laravel 9. Por favor, use a regra Current Password.
# presente
O campo em validação deve estar presente nos dados de entrada, mas pode estar vazio.
# proibido
O campo em validação deve ser uma string vazia ou não estar presente.
# proibido_if:anotherfield,value,…
O campo em validação deve ser uma string vazia ou não está presente se o campo otherfield for igual a qualquer valor.
Se uma lógica de proibição condicional complexa for necessária, você pode utilizar o método Rule::prohibitedIf. Este método aceita um booleano ou um encerramento. Ao receber um fechamento, o fechamento deve retornar true ou false para indicar se o campo em validação deve ser proibido:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regra;
Validator::make($request->all(), [
'role_id' => Rule::prohibitedIf($request->user()->is_admin),
]);
Validator::make($request->all(), [
'role_id' => Rule::prohibitedIf(fn() => $request->user()->is_admin),
]);
# proibido_unless:anotherfield,value,…
O campo em validação deve ser uma string vazia ou não está presente, a menos que o campo anotherfield seja igual a qualquer valor.
# proíbe:outrocampo,…
Se o campo em validação estiver presente, nenhum campo em outro campo poderá estar presente, mesmo que vazio.
# regex:pattern
O campo sob validação deve corresponder à expressão regular fornecida.
Internamente, esta regra usa a função PHP preg_match. O padrão especificado deve obedecer à mesma formatação exigida por preg_match e, portanto, também incluir delimitadores válidos. Por exemplo: 'email' => 'regex:/^.+@.+$/i'.
! Ao usar os padrões regex / not_regex, pode ser necessário especificar regras em uma matriz em vez de usar | delimitadores, especialmente se a expressão regular contiver um | personagem.
# obrigatório
O campo em validação deve estar presente nos dados de entrada e não vazio. Um campo é considerado “vazio” se uma das seguintes condições for verdadeira:
. O valor é nulo.
. O valor é uma string vazia.
. O valor é um array vazio ou um objeto Countable vazio.
. O valor é um arquivo carregado sem caminho.

# required_if:anotherfield,value,…
O campo em validação deve estar presente e não vazio se o campo otherfield for igual a qualquer valor.
Se você quiser construir uma condição mais complexa para a regra required_if, você pode usar o método Rule::requiredIf. Este método aceita um booleano ou um encerramento. Ao passar um encerramento, o encerramento deve retornar verdadeiro ou falso para indicar se o campo em validação é obrigatório:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regra;
Validator::make($request->all(), [
'role_id' => Rule::requiredIf($request->user()->is_admin),
]);
Validator::make($request->all(), [
'role_id' => Rule::requiredIf(fn() => $request->user()->is_admin),
]);
# required_unless:anotherfield,value,…
O campo em validação deve estar presente e não vazio, a menos que o campo anotherfield seja igual a qualquer valor. Isso também significa que outro campo deve estar presente nos dados da solicitação, a menos que o valor seja nulo. Se o valor for nulo (required_unless:name,null), o campo em validação será obrigatório, a menos que o campo de comparação seja nulo ou o campo de comparação esteja ausente dos dados da solicitação.
# required_with:foo,bar,…
O campo sob validação deve estar presente e não vazio somente se algum dos outros campos especificados estiver presente e não vazio.
# required_with_all:foo,bar,…
O campo sob validação deve estar presente e não vazio somente se todos os outros campos especificados estiverem presentes e não vazios.
# required_without:foo,bar,…
O campo sob validação deve estar presente e não vazio apenas quando algum dos outros campos especificados estiver vazio ou não estiver presente.
# required_without_all:foo,bar,…
O campo sob validação deve estar presente e não vazio somente quando todos os outros campos especificados estiverem vazios ou não estiverem presentes.
# required_array_keys:foo,bar,…
O campo sob validação deve ser um array e deve conter pelo menos as chaves especificadas.
# same:field
O campo fornecido deve corresponder ao campo em validação.
#size:value
O campo em validação deve ter um tamanho que corresponda ao valor fornecido. Para dados de string, o valor corresponde ao número de caracteres. Para dados numéricos, valor corresponde a um determinado valor inteiro (o atributo também deve ter a regra numérica ou inteira). Para uma matriz, o tamanho corresponde à contagem da matriz. Para arquivos, o tamanho corresponde ao tamanho do arquivo em kilobytes. Vejamos alguns exemplos:
// Valida se uma string tem exatamente 12 caracteres…
'title' => 'size:12';
// Valida se um inteiro fornecido é igual a 10…
'seats' => 'integer|size:10';
// Valida que um array tem exatamente 5 elementos…
'tags' => 'array|size:5';
// Valida se um arquivo carregado tem exatamente 512 kilobytes…
'image' => 'file|size:512';
starts_with:foo,bar,…
O campo em validação deve iniciar com um dos valores fornecidos.
# string
O campo em validação deve ser uma string. Se você quiser permitir que o campo também seja nulo, atribua a regra anulável ao campo.
# timezone
O campo em validação deve ser um identificador de fuso horário válido de acordo com a função PHP timezone_identifiers_list.
# unique:table,column
O campo sob validação não deve existir na tabela de banco de dados fornecida.
Especificando um nome de tabela/coluna personalizado:
Em vez de especificar o nome da tabela diretamente, você pode especificar o modelo Eloquent que deve ser usado para determinar o nome da tabela:
'email' => 'unique:App\Models\User,email_address'
A opção de coluna pode ser usada para especificar a coluna de banco de dados correspondente ao campo. Se a opção de coluna não for especificada, será utilizado o nome do campo em validação.
'email' => 'unique:users,email_address'
Especificando uma conexão de banco de dados personalizada
Ocasionalmente, pode ser necessário definir uma conexão personalizada para consultas de banco de dados feitas pelo Validador. Para fazer isso, você pode preceder o nome da conexão ao nome da tabela:
'email' => 'unique:connection.users,email_address'
Forçando uma regra exclusiva para ignorar um determinado ID:
Às vezes, você pode querer ignorar um determinado ID durante a validação exclusiva. Por exemplo, considere uma tela de “atualizar perfil” que inclua o nome do usuário, endereço de e-mail e local. Você provavelmente desejará verificar se o endereço de e-mail é exclusivo. No entanto, se o usuário alterar apenas o campo de nome e não o campo de email, você não deseja que um erro de validação seja gerado porque o usuário já é o proprietário do endereço de email em questão.
Para instruir o validador a ignorar o ID do usuário, usaremos a classe Rule para definir a regra com fluência. Neste exemplo, também especificaremos as regras de validação como uma matriz em vez de usar o | caractere para delimitar as regras:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regra;
Validator::make($data, [
'email' => [
'required',
Rule::unique('users')->ignore($user->id),
],
]);
! Você nunca deve passar nenhuma entrada de solicitação controlada pelo usuário para o método ignore. Em vez disso, você deve passar apenas um ID exclusivo gerado pelo sistema, como um ID de incremento automático ou UUID de uma instância do modelo Eloquent. Caso contrário, seu aplicativo ficará vulnerável a um ataque de injeção de SQL.
Em vez de passar o valor da chave do modelo para o método ignore, você também pode passar toda a instância do modelo. O Laravel extrairá automaticamente a chave do modelo:
Rule::unique('users')->ignore($user)
Se sua tabela usa um nome de coluna de chave primária diferente de id, você pode especificar o nome da coluna ao chamar o método ignore:
Rule::unique('users')->ignore($user->id, 'user_id')
Por padrão, a regra exclusiva verificará a exclusividade da coluna correspondente ao nome do atributo que está sendo validado. No entanto, você pode passar um nome de coluna diferente como segundo argumento para o método exclusivo:
Rule::unique('users', 'email_address')->ignore($user->id),
Adicionando Cláusulas Where Adicionais:
Você pode especificar condições de consulta adicionais personalizando a consulta usando o método where. Por exemplo, vamos adicionar uma condição de consulta que abranja a consulta apenas para pesquisar registros que tenham um valor de coluna account_id igual a 1:
'email' => Regra::unique('users')->where(fn ($query) => $query->where('account_id', 1))
# url
O campo em validação deve ser uma URL válida.
# uuid
O campo em validação deve ser um identificador universalmente exclusivo (UUID) válido RFC 4122 (versão 1, 3, 4 ou 5).
# Adicionando regras condicionalmente
# Ignorando a validação quando os campos têm determinados valores
Ocasionalmente, você pode desejar não validar um determinado campo se outro campo tiver um determinado valor. Você pode fazer isso usando a regra de validação exclude_if. Neste exemplo, os campos compromisso_data e médico_nome não serão validados se o campo has_appointment tiver um valor false:
use Illuminate\Support\Facades\Validator;
$validator = Validator::make($data, [
'has_appointment' => 'required|boolean',
'appointment_date' => 'exclude_if:has_appointment,false|required|date',
'doctor_name' => 'exclude_if:has_appointment, false|required|string',
]);
Como alternativa, você pode usar a regra exclude_unless para não validar um determinado campo, a menos que outro campo tenha um determinado valor:
$validator = Validator::make($data, [
'has_appointment' => 'required|boolean',
'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
'doctor_name' => 'exclude_unless:has_appointment, true|required|string',
]);
# Validando quando presente
Em algumas situações, você pode querer executar verificações de validação em um campo somente se esse campo estiver presente nos dados que estão sendo validados. Para fazer isso rapidamente, adicione a regra às vezes à sua lista de regras:
$v = Validador::make($dados, [
'email' => 'às vezes|requerido|email',
]);
No exemplo acima, o campo email só será validado se estiver presente no array $data.
Se você estiver tentando validar um campo que deve estar sempre presente, mas pode estar vazio, confira esta nota sobre campos opcionais.
# Validação condicional complexa
Às vezes você pode querer adicionar regras de validação baseadas em lógica condicional mais complexa. Por exemplo, você pode querer exigir um determinado campo apenas se outro campo tiver um valor maior que 100. Ou você pode precisar que dois campos tenham um determinado valor somente quando outro campo estiver presente. Adicionar essas regras de validação não precisa ser uma dor. Primeiro, crie uma instância do Validator com suas regras estáticas que nunca mudam:
use Illuminate\Support\Facades\Validator;
$validator = Validator::make($request->all(), [
'email' => 'required|email',
'games' => 'required|numeric',
]);
Vamos supor que nosso aplicativo da web seja para colecionadores de jogos. Se um colecionador de jogos se registrar em nosso aplicativo e possuir mais de 100 jogos, queremos que ele explique por que possui tantos jogos. Por exemplo, talvez eles tenham uma loja de revenda de jogos, ou talvez apenas gostem de colecionar jogos. Para adicionar condicionalmente esse requisito, podemos usar o método às vezes na instância do Validador.
$validador->às vezes('motivo', 'obrigatório|max:500', função ($entrada) {
return $entrada->jogos >= 100;
});
O primeiro argumento passado para o método às vezes é o nome do campo que estamos validando condicionalmente. O segundo argumento é uma lista das regras que queremos adicionar. Se o encerramento passado como o terceiro argumento retornar verdadeiro, as regras serão adicionadas. Esse método facilita muito a criação de validações condicionais complexas. Você pode até adicionar validações condicionais para vários campos de uma só vez:
$validator->às vezes(['motivo', 'custo'], 'obrigatório', função ($entrada) {
return $entrada->jogos >= 100;
});
O parâmetro $input passado para seu encerramento será uma instância de Illuminate\Support\Fluent e pode ser usado para acessar sua entrada e arquivos sob validação.
# Validação de Array Condicional Complexo
Às vezes você pode querer validar um campo com base em outro campo no mesmo array aninhado cujo índice você não conhece. Nessas situações, você pode permitir que seu encerramento receba um segundo argumento que será o item individual atual no array sendo validado:
$input = [
'channels' => [
[
'type' => 'email',
'address' => 'abigail@example.com',
],
[
'type' => 'url',
'address' => ' https://example.com ',
],
],
];
$validador->às vezes('canais.*.endereço', 'email', function ($entrada, $item) {
return $item->tipo === 'email';
});
$validator->às vezes('canais.*.endereço', 'url', function ($entrada, $item) {
return $item->tipo !== 'email';
});
Assim como o parâmetro $input passado para o encerramento, o parâmetro $item é uma instância de Illuminate\Support\Fluent quando os dados do atributo são uma matriz; caso contrário, é uma string.
# Validando arrays
Conforme discutido na documentação da regra de validação de array, a regra de array aceita uma lista de chaves de array permitidas. Se alguma chave adicional estiver presente na matriz, a validação falhará:
use Illuminate\Support\Facades\Validator;
$input = [
'user' => [
'name' => 'Taylor Otwell',
'username' => 'taylorotwell',
'admin' => true,
],
];
Validator::make($input, [
'user' => 'array:username,locale',
]);
Em geral, você deve sempre especificar as chaves de matriz que podem estar presentes em sua matriz. Caso contrário, os métodos validate e valided do validador retornarão todos os dados validados, incluindo a matriz e todas as suas chaves, mesmo que essas chaves não tenham sido validadas por outras regras de validação de matriz aninhada.
# Validando a entrada de matriz
aninhada Validar campos de entrada de formulário baseados em matriz aninhada não precisa ser uma dor. Você pode usar “notação de ponto” para validar atributos dentro de um array. Por exemplo, se a solicitação HTTP recebida contiver um campo photos[profile], você poderá validá-lo assim:
use Illuminate\Support\Facades\Validator;
$validator = Validator::make($request->all(), [
'photos.profile' => 'required|image',
]);
Você também pode validar cada elemento de um array. Por exemplo, para validar que cada email em um determinado campo de entrada de matriz é exclusivo, você pode fazer o seguinte:
$validator = Validator::make($request->all(), [
'person. .email' => 'email|unique:users',
'person. .first_name' => 'required_with:person.*.last_name' ,
]);
Da mesma forma, você pode usar o caractere * ao especificar mensagens de validação personalizadas em seus arquivos de idioma, facilitando o uso de uma única mensagem de validação para campos baseados em array:
'custom' => [
'person.*.email' => [
'unique' => 'Cada pessoa deve ter um endereço de e-mail exclusivo',
]
],
# Acessando dados de matriz aninhada
Às vezes, você pode precisar acessar o valor de um determinado elemento de matriz aninhada ao atribuir regras de validação ao atributo. Você pode fazer isso usando o método Rule::forEach. O método forEach aceita um encerramento que será invocado para cada iteração do atributo array sob validação e receberá o valor do atributo e o nome do atributo explícito e totalmente expandido. O fechamento deve retornar uma matriz de regras para atribuir ao elemento da matriz:
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regra;
$validator = Validator::make($request->all(), [
'companies.*.id' => Rule::forEach(function ($value, $attribute) {
return [
Rule::exists(Company:: class, 'id'),
new HasPermission('manage-company', $value),
];
}),
]);
# Índices e posições de mensagens de erro
Ao validar matrizes, você pode fazer referência ao índice ou posição de um item específico que falhou na validação na mensagem de erro exibida pelo seu aplicativo. Para fazer isso, você pode incluir os espaços reservados :index e :position em sua mensagem de validação personalizada:
use Illuminate\Support\Facades\Validator;
$input = [
'photos' => [
[
'name' => 'BeachVacation.jpg',
'description' => 'Uma foto das minhas férias na praia!',
],
[
'name' => 'GrandCanyon.jpg' ,
'descrição' => '',
],
],
];
Validator::validate($input, [
'photos. .description' => 'required',
], [
'photos. .description.required' => 'Por favor, descreva a foto #:position.',
]);
Dado o exemplo acima, a validação falhará e o usuário receberá o seguinte erro de “Por favor, descreva a foto nº 2”.
# Validando Arquivos
Laravel fornece uma variedade de regras de validação que podem ser usadas para validar arquivos carregados, como mimes, image, min e max. Embora você seja livre para especificar essas regras individualmente ao validar arquivos, o Laravel também oferece um construtor fluente de regras de validação de arquivos que você pode achar conveniente:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regras\Arquivo;
Validator::validate($input, [
'attachment' => [
'required',
File::types(['mp3', 'wav'])
->min(1024)
->max(12 * 1024),
] ,
]);
Se seu aplicativo aceitar imagens enviadas por seus usuários, você poderá usar o método construtor de imagem da regra de arquivo para indicar que o arquivo enviado deve ser uma imagem. Além disso, a regra de dimensões pode ser usada para limitar as dimensões da imagem:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regras\Arquivo;
Validator::validate($input, [
'photo' => [
'required',
File::image()
->min(1024)
->max(12 * 1024)
->dimensions(Rule::dimensions()- >maxWidth(1000)->maxHeight(500)),
],
]);
! Mais informações sobre a validação das dimensões da imagem podem ser encontradas na documentação da regra de dimensão.
# Tipos de arquivo
Embora você só precise especificar as extensões ao invocar o método de tipos, esse método realmente valida o tipo MIME do arquivo lendo o conteúdo do arquivo e adivinhando seu tipo MIME. Uma lista completa de tipos MIME e suas extensões correspondentes pode ser encontrada no seguinte local:
https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types
# Validando Senhas
Para garantir que as senhas tenham um nível adequado de complexidade, você pode usar o objeto de regra Senha do Laravel:
use Illuminate\Support\Facades\Validator;
use Iluminar\Validação\Regras\Senha;
$validator = Validator::make($request->all(), [
'password' => ['required', 'confirmed', Password::min(8)],
]);
O objeto Regra de senha permite que você personalize facilmente os requisitos de complexidade de senha para seu aplicativo, como especificar que as senhas exigem pelo menos uma letra, número, símbolo ou caracteres com maiúsculas e minúsculas:
// Requer pelo menos 8 caracteres…
Senha::min(8)
// Requer pelo menos uma letra…
Senha::min(8)->letters()
// Requer pelo menos uma letra maiúscula e uma minúscula…
Password::min(8)->mixedCase()
// Requer pelo menos um número…
Senha::min(8)->numbers()
// Requer pelo menos um símbolo…
Senha::min(8)->symbols()
Além disso, você pode garantir que uma senha não tenha sido comprometida em um vazamento de violação de dados de senha pública usando o método não comprometido:
Senha::min(8)->uncompromised()
Internamente, o objeto de regra Password usa o modelo k-Anonymity para determinar se uma senha vazou por meio do serviço haveibeenpwned.com sem sacrificar a privacidade ou a segurança do usuário.
Por padrão, se uma senha aparecer pelo menos uma vez em um vazamento de dados, ela será considerada comprometida. Você pode personalizar esse limite usando o primeiro argumento do método não comprometido:
// Certifique-se de que a senha apareça menos de 3 vezes no mesmo vazamento de dados…
Password::min(8)->uncompromised(3);
Claro, você pode encadear todos os métodos nos exemplos acima:
Password::min(8)
->letters()
->mixedCase()
->numbers()
->symbols()
->uncompromised()
# Definindo regras de senha padrão
Você pode achar conveniente especificar as regras de validação padrão para senhas em um único local de seu aplicativo. Você pode fazer isso facilmente usando o método Password::defaults, que aceita um encerramento. O encerramento dado ao método defaults deve retornar a configuração padrão da regra Password. Normalmente, a regra de padrões deve ser chamada no método de inicialização de um dos provedores de serviço do seu aplicativo:
use Iluminar\Validação\Regras\Senha;
/**
•	Bootstrap qualquer serviço de aplicativo.
•	
•	@return void
*/
public function boot()
{
Password::defaults(function() {
$rule = Password::min(8);
•	 return $this->app->isProduction()
•	             ? $rule->mixedCase()->uncompromised()
•	             : $rule;
});
}
Então, quando você quiser aplicar as regras padrão a uma senha específica em validação, você pode invocar o método defaults sem argumentos:
'password' => ['required', Password::defaults()],
Ocasionalmente, você pode querer anexar regras de validação adicionais às suas regras de validação de senha padrão. Você pode usar o método de regras para fazer isso:
use App\Rules\ZxcvbnRule;
Password::defaults(function () {
$rule = Password::min(8)->rules([new ZxcvbnRule]);
// ...
});
# Regras de validação personalizadas
#usando objetos de regra
Laravel fornece uma variedade de regras de validação úteis; no entanto, você pode querer especificar algumas de sua preferência. Um método de registro de regras de validação personalizadas é usar objetos de regra. Para gerar um novo objeto de regra, você pode usar o comando make:rule Artisan. Vamos usar este comando para gerar uma regra que verifica se uma string é maiúscula. O Laravel colocará a nova regra no diretório app/Rules. Se este diretório não existir, o Laravel irá criá-lo quando você executar o comando Artisan para criar sua regra:
php artesão make:rule Maiúsculas --invokable
Uma vez que a regra foi criada, estamos prontos para definir seu comportamento. Um objeto de regra contém um único método: __invoke. Este método recebe o nome do atributo, seu valor e um callback que deve ser invocado em caso de falha com a mensagem de erro de validação:
<?php
namespace App\Rules;
use Illuminate\Contracts\Validation\InvokableRule;
class Uppercase implements InvokableRule
{
/**
* Executa a regra de validação.
*
* @param string $attribute
* @param mixed $value
* @param \Closure $fail
* @return void
*/
public function __invoke($attribute, $value, $fail)
{
if (strtoupper($value) !== $value) {
$fail('O :attribute deve ser maiúsculo.');
}
}
}
Depois que a regra for definida, você pode anexá-la a um validador passando uma instância do objeto de regra com suas outras regras de validação:
use App\Regras\Maiúsculas;
$request->validate([
'name' => ['required', 'string', new Maiúsculas],
]);
Traduzindo Mensagens de Validação
Em vez de fornecer uma mensagem de erro literal para o encerramento $fail, você também pode fornecer uma chave de string de tradução e instruir o Laravel a traduzir a mensagem de erro:
if (strtoupper($valor) !== $valor) {
$fail('validation.uppercase')->translate();
}
Se necessário, você pode fornecer substituições de espaço reservado e o idioma preferido como primeiro e segundo argumentos para o método de tradução:
$fail('validation.location')->translate([
'value' => $this->value,
], 'fr')
Acessando dados adicionais
Se sua classe de regra de validação personalizada precisar acessar todos os outros dados que estão sendo validados, sua classe de regra pode implementar a interface Illuminate\Contracts\Validation\DataAwareRule. Essa interface requer que sua classe defina um método setData. Este método será automaticamente invocado pelo Laravel (antes da validação continuar) com todos os dados em validação:
<?php
namespace App\Rules;
use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\InvokableRule;
class Uppercase implementa DataAwareRule, InvokableRule
{
/**
* Todos os dados sob validação.
*
* @var array
*/
protected $data = [];
// ...

/**
 * Set the data under validation.
 *
 * @param  array  $data
 * @return $this
 */
public function setData($data)
{
    $this->data = $data;

    return $this;
}
}
Ou, se sua regra de validação requer acesso à instância do validador que está realizando a validação, você pode implementar a interface ValidatorAwareRule:
<?php
namespace App\Rules;
use Illuminate\Contracts\Validation\InvokableRule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;
class Uppercase implementa InvokableRule, ValidatorAwareRule
{
/**
* A instância do validador.
*
* @var \Illuminate\Validation\Validator
*/
protected $validator;
// ...

/**
 * Set the current validator.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return $this
 */
public function setValidator($validator)
{
    $this->validator = $validator;

    return $this;
}
}
# Usando Closures
Se você só precisa da funcionalidade de uma regra personalizada uma vez em todo o seu aplicativo, você pode usar um closure em vez de um objeto de regra. A closure recebe o nome do atributo, o valor do atributo e um callback $fail que deve ser chamado se a validação falhar:
use Illuminate\Support\Facades\Validator;
$validator = Validator::make($request->all(), [
'title' => [
'required',
'max:255',
function ($attribute, $value, $fail) {
if ($value = == 'foo') {
$fail('O '.$attribute.' é inválido.');
}
},
],
]);
# Regras implícitas
Por padrão, quando um atributo que está sendo validado não está presente ou contém uma string vazia, as regras normais de validação, incluindo regras personalizadas, não são executadas. Por exemplo, a regra exclusiva não será executada em uma string vazia:
use Illuminate\Support\Facades\Validator;
$regras = ['nome' => 'único:usuários,nome'];
$input = ['nome' => ''];
Validator::make($input, $rules)->passes(); // verdadeiro
Para que uma regra personalizada seja executada mesmo quando um atributo está vazio, a regra deve implicar que o atributo é obrigatório. Para gerar rapidamente um novo objeto de regra implícito, você pode usar o comando make:rule Artisan com a opção --implicit:
php artesão make:rule Maiúsculas --invokable --implicit
! Uma regra “implícita” implica apenas que o atributo é obrigatório. Se ele realmente invalida um atributo ausente ou vazio, depende de você.








