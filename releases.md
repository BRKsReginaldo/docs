# Notas de Lançamento

- [Esquema de Versionamento](#versioning-scheme)
- [Politica de Suporte](#support-policy)
- [Laravel 5.8](#laravel-5.8)

<a name="versioning-scheme"></a>
## Esquema de Versionamento

O esquema de versionamento do laravel mantém a seguinte convenção: `paradigma.maiores.menores`. Lançamentos maiores do framework acontecem a cada seis meses (Fevereiro e Agosto), enquanto lançamentos menores podem ser feitos até semanalmente. Lançamentos menores **nunca** devem conter mudanças drásticas, que podem quebrar alguma coisa.

Quando estiver referenciando o framework laravel ou um de seus componentes, você deve sempre usar um padrão de versão como `5.8.*`, já que lançamentos maiores do laravel incluem mudanças drásticas que podem quebrar alguma coisa. No entanto, nós tentamos ao máximo garantir que você possa atualizar para uma nova versão em um dia ou menos.

Lançamentos de mudança de paradigma são separados por muitos anos e representam mudanças fundamentais na arquitetura e convenções do framework. Atualmente, não há um novo paradigma sendo desenvolvido.


<a name="support-policy"></a>
## Politica de Suporte

Para versões LTS, como Laravel 5.5, correções de bugs são fornecidas durante 2 anos, e correções de segurança são fornecidos durante 3 anos. Estas versões possuem o maior tempo de suporte e manutenção. Para outras versão (não LTS), correções de bugs são fornecidos por até 6 meses e correções de segurança são fornecidos durante 1 ano. Para todas as outras bibliotecas adicionais, incluindo Lumen, somente as ultimas versões recebem correções de bugs.

| Versão | Lançamento | Correções de Bugs Até | Correções de Segurança Até |
| --- | --- | --- | --- |
| 5.0 | 4 de Fevereiro de 2015 | 4 de Agosto de 2015 | 4 de Fevereiro de 2016 |
| 5.1 (LTS) | 9 de Junho de 2015 | 9 de Junho de 2017 | 9 de Junho de 2018 |
| 5.2 | 21 de Dezembro de 2015 | 21 de Junho de 2016 | 21 de Dezembro de 2016 |
| 5.3 | 23 de Agosto de 2016 | 23 de Fevereiro de 2017 | 23 de Agosto de 2017 |
| 5.4 | 24 de Janeiro de 2017 | 24 de Julho de 2017 | 24 de Janeiro de 2018 |
| 5.5 (LTS) | 30 de Agosto de 2017 | 30 de Agosto de 2019 | 30 de Agosto de 2020 |
| 5.6 | 7 de Fevereiro de 2018 | 7 de Agosto de 2018 | 7 de Fevereiro de 2019 |
| 5.7 | 4 de Setembro de 2018 | 4 de March de 2019 | 4 de Setembro de 2019 |
| 5.8 | 26 de Fevereiro de 2019 | 26 de Agosto de 2019 | 26 de Fevereiro de 2020 |

<a name="laravel-5.8"></a>
## Laravel 5.8

Laravel 5.8 continua as melhorias feitas no Laravel 5.7 introduzindo relacionamentos Eloquent do tipo hasOneThrough (possui um através de), validação de email melhoradas, registro automatico de politicas de autorização baseado em convenções, drivers de cache e sessões utilizando DynamoDB, configurações de fuso horário do agendador de tarefas melhoradas, suporte para utilização de multiplos guards de autenticação em canais de transmissão, drivers de cache agora seguem o padrão PSR-16, melhorias no comando `artisan serve`, suporte a PHPUnit 8.0, suporte a Carbon 2.0, suporte a Pheanstalk 4.0, e uma variedade de outras correções de bugs e melhorias de usabilidade. 

### Relacionamento do Eloquent `HasOneThrough` (possui um através de)

O Eloquent agora possui suporte para o tipo de relacionamento `hasOneThrough` (possui um através de). Por exemplo, imagine que um model Supplier (fornecedor) `hasOne` (possui) um model Account (conta), e um model Account possui um model AccountHistory (histórico da conta). Você pode utlizar um relacionamento `hasOneThrough` (possui um através de) para acessar o histório da conta do fornecedor através do model Conta (Account) que pertence a ele.

    /**
     * Pegar histórico da conta do fornecedor (AccountHistory)
     * através da conta do fornecedor (Account)
     */
    public function accountHistory()
    {
        return $this->hasOneThrough(AccountHistory::class, Account::class);
    }

### Descobrimento automático de politicas do Model

Quando utlizando Laravel 5.8, cada model possuia sua própria [politica de autorização](/docs/{{version}}/authorization#creating-policies) que precisava ser explicitamente registrada no service provider `AuthServiceProvider` da sua aplicação:

    /**
     * O mapeamento de politicas da sua aplicação
     *
     * @var array
     */
    protected $policies = [
        'App\User' => 'App\Policies\UserPolicy',
    ];

O Laravel 5.8 introduz um novo descobrimento automático de politicas do model desde que o nome do model e da politica sigam a convenção de nomes do laravel. Especificamente, as politicas devem estar na pasta `Politices` acima da pasta que contem os models. Então, por exemplo, os models devem ser colocados na pasta `app` enquanto as politicas devem ser colocadas na pasta `app/Policies`. Para complementar, o nome da politica deve ser igual ao nome do model e ter um sufixo `Policy`. Então, um model `User` teria uma classe correspondente `UserPolicy`. 

Se você quiser usar sua própria lógica de resolução de politicas, você pode registrar um callback customizado utilizando o método `Gate::guessPolicyNamesUsing`. Tipicamente, esse método deve ser chamado do seu service provider `AuthServiceProvider`

    use Illuminate\Support\Facades\Gate;

    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // retorna o nome da classe que contem
        // a politica do model
    });

> {note} Todas as politicas que forem explicitamente mapeadas no seu service provider `AuthServiceProvider` terão precedencia sobre quaisquer politicas auto-descobertas.

### Padrão de Cache PSR-16

Para garantir um tempo de expiração mais flexivel e granular durante o armazenamento de itens e ficar de acordo com o padrão PSR-16, o tempo de vida dos métodos de armazenamento do cache foram alterados de minutos para segundos. Os métodos `put`, `putMany`, `add`, `remember`, e `setDefaultCacheTime` da classe `Illuminate\Cache\Repository` e todas as classes que a extendem, assim como o método `put` de todos os fornecedores de cache tiveram seu comportamento alterado. Veja [a PR relacionada](https://github.com/laravel/framework/pull/27276) para mais informações. 

Se você estiver passando um número inteiro para qualquer um desses métodos, você deve atualizar seu código para garantir que você está passando o numero de segundos que você deseja que o item seja mantido no cache. Alternativamente, você pode passar uma instância de `DateTime` indicando quando o item deve expirar:

    // Laravel 5.7 - Armazenar item por 30 minutos...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.8 - Armazenar item por 30 segundos...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.7 / 5.8 - Armazenar item por 30 segundos...
    Cache::put('foo', 'bar', now()->addSeconds(30));

### Multiplos guards de autenticação para canais de transmissão

Nas versões anteriores do Laravel, canais privados e presenciais autenticavam o usuário utilizando o seu guard de autenticação padrão. A partir do Laravel 5.8, você pode indicar multiplos guards para autenticar o usuario em quaisquer requisições para um canal de transmissão:

    Broadcast::channel('channel', function() {
        // ...
    }, ['guards' => ['web', 'admin']])

### Hash de Token no Guard Token

O guard `token` do laravel, que fornece autenticação basica para a API, agora suporta armazenamento de tokens no formato de hashes do tipo SHA-256. Isso fornece uma segurança melhor do que armazenar tokens em texto plano. Para saber mais sobre hack de tokens, por favor veja a documentação sobre [Autenticação de API](/docs/{{version}}/api-authentication)

> **Note:** Enquanto o laravel fornece um guard basico e simples baseado em tokens, nos recomendamos altamente que você considere utilizar o [Laravel Passport](/docs/{{version}}/passport) for robust, production applications that offer API authentication.

### Validação de Email Melhoradas

O Laravel 5.8 introduz melhorias a logica do validador de email adotando o pacote `egulias/email-validator` utilizado pelo SwiftMailer. As versões anteriores do laravel ocasionalmente considerava emails validos, como por exemplo `example@bär.se`, como sendo invalidos.

### Timezone Padrão do Agendador

O Laravel permite que você customize o fuso horário  de uma tarefa agendada utlizando o método `timezone`:

    $schedule->command('inspire')
             ->hourly()
             ->timezone('America/Sao_Paulo');

No entanto, isso pode se tornar extremamente repetitivo se você estiver especificando o mesmo fuso horário para todas as tarefas da sua aplicação. Por isso, você agora pode definir um método `scheduleTimezone` no seu arquivo `app/Console/Kernel.php`. Esse método deve retornar o fuso horário que você deseja utilizar para todas as tarefas agendadas: 

    /**
     * Recebe o fuso horário que deve ser utilizado por padrão para tarefas agendadas.
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Sao_Paulo';
    }

### Tabelas Intermediarias / Eventos em Models Pivot (Pivôs)

Em versões anteriores do Laravel, [Eventos de models do Eloquent](/docs/{{version}}/eloquent#events) não eram dispachados quando vinculados, desvinculados, ou sincronizados com tabelas intermediarias customizadas / models "pivot" (pivôs) de um relacionamento many-to-many (muitos-pra-muitos). Quando estiver utilizando [models customizados para tabelas intermediarias](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models) no Laravel 5.8, os models aplicaveis agora dispacharão os eventos corretamente.

### Melhorias em Chamadas do Artisan

O laravel permite que você invoque uma ação do laravel através do método `Artisan::call`. Em versões anteriores do laravel, as opções do comando eram passadas através de um array como segundo argumento no método:

    use Illuminate\Support\Facades\Artisan;

    Artisan::call('migrate:install', ['database' => 'foo']);

No entanto, o Laravel 5.8 permite que você passe o comando inteiro, incluindo as opções, junto com o primeiro parâmetro:

    Artisan::call('migrate:install --database=foo');

### Métodos Auxiliares de Teste de Spy / Mock

Para tornar o mocking (simulação) de objetos mais conveniente, os novos métodos `mock` e `spy` foram adicionados a classe base de testes do Laravel. Estes métodos automaticamente ligam uma classe mockada (simulada) no container. Por exemplo:

    // Laravel 5.7
    $this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    }));

    // Laravel 5.8
    $this->mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

### Presevação de Caches em Resources do Eloquent

Quando retornando uma [coleção de resources do Eloquent](/docs/{{version}}/eloquent-resources) de uma rota, o Laravel reseta as chaves de uma coleção, para que eles estejam ordenados numéricamente:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Quando utlizando o Laravel 5.8, você pode agora adicionar uma propriedade `preserveKeys` na classe do seu resource para indicar se as chaves devem ser preservadas ou não. Por padrão, e para manter a consistencia com versões anteriores do Laravel, as chaves são resetadas:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Indica se a coleção desse resource deve preservar suas chaves.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

Quando a propriedade `preserveKeys` está setada como `true`, as chaves serão preservadas:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

### Método do Eloquent do tipo Higher Order `orWhere`

Em versões anteriores do Laravel, cominar multiplos escopos de Models do eloquent utilizando uma query `or` necessitavam do uso de uma Closure:

    // Escopos definidos no nosso model User scopePopular e scopeActive...
    $users = App\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

O Laravel 5.8 introduz um novo método do tipo "higher order" `orWhere` que permite que você encadeie esses escopos juntos sem o uso de uma Closue:

    $users = App\User::popular()->orWhere->active()->get();

### Melhorias no Artisan Serve

Em versões anteriores do Laravel, o comando `serve` do Artisan iniciaria sua aplicação na porta `8000`. Se outro comando `serve` já estivesse escutando esta porta, uma tentativa de iniciar uma segunda aplicação através do `serve` falharia. A partir do Laravel 5.8, `serve` vai buscar por novas portas disponiveis até a `8009`, permitindo que você sirva multiplas aplicações de uma vez.

### Mapeamento de Arquivos do Blade

Quando compilar arquivos do Blade, o laravel agora adiciona um comentario no topo do arquivo compilado que contem o caminho para o arquivo Blade original.

### Drivers de sessão e cache utilizando DynamoDB

O Laravel 5.8 introduz drivers de cache e sessão com [DynamoDB](https://aws.amazon.com/dynamodb/). DynamoDB é um banco de dados NoSQL serverless fornecido pela Amazon Web Services. Por Padrão a configuração para utilizar o driver de cache do `dynamodb` pode ser encontrada no [arquivo de configuração de cache](https://github.com/laravel/laravel/blob/master/config/cache.php)

### Suporte a Carbon 2.0

O Laravel 5.8 fornece suporte para a versão `~2.0` do carbon, uma biblioteca de manipulação de datas.

### Suporte a Pheanstalk 4.0

O Laravel 5.8 fornece suporte para a versão `~4.0` da biblioteca de queue (filas) Pheanstalk. Se você está utlizando a biblioteca Pheanstalk na sua aplicação, por favor atualize a sua biblioteca para utlizar a versão `~4.0` através do composer.
