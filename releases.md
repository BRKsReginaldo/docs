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

Laravel 5.8 continua as melhorias feitas no Laravel 5.7 introduzindo relacionamentos Eloquent do tipo hasOneThrough (possui um através de), validação de email melhoradas, registro automatico de politicas de autorização baseado em convenções, drivers de cache e sessões utilizando DynamoDB, configurações de timezone do agendador de tarefas melhoradas, suporte para utilização de multiplos guards de autenticação em canais de transmissão, drivers de cache agora seguem o padrão PSR-16, melhorias no comando `artisan serve`, suporte a PHPUnit 8.0, suporte a Carbon 2.0, suporte a Pheanstalk 4.0, e uma variedade de outras correções de bugs e melhorias de usabilidade. 

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
        // return policy class name...
    });

> {note} Any policies that are explicitly mapped in your `AuthServiceProvider` will take precedence over any potential auto-discovered policies.

### PSR-16 Cache Compliance

In order to allow a more granular expiration time when storing items and provide compliance with the PSR-16 caching standard, the cache item time-to-live has changed from minutes to seconds. The `put`, `putMany`, `add`, `remember` and `setDefaultCacheTime` methods of the `Illuminate\Cache\Repository` class and its extended classes, as well as the `put` method of each cache store were updated with this changed behavior. See [the related PR](https://github.com/laravel/framework/pull/27276) for more info.

If you are passing an integer to any of these methods, you should update your code to ensure you are now passing the number of seconds you wish the item to remain in the cache. Alternatively, you may pass a `DateTime` instance indicating when the item should expire:

    // Laravel 5.7 - Store item for 30 minutes...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', 30);

    // Laravel 5.7 / 5.8 - Store item for 30 seconds...
    Cache::put('foo', 'bar', now()->addSeconds(30));

### Multiple Broadcast Authentication Guards

In previous releases of Laravel, private and presence broadcast channels authenticated the user via your application's default authentication guard. Beginning in Laravel 5.8, you may now assign multiple guards that should authenticate the incoming request:

    Broadcast::channel('channel', function() {
        // ...
    }, ['guards' => ['web', 'admin']])

### Token Guard Token Hashing

Laravel's `token` guard, which provides basic API authentication, now supports storing API tokens as SHA-256 hashes. This provides improved security over storing plain-text tokens. To learn more about hashed tokens, please review the full [API authentication documentation](/docs/{{version}}/api-authentication).

> **Note:** While Laravel ships with a simple, token based authentication guard, we strongly recommend you consider using [Laravel Passport](/docs/{{version}}/passport) for robust, production applications that offer API authentication.

### Improved Email Validation

Laravel 5.8 introduces improvements to the validator's underlying email validation logic by adopting the `egulias/email-validator` package utilized by SwiftMailer. Laravel's previous email validation logic occasionally considered valid emails, such as `example@bär.se`, to be invalid.

### Default Scheduler Timezone

Laravel allows you to customize the timezone of a scheduled task using the `timezone` method:

    $schedule->command('inspire')
             ->hourly()
             ->timezone('America/Chicago');

However, this can become cumbersome and repetitive if you are specifying the same timezone for all of your scheduled tasks. For that reason, you may now define a `scheduleTimezone` method in your `app/Console/Kernel.php` file. This method should return the default timezone that should be assigned to all scheduled tasks:

    /**
     * Get the timezone that should be used by default for scheduled events.
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Chicago';
    }

### Intermediate Table / Pivot Model Events

In previous versions of Laravel, [Eloquent model events](/docs/{{version}}/eloquent#events) were not dispatched when attaching, detaching, or syncing custom intermediate table / "pivot" models of a many-to-many relationship. When using [custom intermediate table models](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models) in Laravel 5.8, the applicable model events will now be dispatched.

### Artisan Call Improvements

Laravel allows you to invoke Artisan via the `Artisan::call` method. In previous releases of Laravel, the command's options are passed via an array as the second argument to the method:

    use Illuminate\Support\Facades\Artisan;

    Artisan::call('migrate:install', ['database' => 'foo']);

However, Laravel 5.8 allows you to pass the entire command, including options, as the first string argument to the method:

    Artisan::call('migrate:install --database=foo');

### Mock / Spy Testing Helper Methods

In order to make mocking objects more convenient, new `mock` and `spy` methods have been added to the base Laravel test case class. These methods automatically bind the mocked class into the container. For example:

    // Laravel 5.7
    $this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    }));

    // Laravel 5.8
    $this->mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

### Eloquent Resource Key Preservation

When returning an [Eloquent resource collection](/docs/{{version}}/eloquent-resources) from a route, Laravel resets the collection's keys so that they are in simple numerical order:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

When using Laravel 5.8, you may now add a `preserveKeys` property to your resource class indicating if collection keys should be preserved. By default, and to maintain consistency with previous Laravel releases, the keys will be reset by default:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Indicates if the resource's collection keys should be preserved.
         *
         * @var bool
         */
        public $preserveKeys = true;
    }

When the `preserveKeys` property is set to `true`, collection keys will be preserved:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all()->keyBy->id);
    });

### Higher Order `orWhere` Eloquent Method

In previous releases of Laravel, combining multiple Eloquent model scopes via an `or` query operator required the use of Closure callbacks:

    // scopePopular and scopeActive methods defined on the User model...
    $users = App\User::popular()->orWhere(function (Builder $query) {
        $query->active();
    })->get();

Laravel 5.8 introduces a "higher order" `orWhere` method that allows you to fluently chain these scopes together without the use of Closures:

    $users = App\User::popular()->orWhere->active()->get();

### Artisan Serve Improvements

In previous releases of Laravel, Artisan's `serve` command would serve your application on port `8000`. If another `serve` command process was already listening on this port, an attempt to serve a second application via `serve` would fail. Beginning in Laravel 5.8, `serve` will now scan for available ports up to port `8009`, allowing you to serve multiple applications at once.

### Blade File Mapping

When compiling Blade templates, Laravel now adds a comment to the top of the compiled file which contains the path to the original Blade template.

### DynamoDB Cache / Session Drivers

Laravel 5.8 introduces [DynamoDB](https://aws.amazon.com/dynamodb/) cache and session drivers. DynamoDB is a serverless NoSQL database provided by Amazon Web Services. The default configuration for the `dynamodb` cache driver can be found in the Laravel 5.8 [cache configuration file](https://github.com/laravel/laravel/blob/master/config/cache.php).

### Carbon 2.0 Support

Laravel 5.8 provides support for the `~2.0` release of the Carbon date manipulation library.

### Pheanstalk 4.0 Support

Laravel 5.8 provides support for the `~4.0` release of the Pheanstalk queue library. If you are using Pheanstalk library in your application, please upgrade your library to the `~4.0` release via Composer.
