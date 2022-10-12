# Armazenamento de arquivo
## Introdução
## Configuração
 #Driver local
 
 #Disco público
 
 #Pré-requisitos do driver
 
 #Sistemas de arquivos compatíveis com Amazon S3
## Como obter instâncias de disco 
#Discos sob demanda

## Recuperando arquivos

#Baixando arquivos

#URLs de arquivo

#Metadados de arquivo
## Armazenando arquivos
#Anexando e anexando a arquivos

#Copiando e movendo arquivos
  
#Transmissão 

#Uploads de arquivos

#Visibilidade do arquivo
## Excluindo arquivos
## Diretórios
## Sistemas de arquivos personalizados

# Introdução
O Laravel fornece uma poderosa abstração de sistema de arquivos graças ao maravilhoso pacote PHP Flysystem de Frank de Jonge. A integração do Laravel Flysystem fornece drivers simples para trabalhar com sistemas de arquivos locais, SFTP e Amazon S3. Melhor ainda, é incrivelmente simples alternar entre essas opções de armazenamento entre sua máquina de desenvolvimento local e o servidor de produção, pois a API permanece a mesma para cada sistema.

# Configuração
O arquivo de configuração do sistema de arquivos do Laravel está localizado em config/filesystems.php. Dentro deste arquivo, você pode configurar todos os "discos" do seu sistema de arquivos. Cada disco representa um determinado driver de armazenamento e local de armazenamento. As configurações de exemplo para cada driver compatível estão incluídas no arquivo de configuração para que você possa modificar a configuração para refletir suas preferências e credenciais de armazenamento.

O driver local interage com arquivos armazenados localmente no servidor que executa o aplicativo Laravel enquanto o driver s3 é usado para gravar no serviço de armazenamento em nuvem S3 da Amazon.


Você pode configurar quantos discos quiser e pode até ter vários discos que usam o mesmo driver.

# Driver local
Ao usar o driver local, todas as operações de arquivo são relativas ao diretório raiz definido no arquivo de configuração do sistema de arquivos. Por padrão, esse valor é definido para o diretório storage/app. Portanto, o método a seguir gravaria em storage/app/example.txt:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::disk('local')->put('example.txt', 'Contents');
```


# Disco público
O disco público incluído no arquivo de configuração do sistema de arquivos do seu aplicativo destina-se a arquivos que serão acessíveis publicamente. Por padrão, o disco público usa o driver local e armazena seus arquivos em storage/app/public.

Para tornar esses arquivos acessíveis da web, você deve criar um link simbólico de public/storage para storage/app/public. A utilização dessa convenção de pasta manterá seus arquivos acessíveis publicamente em um diretório que pode ser facilmente compartilhado entre implantações ao usar sistemas de implantação com zero tempo de inatividade, como o Envoyer.

Para criar o link simbólico, você pode usar o comando storage:link Artisan:

```php
php artisan storage:link
```

Depois que um arquivo foi armazenado e o link simbólico foi criado, você pode criar uma URL para os arquivos usando o assistente de ativos:

```php
echo asset('storage/file.txt');
```



Você pode configurar links simbólicos adicionais em seu arquivo de configuração do sistema de arquivos. Cada um dos links configurados será criado quando você executar o comando storage:link:

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```


# Pré-requisitos do Driver
### Configuração do driver S3

Antes de usar o driver S3, você precisará instalar o pacote Flysystem S3 através do gerenciador de pacotes Composer:

```php
composer require league/flysystem-aws-s3-v3 "^3.0"
```


As informações de configuração do driver S3 estão localizadas no arquivo de configuração config/filesystems.php. Este arquivo contém uma matriz de configuração de exemplo para um driver S3. Você pode modificar esse array com sua própria configuração e credenciais do S3. Por conveniência, essas variáveis de ambiente correspondem à convenção de nomenclatura usada pela AWS CLI.

# Configuração do Driver FTP
Antes de usar o driver FTP, você precisará instalar o pacote FTP Flysystem através do gerenciador de pacotes Composer:

```php
composer require league/flysystem-ftp "^3.0"
```


As integrações do Flysystem do Laravel funcionam muito bem com FTP; no entanto, uma configuração de amostra não está incluída no arquivo de configuração filesystems.php padrão da estrutura. Se você precisar configurar um sistema de arquivos FTP, você pode usar o exemplo de configuração abaixo:

```php
// 'ftp' => [
 //    'driver' => 'ftp',
 //    'host' => env('FTP_HOST'),
 //    'username' => env('FTP_USERNAME'),
 //   'password' => env('FTP_PASSWORD'),
    // Configurações de FTP opcionais...
     // 'porta' => env('FTP_PORT', 21),
     // 'raiz' => env('FTP_ROOT'),
     // 'passivo' => verdadeiro,
     // 'ssl' => verdadeiro,
     // 'tempo limite' => 30,
],
```


# Configuração do driver SFTP
Antes de usar o driver SFTP, você precisará instalar o pacote Flysystem SFTP através do gerenciador de pacotes Composer:

```php
composer require league/flysystem-sftp-v3 "^3.0"
```


As integrações do Flysystem do Laravel funcionam muito bem com SFTP; no entanto, uma configuração de amostra não está incluída no arquivo de configuração filesystems.php padrão da estrutura. Se você precisar configurar um sistema de arquivos SFTP, você pode usar o exemplo de configuração abaixo:

```php
// 'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'), 
    
    // Configurações para autenticação básica...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),
 
    // Configurações para autenticação baseada em chave SSH com senha de criptografia...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'password' => env('SFTP_PASSWORD'),
 
    // Configurações de SFTP opcionais...
     // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
     // 'maxTries' => 4,
     // 'senha' => env('SFTP_PASSPHRASE'),
     // 'porta' => env('SFTP_PORT', 22),
     // 'raiz' => env('SFTP_ROOT', ''),
     // 'tempo limite' => 30,
     // 'useAgent' => true,
],
```


# Sistemas de arquivos compatíveis com Amazon S3
Por padrão, o arquivo de configuração dos sistemas de arquivos do seu aplicativo contém uma configuração de disco para o disco s3. Além de usar esse disco para interagir com o Amazon S3, você pode usá-lo para interagir com qualquer serviço de armazenamento de arquivos compatível com S3, como MinIO ou DigitalOcean Spaces.

Normalmente, depois de atualizar as credenciais do disco para corresponder às credenciais do serviço que você planeja usar, você só precisa atualizar o valor da opção de configuração do terminal. O valor dessa opção geralmente é definido por meio da variável de ambiente AWS_ENDPOINT:

```php
'endpoint' => env('AWS_ENDPOINT', 'https://minio:9000'),
```


# Como obter instâncias de disco
A fachada de armazenamento pode ser usada para interagir com qualquer um dos discos configurados. Por exemplo, você pode usar o método put na fachada para armazenar um avatar no disco padrão. Se você chamar métodos na fachada de armazenamento sem primeiro chamar o método de disco, o método será automaticamente passado para o disco padrão:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::put('avatars/1', $content)
```
;

Se seu aplicativo interage com vários discos, você pode usar o método disk na fachada de armazenamento para trabalhar com arquivos em um disco específico:

```php
Storage::disk('s3')->put('avatars/1', $content);
```


# Discos sob demanda
Às vezes você pode querer criar um disco em tempo de execução usando uma determinada configuração sem que essa configuração esteja realmente presente no arquivo de configuração do sistema de arquivos do seu aplicativo. Para fazer isso, você pode passar um array de configuração para o método de construção da fachada de armazenamento:

```php
use Illuminate\Support\Facades\Storage;
 
$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);
 
$disk->put('image.jpg', $content);
```


# Recuperando arquivos
O método get pode ser usado para recuperar o conteúdo de um arquivo. O conteúdo da string bruta do arquivo será retornado pelo método. Lembre-se, todos os caminhos de arquivo devem ser especificados em relação ao local "raiz" do disco:

```php
$contents = Storage::get('file.jpg');
```


O método existe pode ser usado para determinar se existe um arquivo no disco:

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```


O método ausente pode ser usado para determinar se um arquivo está ausente do disco:

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```


# Baixando arquivos
O método de download pode ser usado para gerar uma resposta que força o navegador do usuário a baixar o arquivo no caminho fornecido. O método download aceita um nome de arquivo como segundo argumento para o método, que determinará o nome do arquivo que é visto pelo usuário que está baixando o arquivo. Finalmente, você pode passar um array de cabeçalhos HTTP como terceiro argumento para o método:

```php
return Storage::download('file.jpg');
 
return Storage::download('file.jpg', $name, $headers);
```


URLs de arquivo
Você pode usar o método url para obter a URL de um determinado arquivo. Se você estiver usando o driver local, isso normalmente apenas adicionará /storage ao caminho fornecido e retornará uma URL relativa ao arquivo. Se você estiver usando o driver s3, a URL remota totalmente qualificada será retornada:

```php
use Illuminate\Support\Facades\Storage;
 
$url = Storage::url('file.jpg');
```

Ao usar o driver local, todos os arquivos que devem ser acessíveis publicamente devem ser colocados no diretório storage/app/public. Além disso, você deve criar um link simbólico em public/storage que aponte para o diretório storage/app/public.

Ao usar o driver local, o valor de retorno de url não é codificado em URL. Por esse motivo, recomendamos sempre armazenar seus arquivos usando nomes que criarão URLs válidos.

# URLs temporários
Usando o método temporárioUrl, você pode criar URLs temporários para arquivos armazenados usando o driver s3. Este método aceita um caminho e uma instância DateTime especificando quando a URL deve expirar:

```php
use Illuminate\Support\Facades\Storage;
 
$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);
```


Se você precisar especificar parâmetros de solicitação adicionais do S3, poderá passar a matriz de parâmetros de solicitação como o terceiro argumento para o método optionalUrl:

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->addMinutes(5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```


Se precisar personalizar como as URLs temporárias são criadas para um disco de armazenamento específico, você pode usar o método buildTemporaryUrlsUsing. Por exemplo, isso pode ser útil se você tiver um controlador que permita baixar arquivos armazenados por meio de um disco que normalmente não oferece suporte a URLs temporários. Normalmente, esse método deve ser chamado a partir do método de inicialização de um provedor de serviços:

 ```php
<?php
 
namespace App\Providers;
 
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap qualquer serviço de aplicativo.
     *
     * @return void
     */
    public function boot()
    {
        Storage::disk('local')->buildTemporaryUrlsUsing(function ($path, $expiration, $options) {
            return URL::temporarySignedRoute(
                'files.download',
                $expiration,
                array_merge($options, ['path' => $path])
            );
        });
    }
}
```


# Metadados de arquivo
Além de ler e gravar arquivos, o Laravel também pode fornecer informações sobre os próprios arquivos. Por exemplo, o método size pode ser usado para obter o tamanho de um arquivo em bytes:

```php
use Illuminate\Support\Facades\Storage;
 
$size = Storage::size('file.jpg');

O método lastModified retorna o timestamp UNIX da última vez que o arquivo foi modificado:

$time = Storage::lastModified('file.jpg');
```


Você pode usar o método path para obter o caminho para um determinado arquivo. Se você estiver usando o driver local, isso retornará o caminho absoluto para o arquivo. Se você estiver usando o driver s3, este método retornará o caminho relativo para o arquivo no bucket do S3:

```php
use Illuminate\Support\Facades\Storage;
 
$path = Storage::path('file.jpg');
```


# Armazenando arquivos
O método put pode ser usado para armazenar o conteúdo do arquivo em um disco. Você também pode passar um recurso PHP para o método put, que usará o suporte de stream subjacente do Flysystem. Lembre-se, todos os caminhos de arquivo devem ser especificados em relação ao local "raiz" configurado para o disco:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::put('file.jpg', $contents);
 
Storage::put('file.jpg', $resource);
```


# Gravações com falha
Se o método put (ou outras operações de "gravação") não puder gravar o arquivo no disco, false será retornado:

```php
if (! Storage::put('file.jpg', $contents)) {
    // O arquivo não pôde ser gravado no disco...
}
```


Se desejar, você pode definir a opção throw no array de configuração do disco do seu sistema de arquivos. Quando esta opção é definida como verdadeira, os métodos "write", como put, lançarão uma instância de League\Flysystem\UnableToWriteFile quando as operações de gravação falharem:

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

# Anexando e anexando a arquivos
Os métodos prepend e append permitem gravar no início ou no final de um arquivo:

```php
Storage::prepend('file.log', 'Prepended Text');
 
Storage::append('file.log', 'Appended Text');
```


# Copiando e movendo arquivos
O método copy pode ser usado para copiar um arquivo existente para um novo local no disco, enquanto o método move pode ser usado para renomear ou mover um arquivo existente para um novo local:

```php
Storage::copy('old/file.jpg', 'new/file.jpg');
 
Storage::move('old/file.jpg', 'new/file.jpg');
```


# Transmissão automática
O streaming de arquivos para armazenamento oferece um uso de memória significativamente reduzido. Se você deseja que o Laravel gerencie automaticamente o streaming de um determinado arquivo para o seu local de armazenamento, você pode usar o método putFile ou putFileAs. Este método aceita uma instância Illuminate\Http\File ou Illuminate\Http\UploadedFile e transmitirá automaticamente o arquivo para o local desejado:

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;
 
// Gera automaticamente um ID exclusivo para o nome do arquivo...
$path = Storage::putFile('photos', new File('/path/to/photo'));
 
// Especifique manualmente um nome de arquivo...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```


Há algumas coisas importantes a serem observadas sobre o método putFile. Observe que especificamos apenas um nome de diretório e não um nome de arquivo. Por padrão, o método putFile gerará um ID exclusivo para servir como nome do arquivo. A extensão do arquivo será determinada examinando o tipo MIME do arquivo. O caminho para o arquivo será retornado pelo método putFile para que você possa armazenar o caminho, incluindo o nome do arquivo gerado, em seu banco de dados.

Os métodos putFile e putFileAs também aceitam um argumento para especificar a "visibilidade" do arquivo armazenado. Isso é particularmente útil se você estiver armazenando o arquivo em um disco na nuvem, como o Amazon S3, e quiser que o arquivo seja acessível publicamente por meio de URLs gerados:

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```


# Uploads de arquivos
Em aplicativos da Web, um dos casos de uso mais comuns para armazenar arquivos é armazenar arquivos carregados pelo usuário, como fotos e documentos. O Laravel facilita muito o armazenamento de arquivos carregados usando o método store em uma instância de arquivo carregada. Chame o método store com o caminho no qual você deseja armazenar o arquivo carregado:

```php
<?php
 
namespace App\Http\Controllers;
 
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
 
class UserAvatarController extends Controller
{
    /**
     * Atualize o avatar para o usuário.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars');
 
        return $path;
    }
}
```


Há algumas coisas importantes a serem observadas sobre este exemplo. Observe que especificamos apenas um nome de diretório, não um nome de arquivo. Por padrão, o método store gerará um ID exclusivo para servir como o nome do arquivo. A extensão do arquivo será determinada examinando o tipo MIME do arquivo. O caminho para o arquivo será retornado pelo método store para que você possa armazenar o caminho, incluindo o nome do arquivo gerado, em seu banco de dados.

Você também pode chamar o método putFile na fachada Storage para realizar a mesma operação de armazenamento de arquivos do exemplo acima:

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```


# Especificando um nome de arquivo
Se você não quiser que um nome de arquivo seja atribuído automaticamente ao seu arquivo armazenado, você pode usar o método storeAs, que recebe o caminho, o nome do arquivo e o disco (opcional) como seus argumentos:

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```


Você também pode usar o método putFileAs na fachada de armazenamento, que realizará a mesma operação de armazenamento de arquivos do exemplo acima:

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```


Caracteres unicode não imprimíveis e inválidos serão automaticamente removidos dos caminhos de arquivo. Portanto, você pode querer limpar seus caminhos de arquivo antes de passá-los para os métodos de armazenamento de arquivos do Laravel. Os caminhos de arquivo são normalizados usando o método League\Flysystem\WhitespacePathNormalizer::normalizePath.

# Especificando um disco
Por padrão, o método de armazenamento deste arquivo carregado usará seu disco padrão. Se você quiser especificar outro disco, passe o nome do disco como segundo argumento para o método store:

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```


Se estiver usando o método storeAs, você pode passar o nome do disco como terceiro argumento para o método:

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```


Outras informações do arquivo enviado
Se você deseja obter o nome original e a extensão do arquivo carregado, pode fazê-lo usando os métodos getClientOriginalName e getClientOriginalExtension:

```php
$file = $request->file('avatar');
 
$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```


No entanto, lembre-se de que os métodos getClientOriginalName e getClientOriginalExtension são considerados inseguros, pois o nome e a extensão do arquivo podem ser adulterados por um usuário mal-intencionado. Por esse motivo, você normalmente deve preferir os métodos hashName e extension para obter um nome e uma extensão para o upload de arquivo fornecido:

```php
$file = $request->file('avatar');
 
$name = $file->hashName(); // Gera um nome único e aleatório...
$extension = $file->extension(); // Determina a extensão do arquivo com base no tipo MIME do arquivo...
```


# Visibilidade do arquivo
Na integração do Flysystem do Laravel, "visibilidade" é uma abstração de permissões de arquivo em várias plataformas. Os arquivos podem ser declarados públicos ou privados. Quando um arquivo é declarado público, você está indicando que o arquivo geralmente deve ser acessível a outras pessoas. Por exemplo, ao usar o driver S3, você pode recuperar URLs de arquivos públicos.

Você pode definir a visibilidade ao escrever o arquivo através do método put:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::put('file.jpg', $contents, 'public');
```


Se o arquivo já foi armazenado, sua visibilidade pode ser recuperada e definida por meio dos métodos getVisibility e setVisibility:

```php
$visibility = Storage::getVisibility('file.jpg');
 
Storage::setVisibility('file.jpg', 'public');
```


Ao interagir com arquivos carregados, você pode usar os métodos storePublicly e storePubliclyAs para armazenar o arquivo carregado com visibilidade pública:

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');
 
$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```


# Arquivos locais e visibilidade
Ao usar o driver local, a visibilidade pública se traduz em 0755 permissões para diretórios e 0644 permissões para arquivos. Você pode modificar os mapeamentos de permissões no arquivo de configuração dos sistemas de arquivos do seu aplicativo:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
],
```


# Excluindo arquivos
O método delete aceita um único nome de arquivo ou uma matriz de arquivos a serem excluídos:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::delete('file.jpg');
 
Storage::delete(['file.jpg', 'file2.jpg']);
```


Se necessário, você pode especificar o disco do qual o arquivo deve ser excluído:

```php
use Illuminate\Support\Facades\Storage;
 
Storage::disk('s3')->delete('path/file.jpg');
```


# Diretórios
# Obter todos os arquivos em um diretório
O método files retorna uma matriz de todos os arquivos em um determinado diretório. Se você deseja recuperar uma lista de todos os arquivos em um determinado diretório, incluindo todos os subdiretórios, você pode usar o método allFiles:

```php
use Illuminate\Support\Facades\Storage;
 
$files = Storage::files($directory);
 
$files = Storage::allFiles($directory);
```


# Obter todos os diretórios dentro de um diretório
O método directorys retorna uma matriz de todos os diretórios dentro de um determinado diretório. Além disso, você pode usar o método allDirectories para obter uma lista de todos os diretórios dentro de um determinado diretório e todos os seus subdiretórios:

```php
$directories = Storage::directories($directory);
 
$directories = Storage::allDirectories($directory);
```


# Criar um diretório
O método makeDirectory criará o diretório fornecido, incluindo quaisquer subdiretórios necessários:

```php
Storage::makeDirectory($directory);
```


# Excluir um diretório
Finalmente, o método deleteDirectory pode ser usado para remover um diretório e todos os seus arquivos:

```php
Storage::deleteDirectory($directory);
```


# Sistemas de arquivos personalizados
A integração do Flysystem do Laravel fornece suporte para vários "drivers" prontos para uso; no entanto, o Flysystem não se limita a eles e possui adaptadores para muitos outros sistemas de armazenamento. Você pode criar um driver personalizado se quiser usar um desses adaptadores adicionais em seu aplicativo Laravel.

Para definir um sistema de arquivos personalizado, você precisará de um adaptador Flysystem. Vamos adicionar um adaptador do Dropbox mantido pela comunidade ao nosso projeto:

```php
composer require spatie/flysystem-dropbox
```


Em seguida, você pode registrar o driver no método de inicialização de um dos provedores de serviço do seu aplicativo. Para fazer isso, você deve usar o método extend da fachada Storage:

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * Registre qualquer serviço de aplicativo.
     *
     * @return void
     */
    public function register()
    {
        //
    }
 
    /**
     * Bootstrap qualquer serviço de aplicativo.
     *
     * @return void
     */
    public function boot()
    {
        Storage::extend('dropbox', function ($app, $config) {
            $adapter = new DropboxAdapter(new DropboxClient(
                $config['authorization_token']
            ));
 
            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```


O primeiro argumento do método extend é o nome do driver e o segundo é um closure que recebe as variáveis $app e $config. O encerramento deve retornar uma instância de Illuminate\Filesystem\FilesystemAdapter. A variável $config contém os valores definidos em config/filesystems.php para o disco especificado.

Depois de criar e registrar o provedor de serviços da extensão, você pode usar o driver da caixa de depósito em seu arquivo de configuração config/filesystems.php.