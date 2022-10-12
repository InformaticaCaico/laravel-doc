# Conversões: Recebendo um ínicio
## **Introdução**
## **Geração de classes de modelos**
## **Conversões de Modelos Convertores**

## Nomes das tabelas
## Chaves Primárias
## Carimbo de data/hora
## Ligação á base de dados
## Valores de atributos por defeito

## **Recuperar Modelos**
## Coleções
## Resultados do Chuking
## Chunk usado em coleções preguiçosas
## Cursores
## Subconsultas avançadas
## **Recuperação de modelos únicos / Agregados**
## Recuperar ou Criar Modelos
## Obtenção de Agregados
## **Inserção e Atualização de Modelos**
## Inserções
## Atualizações
## Atualizações em massa


## **Excluindo Modelos**
## Exclusão Suave
## Como consultar modelos com exclusão reversível

## **Modelos de Podas**
## **Modelos de Replicação**
## **Escopos de consulta**
## Escopos Globais
## Escopos Locais
## **Comparação de Modelos**
## **Eventos**
## Usando Fechamentos
## Observadores
## Eventos de Silenciamento



# Introdução

Laravel inclui Conversor, um mapeador de objetos (ORM) que torna agradável interagir com a sua base de dados. Ao utilizar o Conversor, cada tabela da base de dados tem um "Modelo" correspondente que é utilizado para interagir com essa tabela. Além de recuperar registos da tabela da base de dados, os modelos Conversores permitem-lhe inserir, atualizar, e apagar registos da tabela também.

> Antes de começar, certifique-se de configurar uma conexão de banco de dados no arquivo de configuração config/database.php do seu aplicativo. Para obter mais informações sobre como configurar seu banco de dados, consulte a documentação de configuração do banco de dados.

# Geração de classes de modelos
Para começar, vamos criar um modelo Eloquent. Os modelos geralmente ficam no diretório app\Models e estendem a classe Illuminate\Database\Eloquent\Model. Você pode usar o comando make:model Artisan para gerar um novo modelo:
```
php artisan make:model Flight
```
Se você deseja gerar uma migração de banco de dados ao gerar o modelo, você pode usar a opção --migration ou -m:

```
php artisan make:model Flight --migration
```
Você pode gerar vários outros tipos de classes ao gerar um modelo, como fábricas, semeadores, políticas, controladores e solicitações de formulário. Além disso, essas opções podem ser combinadas para criar várias classes de uma só vez:
``` php
# Generate a model and a FlightFactory class...
php artisan make:model Flight --factory
php artisan make:model Flight -f
 
# Generate a model and a FlightSeeder class...
php artisan make:model Flight --seed
php artisan make:model Flight -s
 
# Generate a model and a FlightController class...
php artisan make:model Flight --controller
php artisan make:model Flight -c
 
# Generate a model, FlightController resource class, and form request classes...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR
 
# Generate a model and a FlightPolicy class...
php artisan make:model Flight --policy
 
# Generate a model and a migration, factory, seeder, and controller...
php artisan make:model Flight -mfsc
 
# Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
php artisan make:model Flight --all
 
# Generate a pivot model...
php artisan make:model Member --pivot
```
## Inspecionando modelos
Às vezes, pode ser difícil determinar todos os atributos e relacionamentos disponíveis de um modelo apenas analisando seu código. Em vez disso, tente o comando model:show Artisan, que fornece uma visão geral conveniente de todos os atributos e relações do modelo:

```
php artisan model:show Flight
```
# Conversões de Modelos Convertores
Os modelos gerados pelo comando make:model serão colocados no diretório app/Models. Vamos examinar uma classe de modelo básico e discutir algumas das principais convenções do convertor:

``` php
?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    //
}
```
## Nomes das tabelas
Após dar uma olhada no exemplo acima, você deve ter notado que não informamos ao convertor qual tabela de banco de dados corresponde ao nosso modelo de voo. Por convenção, o nome plural "snake case" da classe será usado como o nome da tabela, a menos que outro nome seja especificado explicitamente. Portanto, neste caso, o convertor assumirá que o modelo Flight armazena registros na tabela de voos, enquanto um modelo AirTrafficController armazenaria registros em uma tabela air_traffic_controllers.

Se a tabela de banco de dados correspondente do seu modelo não se encaixa nessa convenção, você pode especificar manualmente o nome da tabela do modelo definindo uma propriedade de tabela no modelo:

``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```
## Chaves Primárias
O Convertor também assumirá que a tabela de banco de dados correspondente de cada modelo possui uma coluna de chave primária chamada id. Se necessário, você pode definir uma propriedade $primaryKey protegida em seu modelo para especificar uma coluna diferente que serve como chave primária de seu modelo:
```php
?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
}
```
Além disso, o conversor assume que a chave primária é um valor inteiro incrementado, o que significa que o conversor converterá automaticamente a chave primária em um inteiro. Se você deseja usar uma chave primária não-incremental ou não numérica, você deve definir uma propriedade public $incrementing em seu modelo que seja definida como false:
``` php
<?php
 
class Flight extends Model
{
    /**
     * Indicates if the model's ID is auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = false;
}
```
Se a chave primária do seu modelo não for um número inteiro, você deve definir uma propriedade $keyType protegida em seu modelo. Esta propriedade deve ter um valor de string:
``` php
<?php
 
class Flight extends Model
{
    /**
     * The data type of the auto-incrementing ID.
     *
     * @var string
     */
    protected $keyType = 'string';
}
```
### Chaves Primárias Compostas
O Conversos exige que cada modelo tenha pelo menos um "ID" de identificação exclusiva que possa servir como sua chave primária. Chaves primárias "compostas" não são suportadas pelos modelos Conversor. No entanto, você pode adicionar índices exclusivos de várias colunas adicionais às tabelas do banco de dados, além da chave primária de identificação exclusiva da tabela.

# Carimbo de data/hora
Por padrão, o conversor espera que as colunas created_at e updated_at existam na tabela de banco de dados correspondente do seu modelo. O Eloquent definirá automaticamente os valores dessas colunas quando os modelos forem criados ou atualizados. Se você não quiser que essas colunas sejam gerenciadas automaticamente pelo Eloquent, você deve definir uma propriedade $timestamps em seu modelo com um valor false:
```php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    public $timestamps = false;
}
```
Se você precisar personalizar o formato dos carimbos de data/hora do seu modelo, defina a propriedade $dateFormat no seu modelo. Esta propriedade determina como os atributos de data são armazenados no banco de dados, bem como seu formato quando o modelo é serializado para um array ou JSON:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```
Se você precisar personalizar os nomes das colunas usadas para armazenar os timestamps, você pode definir as constantes CREATED_AT e UPDATED_AT em seu modelo:
```php
<?php
 
class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}
```
## Ligação á base de dados
Por padrão, todos os modelos do conversor usarão a conexão de banco de dados padrão configurada para seu aplicativo. Se você deseja especificar uma conexão diferente que deve ser usada ao interagir com um modelo específico, você deve definir uma propriedade $connection no modelo:
```php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The database connection that should be used by the model.
     *
     * @var string
     */
    protected $connection = 'sqlite';
}
```
## Valores de atributos por defeito
Por padrão, uma instância de modelo recém-instanciada não conterá nenhum valor de atributo. Se você quiser definir os valores padrão para alguns dos atributos do seu modelo, você pode definir uma propriedade $attributes no seu modelo:
```php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The model's default values for attributes.
     *
     * @var array
     */
    protected $attributes = [
        'delayed' => false,
    ];
}
```
# Recuperar Modelos
Depois de criar um modelo e sua tabela de banco de dados associada, você estará pronto para começar a recuperar dados de seu banco de dados. Você pode pensar em cada modelo do conversor como um poderoso construtor de consultas que permite consultar com fluência a tabela de banco de dados associada ao modelo. O método all do modelo recuperará todos os registros da tabela de banco de dados associada ao modelo:
```php
use App\Models\Flight;
 
foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```
### Construção de consultas
O método convertor retornará todos os resultados da tabela do modelo. No entanto, como cada modelo do Eloquent serve como um construtor de consultas, você pode adicionar restrições adicionais às consultas e, em seguida, invocar o método get para recuperar os resultados:
```php
$flights = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();
```
> Como os modelos conversores são construtores de consultas, você deve revisar todos os métodos fornecidos pelo construtor de consultas do Laravel. Você pode usar qualquer um desses métodos ao escrever suas consultas do conversor.

### Modelos de atualização
Se você já tem uma instância de um modelo conversor que foi recuperado do banco de dados, você pode "atualizar" o modelo usando os métodos fresh e refresh. O novo método recuperará o modelo do banco de dados. A instância de modelo existente não será afetada:
```php
$flight = Flight::where('number', 'FR 900')->first();
 
$freshFlight = $flight->fresh();
```
O método de atualização irá reidratar o modelo existente usando dados atualizados do banco de dados. Além disso, todos os seus relacionamentos carregados também serão atualizados:
```php
$flight = Flight::where('number', 'FR 900')->first();
 
$flight->number = 'FR 456';
 
$flight->refresh();
 
$flight->number; // "FR 900"
```
## Coleções
Como vimos, os métodos conversores gostam de tudo e recuperam vários registros do banco de dados. No entanto, esses métodos não retornam um array PHP simples. Em vez disso, uma instância de Illuminate\Database\Eloquent\Collection é retornada.

A classe Conversão Collection estende a classe base Illuminate\Support\Collection do Laravel, que fornece uma variedade de métodos úteis para interagir com coleções de dados. Por exemplo, o método de rejeição pode ser usado para remover modelos de uma coleção com base nos resultados de um encerramento invocado:

```php
$flights = Flight::where('destination', 'Paris')->get();
 
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
```
Além dos métodos fornecidos pela classe base de coleção do Laravel, a classe de coleção conversão fornece alguns métodos extras que são especificamente destinados a interagir com coleções de modelos conversores.

Como todas as coleções do Laravel implementam as interfaces iteráveis ​​do PHP, você pode fazer um ciclo sobre as coleções como se fossem um array:
```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```
## Resultados do Chuking
Seu aplicativo pode ficar sem memória se você tentar carregar dezenas de milhares de registros do conversor por meio dos métodos all ou get. Em vez de usar esses métodos, o método chunk pode ser usado para processar um grande número de modelos com mais eficiência.

O método chunk recuperará um subconjunto de modelos conversores, passando-os para um encerramento para processamento. Como apenas a parte atual dos modelos do conversor é recuperada por vez, o método chunk fornecerá um uso de memória significativamente reduzido ao trabalhar com um grande número de modelos:
```php
use App\Models\Flight;
 
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```
O primeiro argumento passado para o método chunk é o número de registros que você deseja receber por "pedaço". O encerramento passado como segundo argumento será invocado para cada fragmento recuperado do banco de dados. Uma consulta de banco de dados será executada para recuperar cada pedaço de registros passados ​​para o encerramento.

Se você estiver filtrando os resultados do método chunk com base em uma coluna que você também atualizará enquanto itera sobre os resultados, você deve usar o método chunkById. O uso do método chunk nesses cenários pode levar a resultados inesperados e inconsistentes. Internamente, o método chunkById sempre recuperará modelos com uma coluna id maior que o último modelo no bloco anterior:
```php
Flight::where('departed', true)
    ->chunkById(200, function ($flights) {
        $flights->each->update(['departed' => false]);
    }, $column = 'id');
```
## Chunk usado em coleções preguiçosas
O método lazy funciona de maneira semelhante ao método chunk no sentido de que, nos bastidores, ele executa a consulta em blocos. No entanto, em vez de passar cada pedaço diretamente para um retorno de chamada como está, o método lazy retorna um LazyCollection de modelos Eloquent, que permite interagir com os resultados como um único fluxo:
```php
use App\Models\Flight;
 
foreach (Flight::lazy() as $flight) {
    //
}
```
Se você estiver filtrando os resultados do método lazy com base em uma coluna que você também atualizará enquanto itera sobre os resultados, você deve usar o método lazyById. Internamente, o método lazyById sempre recuperará modelos com uma coluna de id maior que o último modelo no bloco anterior:
```php
Flight::where('departed', true)
    ->lazyById(200, $column = 'id')
    ->each->update(['departed' => false]);
```
Você pode filtrar os resultados com base na ordem decrescente do id usando o método lazyByIdDesc.

## Cursores
Semelhante ao método lazy, o método cursor pode ser usado para reduzir significativamente o consumo de memória do seu aplicativo ao percorrer dezenas de milhares de registros do modelo conversor.

O método cursor executará apenas uma única consulta ao banco de dados; no entanto, os modelos individuais do conversor não serão hidratados até que sejam realmente interados. Portanto, apenas um modelo conversor é mantido na memória a qualquer momento durante a interação sobre o cursor.

>Como o método do cursor mantém apenas um único modelo conversor na memória por vez, ele não pode antecipar relacionamentos de carregamento. Se você precisar antecipar relacionamentos de carregamento, considere usar o método lazy.

Internamente, o método cursor usa geradores PHP para implementar esta funcionalidade:
```php
use App\Models\Flight;
 
foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    //
}
```
O cursor retorna uma instância Illuminate\Support\LazyCollection. Coleções preguiçosas onde permitem que você use muitos dos métodos de coleta disponíveis em coleções típicas do Laravel enquanto carrega apenas um único modelo na memória por vez:
```php
use App\Models\User;
 
$users = User::cursor()->filter(function ($user) {
    return $user->id > 500;
});
 
foreach ($users as $user) {
    echo $user->id;
}
```
Embora o método do cursor use muito menos memória do que uma consulta regular (mantendo apenas um único modelo conversor na memória por vez), ele ainda ficará sem memória. Isso se deve ao driver PDO do PHP armazenar em cache internamente todos os resultados brutos da consulta em seu buffer. Se você estiver lidando com um número muito grande de registros do conversor, considere usar o método lazy.

## Subconsultas avançadas
### Seleções de Subconsultas 
O conversor também oferece suporte avançado a subconsultas, que permite extrair informações de tabelas relacionadas em uma única consulta. Por exemplo, vamos imaginar que temos uma tabela de destinos de voos e uma tabela de voos para destinos. A tabela de voos contém uma coluna chegou_at que indica quando o voo chegou ao destino.

Usando a funcionalidade de subconsulta disponível para os métodos select e addSelect do construtor de consultas, podemos selecionar todos os destinos e o nome do voo que chegou mais recentemente a esse destino usando uma única consulta:
```php
use App\Models\Destination;
use App\Models\Flight;
 
return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```
### Ordenação de Subconsulta
Além disso, a função orderBy do construtor de consultas suporta subconsultas. Continuando a usar nosso exemplo de voo, podemos usar essa funcionalidade para classificar todos os destinos com base em quando o último voo chegou a esse destino. Novamente, isso pode ser feito durante a execução de uma única consulta de banco de dados:
```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```
## Recuperação de modelos únicos / Agregados
Além de recuperar todos os registros correspondentes a uma determinada consulta, você também pode recuperar registros únicos usando os métodos find, first ou firstWhere. Em vez de retornar uma coleção de modelos, esses métodos retornam uma única instância de modelo:
```php
use App\Models\Flight;
 
// Retrieve a model by its primary key...
$flight = Flight::find(1);
 
// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();
 
// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```
Às vezes você pode querer realizar alguma outra ação se nenhum resultado for encontrado. Os métodos findOr e firstOr retornarão uma única instância do modelo ou, se nenhum resultado for encontrado, executará o encerramento fornecido. O valor retornado pelo fechamento será considerado o resultado do método:
```php
$flight = Flight::findOr(1, function () {
    // ...
});
 
$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```
### Exceções não encontradas
Às vezes você pode querer lançar uma exceção se um modelo não for encontrado. Isso é particularmente útil em rotas ou controladores. Os métodos findOrFail e firstOrFail recuperarão o primeiro resultado da consulta; no entanto, se nenhum resultado for encontrado, um Illuminate\Database\Eloquent\ModelNotFoundException será lançado:
```php
$flight = Flight::findOrFail(1);
 
$flight = Flight::where('legs', '>', 3)->firstOrFail();
```
Se a ModelNotFoundException não for capturada, uma resposta HTTP 404 é enviada automaticamente de volta ao cliente:
```php
use App\Models\Flight;
 
Route::get('/api/flights/{id}', function ($id) {
    return Flight::findOrFail($id);
});
```
## Recuperar ou Criar Modelos
O método firstOrCreate tentará localizar um registro de banco de dados usando os pares de coluna/valor fornecidos. Se o modelo não puder ser encontrado no banco de dados, será inserido um registro com os atributos resultantes da mesclagem do primeiro argumento do array com o segundo argumento opcional do array: 

O método firstOrNew, como firstOrCreate, tentará localizar um registro no banco de dados que corresponda aos atributos fornecidos. No entanto, se um modelo não for encontrado, uma nova instância de modelo será retornada. Observe que o modelo retornado por firstOrNew ainda não foi persistido no banco de dados. Você precisará chamar manualmente o método save para persistir:

```php
use App\Models\Flight;
 
// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);
 
// Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
 
// Retrieve flight by name or instantiate a new Flight instance...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);
 
// Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```
## Obtenção de Agregados
Ao interagir com modelos conversores, você também pode usar o count, sum, max e outros métodos agregados fornecidos pelo construtor de consultas Laravel. Como você pode esperar, esses métodos retornam um valor escalar em vez de uma instância do modelo Eloquent:
```php
$count = Flight::where('active', 1)->count();
 
$max = Flight::where('active', 1)->max('price');
```
## **Inserção e Atualização de Modelos**

## Inserções
É claro que, ao usar o Eloquent, não precisamos apenas recuperar os modelos do banco de dados. Também precisamos inserir novos registros. Felizmente, o Eloquent torna isso simples. Para inserir um novo registro no banco de dados, você deve instanciar uma nova instância de modelo e definir atributos no modelo. Em seguida, chame o método save na instância do modelo:
```php
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use App\Models\Flight;
use Illuminate\Http\Request;
 
class FlightController extends Controller
{
    /**
     * Store a new flight in the database.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate the request...
 
        $flight = new Flight;
 
        $flight->name = $request->name;
 
        $flight->save();
    }
}
```
Neste exemplo, atribuímos o campo name da solicitação HTTP recebida ao atributo name da instância do modelo App\Models\Flight. Quando chamamos o método save, um registro será inserido no banco de dados. Os timestamps created_at e updated_at do modelo serão definidos automaticamente quando o método save for chamado, portanto, não há necessidade de defini-los manualmente.

Alternativamente, você pode usar o método create para "salvar" um novo modelo usando uma única instrução PHP. A instância de modelo inserida será devolvida a você pelo método create:
```php
use App\Models\Flight;
 
$flight = Flight::create([
    'name' => 'London to Paris',
]);
```
No entanto, antes de usar o método create, você precisará especificar uma propriedade preenchível ou protegida em sua classe de modelo. Essas propriedades são necessárias porque todos os modelos Eloquent são protegidos contra vulnerabilidades de atribuição em massa por padrão. Para saber mais sobre atribuição em massa, consulte a documentação de atribuição em massa.

## Atualizações
O método save também pode ser usado para atualizar modelos que já existem no banco de dados. Para atualizar um modelo, você deve recuperá-lo e definir os atributos que deseja atualizar. Em seguida, você deve chamar o método save do modelo. Novamente, o timestamp updated_at será atualizado automaticamente, portanto, não há necessidade de definir manualmente seu valor:
```php
use App\Models\Flight;
 
$flight = Flight::find(1);
 
$flight->name = 'Paris to London';
 
$flight->save();
```
### Inserções em massa
As atualizações também podem ser executadas em modelos que correspondem a uma determinada consulta. Neste exemplo, todos os voos ativos e com destino a San Diego serão marcados como atrasados:
```php
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```
O método de atualização espera uma matriz de pares de colunas e valores representando as colunas que devem ser atualizadas. O método de atualização retorna o número de linhas afetadas.

> Ao emitir uma atualização em massa via Eloquent, os eventos de modelo salvando, salvar, atualizando e atualizado não serão acionados para os modelos atualizados. Isso ocorre porque os modelos nunca são realmente recuperados ao emitir uma atualização em massa.

### Examinando Alterações de Atributos
O Eloquent fornece os métodos isDirty, isClean e wasChanged para examinar o estado interno de seu modelo e determinar como seus atributos foram alterados desde quando o modelo foi recuperado originalmente.

O método isDirty determina se algum dos atributos do modelo foi alterado desde que o modelo foi recuperado. Você pode passar um nome de atributo específico ou uma matriz de atributos para o método isDirty para determinar se algum dos atributos é "sujo". O método isClean determinará se um atributo permaneceu inalterado desde que o modelo foi recuperado. Este método também aceita um argumento de atributo opcional:
```php
use App\Models\User;
 
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);
 
$user->title = 'Painter';
 
$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true
 
$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false
 
$user->save();
 
$user->isDirty(); // false
$user->isClean(); // true
```
O método wasChanged determina se algum atributo foi alterado quando o modelo foi salvo pela última vez no ciclo de solicitação atual. Se necessário, você pode passar um nome de atributo para ver se um determinado atributo foi alterado:
```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);
 
$user->title = 'Painter';
 
$user->save();
 
$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```
O método getOriginal retorna uma matriz contendo os atributos originais do modelo, independentemente de quaisquer alterações no modelo desde que ele foi recuperado. Se necessário, você pode passar um nome de atributo específico para obter o valor original de um atributo específico:
```php
$user = User::find(1);
 
$user->name; // John
$user->email; // john@example.com
 
$user->name = "Jack";
$user->name; // Jack
 
$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```
## Inserções em massa
Você pode usar o método create para "salvar" um novo modelo usando uma única instrução PHP. A instância de modelo inserida será devolvida a você pelo método:
```php
use App\Models\Flight;
 
$flight = Flight::create([
    'name' => 'London to Paris',
]);
```
No entanto, antes de usar o método create, você precisará especificar uma propriedade preenchível ou protegida em sua classe de modelo. Essas propriedades são necessárias porque todos os modelos Eloquent são protegidos contra vulnerabilidades de atribuição em massa por padrão.

Uma vulnerabilidade de atribuição em massa ocorre quando um usuário passa um campo de solicitação HTTP inesperado e esse campo altera uma coluna em seu banco de dados que você não esperava. Por exemplo, um usuário mal-intencionado pode enviar um parâmetro is_admin por meio de uma solicitação HTTP, que é então passada para o método create do seu modelo, permitindo que o usuário se encaminhe para um administrador.

Portanto, para começar, você deve definir quais atributos de modelo deseja tornar atribuíveis em massa. Você pode fazer isso usando a propriedade $fillable no modelo. Por exemplo, vamos tornar o atributo name da massa do nosso modelo de voo atribuível:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```
Depois de especificar quais atributos são atribuíveis em massa, você pode usar o método create para inserir um novo registro no banco de dados. O método create retorna a instância de modelo recém-criada:
``` php
$flight = Flight::create(['name' => 'London to Paris']);
```
Se você já tem uma instância de modelo, você pode usar o método fill para preenchê-la com um array de atributos:
``` php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```
## Inserções em massa e Colunas JSON
Ao atribuir colunas JSON, a chave atribuível em massa de cada coluna deve ser especificada na matriz $fillable do seu modelo. Por segurança, o Laravel não suporta a atualização de atributos JSON aninhados ao usar a propriedade protegida:
``` php
/**
 * The attributes that are mass assignable.
 *
 * @var array
 */
protected $fillable = [
    'options->enabled',
];
```
## Permitindo Atribuição em Massa
Se você quiser tornar todos os seus atributos atribuíveis em massa, você pode definir a propriedade $guarded do seu modelo como um array vazio. Se você optar por desproteger seu modelo, você deve ter um cuidado especial para sempre criar manualmente os arrays passados ​​para os métodos fill, create e update do Eloquent:

``` php
/**
 * The attributes that aren't mass assignable.
 *
 * @var array
 */
protected $guarded = [];
```
## Atualizações
O método save também pode ser usado para atualizar modelos que já existem no banco de dados. Para atualizar um modelo, você deve recuperá-lo e definir os atributos que deseja atualizar. Em seguida, você deve chamar o método save do modelo. Novamente, o timestamp updated_at será atualizado automaticamente, portanto, não há necessidade de definir manualmente seu valor:
``` php
use App\Models\Flight;
 
$flight = Flight::find(1);
 
$flight->name = 'Paris to London';
 
$flight->save();
```
### Atualizações em massa
As atualizações também podem ser executadas em modelos que correspondem a uma determinada consulta. Neste exemplo, todos os voos ativos e com destino a San Diego serão marcados como atrasados:
``` php
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```
O método de atualização espera uma matriz de pares de colunas e valores representando as colunas que devem ser atualizadas. O método de atualização retorna o número de linhas afetadas.

> Ao emitir uma atualização em massa via Eloquent, os eventos de modelo salvar, salvar, atualizar e atualizar não serão acionados para os modelos atualizados. Isso ocorre porque os modelos nunca são realmente recuperados ao emitir uma atualização em massa.

### Examinando Alterações de Atributos
O Eloquent fornece os métodos isDirty, isClean e wasChanged para examinar o estado interno de seu modelo e determinar como seus atributos foram alterados desde quando o modelo foi recuperado originalmente.

O método isDirty determina se algum dos atributos do modelo foi alterado desde que o modelo foi recuperado. Você pode passar um nome de atributo específico ou uma matriz de atributos para o método isDirty para determinar se algum dos atributos é "sujo". O método isClean determinará se um atributo permaneceu inalterado desde que o modelo foi recuperado. Este método também aceita um argumento de atributo opcional:
``` php
use App\Models\User;
 
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);
 
$user->title = 'Painter';
 
$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true
 
$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false
 
$user->save();
 
$user->isDirty(); // false
$user->isClean(); // true
```
O método wasChanged determina se algum atributo foi alterado quando o modelo foi salvo pela última vez no ciclo de solicitação atual. Se necessário, você pode passar um nome de atributo para ver se um determinado atributo foi alterado:
``` php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);
 
$user->title = 'Painter';
 
$user->save();
 
$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```
O método getOriginal retorna uma matriz contendo os atributos originais do modelo, independentemente de quaisquer alterações no modelo desde que ele foi recuperado. Se necessário, você pode passar um nome de atributo específico para obter o valor original de um atributo específico:
``` php
$user = User::find(1);
 
$user->name; // John
$user->email; // john@example.com
 
$user->name = "Jack";
$user->name; // Jack
 
$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```
## Atribuição em massa
Você pode usar o método create para "salvar" um novo modelo usando uma única instrução PHP. A instância de modelo inserida será devolvida a você pelo método:
``` php
use App\Models\Flight;
 
$flight = Flight::create([
    'name' => 'London to Paris',
]);
```
No entanto, antes de usar o método create, você precisará especificar uma propriedade preenchível ou protegida em sua classe de modelo. Essas propriedades são necessárias porque todos os modelos Eloquent são protegidos contra vulnerabilidades de atribuição em massa por padrão.

Uma vulnerabilidade de atribuição em massa ocorre quando um usuário passa um campo de solicitação HTTP inesperado e esse campo altera uma coluna em seu banco de dados que você não esperava. Por exemplo, um usuário mal-intencionado pode enviar um parâmetro is_admin por meio de uma solicitação HTTP, que é então passada para o método create do seu modelo, permitindo que o usuário se encaminhe para um administrador.

Portanto, para começar, você deve definir quais atributos de modelo deseja tornar atribuíveis em massa. Você pode fazer isso usando a propriedade $fillable no modelo. Por exemplo, vamos tornar o atributo name da massa do nosso modelo de voo atribuível:

``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```
Depois de especificar quais atributos são atribuíveis em massa, você pode usar o método create para inserir um novo registro no banco de dados. O método create retorna a instância de modelo recém-criada:
``` php
$flight = Flight::create(['name' => 'London to Paris']);
```
Se você já tem uma instância de modelo, você pode usar o método fill para preenchê-la com um array de atributos:
``` php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

### Atribuição em Massa e Colunas JSON
Ao atribuir colunas JSON, a chave atribuível em massa de cada coluna deve ser especificada na matriz $fillable do seu modelo. Por segurança, o Laravel não suporta a atualização de atributos JSON aninhados ao usar a propriedade protegida:
``` php
/**
 * The attributes that are mass assignable.
 *
 * @var array
 */
protected $fillable = [
    'options->enabled',
];
```
### Permitindo atribuição em massa
Se você quiser tornar todos os seus atributos atribuíveis em massa, você pode definir a propriedade $guarded do seu modelo como um array vazio. Se você optar por desproteger seu modelo, você deve ter um cuidado especial para sempre criar manualmente os arrays passados ​​para os métodos fill, create e update do Eloquent:
``` php
/**
 * The attributes that aren't mass assignable.
 *
 * @var array
 */
protected $guarded = [];
```
## Upsert
Ocasionalmente, pode ser necessário atualizar um modelo existente ou criar um novo modelo se não existir nenhum modelo correspondente. Assim como o método firstOrCreate, o método updateOrCreate persiste o modelo, portanto, não há necessidade de chamar manualmente o método save.

No exemplo abaixo, se existir um voo com local de partida em Oakland e local de destino em San Diego, suas colunas de preço e desconto serão atualizadas. Se não existir tal voo, será criado um novo voo que possui os atributos resultantes da mesclagem do primeiro array de argumentos com o segundo array de argumentos:
``` php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```
Se você quiser executar vários "upserts" em uma única consulta, use o método upsert. O primeiro argumento do método consiste nos valores a serem inseridos ou atualizados, enquanto o segundo argumento lista as colunas que identificam exclusivamente os registros na tabela associada. O terceiro e último argumento do método é uma matriz das colunas que devem ser atualizadas se já existir um registro correspondente no banco de dados. O método upsert definirá automaticamente os timestamps created_at e updated_at se os timestamps estiverem habilitados no modelo:
``` php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], ['departure', 'destination'], ['price']);
```
> Todos os bancos de dados, exceto o SQL Server, exigem que as colunas no segundo argumento do método upsert tenham um índice "primário" ou "único". Além disso, o driver de banco de dados MySQL ignora o segundo argumento do método upsert e sempre usa os índices "primário" e "único" da tabela para detectar os registros existentes.

## Excluindo Modelos
Para excluir um modelo, você pode chamar o método delete na instância do modelo:
``` php
use App\Models\Flight;
 
$flight = Flight::find(1);
 
$flight->delete();
```
Você pode chamar o método truncate para excluir todos os registros de banco de dados associados ao modelo. A operação de truncar também redefinirá quaisquer IDs de incremento automático na tabela associada ao modelo:
``` php
Flight::truncate();
```
### Excluindo um modelo existente por sua chave primária
No exemplo acima, estamos recuperando o modelo do banco de dados antes de chamar o método delete. No entanto, se você conhece a chave primária do modelo, pode excluir o modelo sem recuperá-lo explicitamente chamando o método destroy. Além de aceitar a chave primária única, o método destroy aceitará várias chaves primárias, uma matriz de chaves primárias ou uma coleção de chaves primárias:
``` php 
Flight::destroy(1);
 
Flight::destroy(1, 2, 3);
 
Flight::destroy([1, 2, 3]);
 
Flight::destroy(collect([1, 2, 3]));
```
> O método destroy carrega cada modelo individualmente e chama o método delete para que os eventos delete e delete sejam despachados adequadamente para cada modelo.

### Excluindo Modelos Usando Consultas
Claro, você pode construir uma consulta Eloquent para excluir todos os modelos que correspondam aos critérios da sua consulta. Neste exemplo, excluiremos todos os voos marcados como inativos. Assim como as atualizações em massa, as exclusões em massa não despacharão eventos de modelo para os modelos que foram excluídos:
``` php
$deleted = Flight::where('active', 0)->delete();
```
> Ao executar uma instrução de exclusão em massa via Eloquent, os eventos de modelo de exclusão e exclusão não serão despachados para os modelos excluídos. Isso ocorre porque os modelos nunca são realmente recuperados ao executar a instrução delete.

## Exclusão Suave
Além de realmente remover registros do seu banco de dados, o Eloquent também pode "excluir suavemente" os modelos. Quando os modelos são excluídos de forma reversível, eles não são realmente removidos do banco de dados. Em vez disso, um atributo delete_at é definido no modelo, indicando a data e hora em que o modelo foi "excluído". Para habilitar exclusões reversíveis para um modelo, adicione o atributo Illuminate\Database\Eloquent\SoftDeletes ao modelo:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
 
class Flight extends Model
{
    use SoftDeletes;
}
```
> O atributo SoftDeletes converterá automaticamente o atributo delete_at para uma instância DateTime / Carbon para você.

Você também deve adicionar a coluna delete_at à sua tabela de banco de dados. O construtor de esquema Laravel contém um método auxiliar para criar esta coluna:
``` php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
 
Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
 
Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
```
Agora, quando você chamar o método delete no modelo, a coluna delete_at será definida para a data e hora atuais. No entanto, o registro do banco de dados do modelo será deixado na tabela. Ao consultar um modelo que usa exclusões reversíveis, os modelos com exclusão reversível serão automaticamente excluídos de todos os resultados da consulta.

Para determinar se uma determinada instância de modelo foi excluída de forma reversível, você pode usar o método trashed:
``` php
if ($flight->trashed()) {
    //
}
```

### Restaurando Modelos Excluídos Suavemente
Às vezes, você pode querer "desfazer a exclusão" de um modelo com exclusão reversível. Para restaurar um modelo com exclusão reversível, você pode chamar o método de restauração em uma instância do modelo. O método de restauração definirá a coluna delete_at do modelo como nula:
``` php
$flight->restore();
```
Você também pode usar o método de restauração em uma consulta para restaurar vários modelos. Novamente, como outras operações "em massa", isso não despachará nenhum evento de modelo para os modelos restaurados:
``` php
Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```
O método de restauração também pode ser usado ao construir consultas de relacionamento:
``` php
$flight->history()->restore();
```
### Exclusão Permanente de Modelos
Às vezes, você pode precisar realmente remover um modelo do seu banco de dados. Você pode usar o método forceDelete para remover permanentemente um modelo de exclusão reversível da tabela do banco de dados:
``` php
$flight->forceDelete();
```
Você também pode usar o método forceDelete ao construir consultas de relacionamento do Eloquent:
``` php
$flight->history()->forceDelete();
```
## Como Consultar Modelos com Exclusão Reversível
### Incluindo Modelos com Exclusão Reversível
Conforme observado acima, os modelos com exclusão reversível serão automaticamente excluídos dos resultados da consulta. No entanto, você pode forçar a inclusão de modelos com exclusão reversível nos resultados de uma consulta chamando o método withTrashed na consulta:
``` php
use App\Models\Flight;
 
$flights = Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```
O método withTrashed também pode ser chamado ao construir uma consulta de relacionamento:
``` php
$flight->history()->withTrashed()->get();
```
### Recuperando Apenas Modelos Excluídos de Forma Reversível
O método onlyTrashed recuperará apenas modelos com exclusão reversível:
``` php
$flights = Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```
## **Modelos de Podas**
Às vezes, você pode querer excluir periodicamente os modelos que não são mais necessários. Para fazer isso, você pode adicionar o traço Illuminate\Database\Eloquent\Prunable ou Illuminate\Database\Eloquent\MassPrunable aos modelos que você gostaria de podar periodicamente. Depois de adicionar uma das características ao modelo, implemente um método que pode ser removido que retorna um construtor de consultas Eloquent que resolve os modelos que não são mais necessários:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;
 
class Flight extends Model
{
    use Prunable;
 
    /**
     * Get the prunable model query.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function prunable()
    {
        return static::where('created_at', '<=', now()->subMonth());
    }
}
```
Ao marcar modelos como Prunable, você também pode definir um método de poda no modelo. Este método será chamado antes que o modelo seja excluído. Esse método pode ser útil para excluir quaisquer recursos adicionais associados ao modelo, como arquivos armazenados, antes que o modelo seja removido permanentemente do banco de dados:
``` php
/**
 * Prepare the model for pruning.
 *
 * @return void
 */
protected function pruning()
{
    //
}
```
Depois de configurar seu modelo editável, você deve agendar o comando model:prune Artisan na classe App\Console\Kernel do seu aplicativo. Você é livre para escolher o intervalo apropriado em que este comando deve ser executado:
``` php
/**
 * Define the application's command schedule.
 *
 * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
 * @return void
 */
protected function schedule(Schedule $schedule)
{
    $schedule->command('model:prune')->daily();
}
```
Nos bastidores, o comando model:prune detectará automaticamente os modelos "Prunable" no diretório app/Models do seu aplicativo. Se seus modelos estiverem em um local diferente, você pode usar a opção --model para especificar os nomes das classes do modelo:
``` php
$schedule->command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```
Se você deseja excluir certos modelos da remoção enquanto remove todos os outros modelos detectados, você pode usar a opção --except:
``` php
$schedule->command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```
Você pode testar sua consulta editável executando o comando model:prune com a opção --pretend. Ao fingir, o comando model:prune simplesmente reportará quantos registros seriam removidos se o comando realmente fosse executado:
``` php
php artisan model:prune --pretend
```
> Os modelos de exclusão suave serão excluídos permanentemente (forceDelete) se corresponderem à consulta editável.

### Poda em Massa
Quando os modelos são marcados com o traço Illuminate\Database\Eloquent\MassPrunable, os modelos são excluídos do banco de dados usando consultas de exclusão em massa. Portanto, o método pruning não será invocado, nem os eventos de modelo delete e delete serão despachados. Isso ocorre porque os modelos nunca são realmente recuperados antes da exclusão, tornando o processo de poda muito mais eficiente:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\MassPrunable;
 
class Flight extends Model
{
    use MassPrunable;
 
    /**
     * Get the prunable model query.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function prunable()
    {
        return static::where('created_at', '<=', now()->subMonth());
    }
}
```
## Modelos de Replicação
Você pode criar uma cópia não salva de uma instância de modelo existente usando o método de replicação. Este método é particularmente útil quando você tem instâncias de modelo que compartilham muitos dos mesmos atributos:
``` php
use App\Models\Address;
 
$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);
 
$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);
 
$billing->save();
```
Para excluir um ou mais atributos de serem replicados para o novo modelo, você pode passar um array para o método de replicação:
``` php
$flight = Flight::create([
    'destination' => 'LAX',
    'origin' => 'LHR',
    'last_flown' => '2020-03-04 11:00:00',
    'last_pilot_id' => 747,
]);
 
$flight = $flight->replicate([
    'last_flown',
    'last_pilot_id'
]);
```
## Escopos de consulta
### Escopos Globais
Os escopos globais permitem adicionar restrições a todas as consultas de um determinado modelo. A própria funcionalidade de exclusão reversível do Laravel utiliza escopos globais para recuperar apenas modelos "não excluídos" do banco de dados. Escrever seus próprios escopos globais pode fornecer uma maneira conveniente e fácil de garantir que cada consulta para um determinado modelo receba certas restrições.
### Escrevendo Escopos Globais
Escrever um escopo global é simples. Primeiro, defina uma classe que implemente a interface Illuminate\Database\Eloquent\Scope. O Laravel não tem um local convencional onde você deve colocar classes de escopo, então você está livre para colocar esta classe em qualquer diretório que desejar.

A interface Scope requer que você implemente um método: apply. O método apply pode adicionar restrições where ou outros tipos de cláusulas à consulta conforme necessário:

``` php
<?php
 
namespace App\Scopes;
 
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;
 
class AncientScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('created_at', '<', now()->subYears(2000));
    }
}
```
> Se seu escopo global estiver adicionando colunas à cláusula select da consulta, você deverá usar o método addSelect em vez de select. Isso evitará a substituição não intencional da cláusula select existente da consulta.

### Aplicando Escopos Globais
Para atribuir um escopo global a um modelo, você deve substituir o método inicializado do modelo e invocar o método addGlobalScope do modelo. O método addGlobalScope aceita uma instância do seu escopo como seu único argumento:
``` php
<?php
 
namespace App\Models;
 
use App\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        static::addGlobalScope(new AncientScope);
    }
}
```
Após adicionar o escopo do exemplo acima ao modelo App\Models\User, uma chamada ao método User::all() executará a seguinte consulta SQL:
``` php
select * from `users` where `created_at` < 0021-02-18 00:00:00
```
### Escopos Globais Anônimos
O Eloquent também permite definir escopos globais usando closures, o que é particularmente útil para escopos simples que não garantem uma classe separada própria. Ao definir um escopo global usando um encerramento, você deve fornecer um nome de escopo de sua própria escolha como o primeiro argumento para o método addGlobalScope:
``` php 
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        static::addGlobalScope('ancient', function (Builder $builder) {
            $builder->where('created_at', '<', now()->subYears(2000));
        });
    }
}
```
### Removendo Escopos Globais
Se você deseja remover um escopo global para uma determinada consulta, você pode usar o método withoutGlobalScope. Este método aceita o nome da classe do escopo global como seu único argumento:
``` php
User::withoutGlobalScope(AncientScope::class)->get()
```
Ou, se você definiu o escopo global usando um closure, você deve passar o nome da string que você atribuiu ao escopo global:
``` php
User::withoutGlobalScope('ancient')->get();
```
Se você deseja remover vários ou até mesmo todos os escopos globais da consulta, você pode usar o método withoutGlobalScopes:
``` php
// Remove all of the global scopes...
User::withoutGlobalScopes()->get();
 
// Remove some of the global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```
## Escopos locais
Os escopos locais permitem definir conjuntos comuns de restrições de consulta que podem ser reutilizados facilmente em todo o aplicativo. Por exemplo, pode ser necessário recuperar com frequência todos os usuários considerados "populares". Para definir um escopo, prefixe um método de modelo Eloquent com escopo.

Os escopos devem sempre retornar a mesma instância do construtor de consultas ou void:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    /**
     * Scope a query to only include popular users.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }
 
    /**
     * Scope a query to only include active users.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return void
     */
    public function scopeActive($query)
    {
        $query->where('active', 1);
    }
}
```
### Utilizando um Escopo Local
Uma vez que o escopo foi definido, você pode chamar os métodos de escopo ao consultar o modelo. No entanto, você não deve incluir o prefixo do escopo ao chamar o método. Você pode até encadear chamadas para vários escopos:
``` php
use App\Models\User;
 
$users = User::popular()->active()->orderBy('created_at')->get();
```
A combinação de vários escopos de modelo do Eloquent por meio de um operador de consulta ou pode exigir o uso de closures para obter o agrupamento lógico correto:
``` php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```
No entanto, como isso pode ser complicado, o Laravel fornece um método orWhere de "ordem superior" que permite encadear escopos fluentemente sem o uso de closures:
``` php
$users = App\Models\User::popular()->orWhere->active()->get();
```
### Escopos Dinâmicos
Às vezes você pode querer definir um escopo que aceite parâmetros. Para começar, basta adicionar seus parâmetros adicionais à assinatura do seu método de escopo. Os parâmetros de escopo devem ser definidos após o parâmetro $query:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @param  mixed  $type
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
```
Uma vez que os argumentos esperados tenham sido adicionados à assinatura do seu método de escopo, você pode passar os argumentos ao chamar o escopo:
``` php
$users = User::ofType('admin')->get();
```
## Comparação de Modelos
Às vezes, você pode precisar determinar se dois modelos são "iguais" ou não. Os métodos is e isNot podem ser usados ​​para verificar rapidamente se dois modelos possuem a mesma chave primária, tabela e conexão com o banco de dados ou não:
``` php
if ($post->is($anotherPost)) {
    //
}
 
if ($post->isNot($anotherPost)) {
    //
}
```
Os métodos is e isNot também estão disponíveis ao usar os relacionamentos belongsTo, hasOne, morphTo e morphOne. Esse método é particularmente útil quando você deseja comparar um modelo relacionado sem emitir uma consulta para recuperar esse modelo:
``` php
if ($post->author()->is($user)) {
    //
}
```
## Eventos
> Quer transmitir seus eventos do Eloquent diretamente para o seu aplicativo do lado do cliente? Confira o modelo de transmissão de eventos do Laravel.

Os modelos Eloquent despacham vários eventos, permitindo que você se conecte aos seguintes momentos no ciclo de vida de um modelo: recuperado, criado, criado, atualizado, atualizado, salvo, salvo, excluído, excluído, descartado, forçado a excluir, restaurado, restaurado e replicado.

O evento recuperado será despachado quando um modelo existente for recuperado do banco de dados. Quando um novo modelo é salvo pela primeira vez, os eventos de criação e criação serão despachados. Os eventos de atualização/atualização serão despachados quando um modelo existente for modificado e o método de salvamento for chamado. Os eventos de salvamento/salvamento serão despachados quando um modelo for criado ou atualizado - mesmo que os atributos do modelo não tenham sido alterados. Os nomes de eventos que terminam com -ing são despachados antes que qualquer alteração no modelo seja persistida, enquanto os eventos que terminam com -ed são despachados após as alterações no modelo persistirem.

Para começar a ouvir eventos de modelo, defina uma propriedade $dispatchesEvents em seu modelo Eloquent. Esta propriedade mapeia vários pontos do ciclo de vida do modelo Eloquent para suas próprias classes de eventos. Cada classe de evento de modelo deve esperar receber uma instância do modelo afetado por meio de seu construtor:
``` php
<?php
 
namespace App\Models;
 
use App\Events\UserDeleted;
use App\Events\UserSaved;
use Illuminate\Foundation\Auth\User as Authenticatable;
 
class User extends Authenticatable
{
    use Notifiable;
 
    /**
     * The event map for the model.
     *
     * @var array
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```
Depois de definir e mapear seus eventos do Eloquent, você pode usar ouvintes de eventos para lidar com os eventos.

> Ao emitir uma atualização em massa ou uma consulta de exclusão via Eloquent, os eventos de modelo salvos, atualizados, excluídos e excluídos não serão enviados para os modelos afetados. Isso ocorre porque os modelos nunca são realmente recuperados ao realizar atualizações ou exclusões em massa.

## Usando Fechamentos
Em vez de usar classes de eventos personalizadas, você pode registrar closures que são executados quando vários eventos de modelo são despachados. Normalmente, você deve registrar esses fechamentos no método inicializado do seu modelo:
``` php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        static::created(function ($user) {
            //
        });
    }
}
```
Se necessário, você pode utilizar ouvintes de eventos anônimos enfileirados ao registrar eventos de modelo. Isso instruirá o Laravel a executar o ouvinte de eventos do modelo em segundo plano usando a fila do seu aplicativo:
``` php
use function Illuminate\Events\queueable;
 
static::created(queueable(function ($user) {
    //
}));
```
## Observadores
### Definindo Observadores
Se você estiver ouvindo muitos eventos em um determinado modelo, poderá usar observadores para agrupar todos os seus ouvintes em uma única classe. As classes de observadores têm nomes de métodos que refletem os eventos do Eloquent que você deseja ouvir. Cada um desses métodos recebe o modelo afetado como seu único argumento. O comando make:observer Artisan é a maneira mais fácil de criar uma nova classe de observador:
``` php
php artisan make:observer UserObserver --model=User
```
Este comando colocará o novo observador em seu diretório App/Observers. Se este diretório não existir, Artisan irá criá-lo para você. Seu novo observador terá a seguinte aparência:
``` php
<?php
 
namespace App\Observers;
 
use App\Models\User;
 
class UserObserver
{
    /**
     * Handle the User "created" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }
 
    /**
     * Handle the User "updated" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function updated(User $user)
    {
        //
    }
 
    /**
     * Handle the User "deleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function deleted(User $user)
    {
        //
    }
 
    /**
     * Handle the User "restored" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function restored(User $user)
    {
        //
    }
 
    /**
     * Handle the User "forceDeleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function forceDeleted(User $user)
    {
        //
    }
}
```
Para registrar um observador, você precisa chamar o método observe no modelo que deseja observar. Você pode registrar observadores no método de inicialização do provedor de serviços App\Providers\EventServiceProvider do seu aplicativo:
``` php
use App\Models\User;
use App\Observers\UserObserver;
 
/**
 * Register any events for your application.
 *
 * @return void
 */
public function boot()
{
    User::observe(UserObserver::class);
}
```
Como alternativa, você pode listar seus observadores em uma propriedade $observers da classe App\Providers\EventServiceProvider de seus aplicativos:
``` php
use App\Models\User;
use App\Observers\UserObserver;
 
/**
 * The model observers for your application.
 *
 * @var array
 */
protected $observers = [
    User::class => [UserObserver::class],
];
```
> Há eventos adicionais que um observador pode ouvir, como salvar e recuperar. Esses eventos são descritos na documentação de eventos.

## Observadores e Transações de Banco de Dados
Quando os modelos estão sendo criados em uma transação de banco de dados, você pode instruir um observador a executar apenas seus manipuladores de eventos depois que a transação de banco de dados for confirmada. Você pode fazer isso definindo uma propriedade $afterCommit no observador. Se uma transação de banco de dados não estiver em andamento, os manipuladores de eventos serão executados imediatamente:
``` php
<?php
 
namespace App\Observers;
 
use App\Models\User;
 
class UserObserver
{
    /**
     * Handle events after all transactions are committed.
     *
     * @var bool
     */
    public $afterCommit = true;
 
    /**
     * Handle the User "created" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }
}
```
## Eventos de Silenciamento
Ocasionalmente, você pode precisar "silenciar" temporariamente todos os eventos disparados por um modelo. Você pode conseguir isso usando o método withoutEvents. O método withoutEvents aceita um encerramento como seu único argumento. Qualquer código executado dentro deste fechamento não despachará eventos de modelo, e qualquer valor retornado pelo fechamento será retornado pelo método withoutEvents:
``` php
use App\Models\User;
 
$user = User::withoutEvents(function () use () {
    User::findOrFail(1)->delete();
 
    return User::find(2);
});
```
## Salvando um Único Modelo Sem Eventos
Às vezes você pode querer "salvar" um determinado modelo sem despachar nenhum evento. Você pode fazer isso usando o método saveQuietly:
``` php
$user = User::findOrFail(1);
 
$user->name = 'Victoria Faith';
 
$user->saveQuietly();
```
Você também pode "atualizar", "excluir", "excluir por software", "restaurar" e "replicar" um determinado modelo sem despachar nenhum evento:
```php
$user->deleteQuietly();
 
$user->restoreQuietly();
```
