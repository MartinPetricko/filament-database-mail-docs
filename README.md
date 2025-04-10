## Introduction

Are you tired of writing email templates in Laravel, managing all translations and constantly editing them whenever
something changes? Well, not anymore! Let your clients manage them in filament panel and call it a feature!

Filament Database Mail is a FilamentPHP implementation of an
open-source [Laravel Database Mail](https://github.com/MartinPetricko/laravel-database-mail) package that lets you
store email templates in your database, link them to events, and automatically send them when those events are
dispatched. All event properties are shared to email and you can use them just like you would in
standard [Blade views](https://laravel.com/docs/12.x/blade).

This package prepares FilamentPHP resources for managing your email templates with an implementation
of [Unlayer Editor](https://docs.unlayer.com/builder/latest/email-builder).

> **Security Notice:** This package allows php code executions as email templates are parsed
> with [Blade::render()](https://laravel.com/docs/12.x/blade#rendering-inline-blade-templates). So only administrators
> with highest level of access should be able to use this package.

## Features

- Store email templates in database
- Assign email templates to standard Laravel events
- Send emails when the events are fired
- Access all of event public properties in email templates
- Define possible email recipients for each event
- Define possible email attachments for each event
- Use Unlayer editor to manage your templates
- Use blade syntax in email templates
- Delay emails sending for time interval after the event is dispatched
- Load templates from locally created designs or from your Unlayer account
- See any exceptions that occurred while sending email due to badly formatted mail templates
- Easy to extend

## Screenshots

#### List of Email Templates

![list](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/list.png)

#### Email Detail

![view](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/view.png)

#### Email Creation

![edit](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/edit.png)

#### UI for Available Properties

![merge-tags](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tags.png)

#### Built-in Conditions and Loops

![merge-tag-rules-detail](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tag-rules-detail.png)

#### Custom Blade Syntax

![custom-blade](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/custom-blade.png)

#### Load Templates

![load-template](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/load-template.png)

#### Email Formatting Exceptions

![exception](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/exception.png)

#### Received Email

![mail](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/mail.png)

## Installation

### Requirements

Filament Database Mail requires `PHP ^8.2`, `Filament ^3.3` and `Laravel 11+`.

### Activating Your License on AnyStack

Filament Database Mail uses AnyStack to handle payment, licensing, and distribution.

During the purchasing process, AnyStack will provide you with a license key. Once your license key is activated, you can
proceed installing with composer below.

### Installing with Composer

To Install Filament Database Mail with Composer, you'll need to add the following package repository to your
`composer.json` file:

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://filament-database-mail.composer.sh"
        }
    ]
}
```

Once the repository has been added to the composer.json file, you can install Filament Database Mail like any other
composer package using the composer require command:

```bash
composer require martinpetricko/filament-database-mail
```

Next, you will be prompted to provide your username and password.

```bash
Loading composer repositories with package information
Authentication required (filament-database-mail.composer.sh):
Username: [license-email]
Password: [license-key]
```

Your username will be your email address and the password is your license key. For example, let's say we have the
following email and license activation:

- Contact email: john@gmail.com
- License key: 8c21df8f-6273-4932-b4ba-8bcc723ef500

You will need to enter the above information as follows when prompted for your credentials:

```bash
Loading composer repositories with package information
Authentication required (filament-database-mail.composer.sh):
Username: john@gmail.com
Password: 8c21df8f-6273-4932-b4ba-8bcc723ef500
```

### Setting up Filament Database Mail

Publish and run the migrations with:

```bash
php artisan vendor:publish --tag="database-mail-migrations"
php artisan migrate
```

Publish the config file with:

```bash
php artisan vendor:publish --tag="database-mail-config"
```

These are the contents of the published config file:

```php
return [
    /**
     * Register event listener for all TriggersDatabaseMail events,
     * that sends mails associated with the event.
     */
    'register_event_listener' => true,

    /**
     * Period of time when mail exceptions are pruned.
     */
    'prune_exceptions_period' => now()->subMonth(),

    /**
     * Models that are used by Laravel Database Mail.
     */
    'models' => [
        'mail_exception' => \MartinPetricko\LaravelDatabaseMail\Models\MailException::class,
        'mail_template' => \MartinPetricko\LaravelDatabaseMail\Models\MailTemplate::class,
    ],

    /**
     * Mailable that is used to send the mail from database.
     */
    'event_mail' => \MartinPetricko\LaravelDatabaseMail\Mail\EventMail::class,

    /**
     * Resolvers are used to automatically resolve properties of the event.
     * These property definitions can be later shown to user as available
     * variables that can be used in the mail template.
     */
    'resolvers' => [
        \MartinPetricko\LaravelDatabaseMail\Properties\Resolvers\EloquentResolver::class,
        \MartinPetricko\LaravelDatabaseMail\Properties\Resolvers\BooleanResolver::class,
        \MartinPetricko\LaravelDatabaseMail\Properties\Resolvers\StringResolver::class,
    ],

    /**
     * Register events that implement TriggersDatabaseMail interface.
     * Events will be used to trigger the mail and this list
     * of events can be shown to user as available events.
     */
    'events' => [
        // \Illuminate\Auth\Events\Registered::class,
    ],
];
```

Optionally, you can publish other config file:

```bash
php artisan vendor:publish --tag="filament-database-mail-config"
```

These are the contents of the published config file:

```php
return [
    /**
     * Unlayer configuration
     */
    'unlayer' => [
        /**
         * https://docs.unlayer.com/builder/installation#where-to-get-the-project-id
         */
        'project_id' => env('UNLAYER_PROJECT_ID'),

        /**
         * https://docs.unlayer.com/api#section/Authentication/Get-your-API-Key
         */
        'api_key' => env('UNLAYER_API_KEY'),
    ],
];
```

> **Note:** You can use this package without setting up Unlayer project, but option to load templates from Unlayer will
> be disabled.

Register exceptions reporting in `bootstrap/app.php`:

```php
use Illuminate\Foundation\Configuration\Exceptions;
use MartinPetricko\LaravelDatabaseMail\Exceptions\DatabaseMailException;
use MartinPetricko\LaravelDatabaseMail\Facades\LaravelDatabaseMail;

//...

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (DatabaseMailException $e) {
        LaravelDatabaseMail::logException($e);
    });
})
```

Enable exceptions table pruning:

```php
Schedule::command('model:prune', [
    '--model' => [
        \MartinPetricko\LaravelDatabaseMail\Models\MailException::class,
    ],
])->daily();
```

Register and configure plugin for your filament panel:

```php
->plugins([
    \MartinPetricko\FilamentDatabaseMail\FilamentDatabaseMailPlugin::make()
        ->navigationLabel('Mails')
        ->navigationGroup('Settings')
        ->navigationIcon('heroicon-o-envelope')
        ->navigationSort(5);
])
```

## Usage

### Create Events

Add `TriggersDatabaseMail` interface and `CanTriggerDatabaseMail` trait to your standard laravel events.

```php
namespace App\Events;

use App\Models\User;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
use MartinPetricko\LaravelDatabaseMail\Attachments\Attachment;
use MartinPetricko\LaravelDatabaseMail\Events\Concerns\CanTriggerDatabaseMail;
use MartinPetricko\LaravelDatabaseMail\Events\Contracts\TriggersDatabaseMail;
use MartinPetricko\LaravelDatabaseMail\Recipients\Recipient;

class Registered implements TriggersDatabaseMail
{
    use Dispatchable;
    use SerializesModels;
    use CanTriggerDatabaseMail;
    
    /**
     * All public properties of the event will be passed 
     * to the mail template body and subject.
     */
    public function __construct(public User $user, public string $emailVerificationUrl, public ?User $inviter = null)
    {
        //
    }

    /**
     * Name of the event that can be used in the UI.
     */
    public static function getName(): string
    {
        return 'User Registered';
    }

    /**
     * Description of the event that can be used in the UI.
     */
    public static function getDescription(): ?string
    {
        return 'Fires when a user is registered';
    }

    /**
     * List of possible recipients that can receive the email.
     * MailTemplate stores recipient keys that will
     * receive the email when event is triggered.
     *
     * @return Recipient<Registered>[]
     */
    public static function getRecipients(): array
    {
        return [
            'user' => new Recipient('Registered User', fn (Registered $event) => $event->user),
            'inviter-user' => new Recipient('Inviter user', fn (Registered $event) => $event->inviter),
            'all' => new Recipient('All', fn (Registered $event) => [
                $event->user,
                $event->inviter,
            ]),
        ];
    }

    /**
     * List of possible attachments that can be attached to the email.
     * MailTemplate stores attachment keys that will be attached 
     * to the email when the event is triggered.
     *
     * @return Attachment<Registered>[]
     */
    public static function getAttachments(): array
    {
        return [
            'terms-of-service' => new Attachment('Terms of Services', fn (Registered $event) => [
                \Illuminate\Mail\Attachment::fromUrl('https://my-project.com/tos')->as('tos.pdf'),
            ]),
        ];
    }
}
```

#### Event Properties

Public properties of the event will try to get parsed
into [Unlayer merge tags](https://docs.unlayer.com/builder/dynamic-content/merge-tags) automatically via registered
resolvers in your `config/database-mail.php` file. However not all properties can be parsed into merge tags, so you can
define your own resolvers or add properties map to the event.

```php
namespace App\Events;

use MartinPetricko\LaravelDatabaseMail\Events\Contracts\TriggersDatabaseMail;
use MartinPetricko\LaravelDatabaseMail\Properties\Property;

class Registered implements TriggersDatabaseMail
{
    /**
     * @param bool $isUserAdmin
     * @param ?array{lorem: string, ipsum: string} $additionalData
     * @param Product[] $products
     * @param string[] $sponsorNames
     */
    public function __construct(
        public bool $isUserAdmin,
        public ?array $additionalData,
        public array $products,
        public array $sponsorNames,
    ) {
        //
    }
    
    //...
    
    /**
     * List of additional properties that can be used in the mail template
     * and were not automatically resolved via registered resolvers.
     *
     * @return array<string, Property>
     */
    public static function mergeProperties(): array
    {
        return [
            'isUserAdmin' => new Property('isUserAdmin')
                ->boolean(),
            'additionalData' => (new Property('additionalData'))
                ->nullable()
                ->properties([
                    new Property('lorem'),
                    new Property('ipsum'),
                ]),
            'products' => new Property('products')
                ->traversable()
                ->properties([
                    (new Property('name'))
                        ->accessor('->getName()'),
                    new Property('url')
                        ->accessor('->getProductUrl()'),
                ]),
            'sponsorNames' => new Property('sponsorNames')
                ->traversable(),
        ];
    }
}
```

> **Note:** Properties map is only used for UI. As long as event property is public, you can use it in your mail
> template just like you would in a blade view.

![is-user-admin](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/is-user-admin.png)

![additional-data](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/additional-data.png)

![products](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/products.png)

![sponsor-names](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/sponsor-names.png)

### Register Events

Add a list of events to your published `config/database-mail.php` file:

```php
'events' => [
    \App\Events\Registered::class,
],
```

### Create Mail Template

![create](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/create.png)

> **Note:** Don't forget to activate the email after creating the template, as by default it is deactivated.

### Dispatch Event

Dispatch the event as you would any other Laravel event with its parameters.

```php
use App\Events\Registered;

// ... your business logic

Registered::dispatch($registeredUser, $registeredUserEmailVerificationUrl);
```

### Import/Export Mail Templates

You can prepare your mail templates before deploying your application to production and then import them in your
seeders.

#### Export Mail Templates

```bash
php artisan mail:export
```

#### Import Mail Templates

```bash
php artisan mail:import
```

#### Seeder Setup

```php
use Illuminate\Support\Facades\Artisan;

public function run(): void
{
    /**
     * Import all mail templates from json files and replace localhost url with production url. 
     */
    Artisan::call('mail:import', [
        '--all' => true,
        '--search' => 'http:\/\/localhost',
        '--replace' => config('app.url'),
    ]);
}
```

## Authorization

In addition to standard policy actions, you can add methods to allow mail template activation/deactivation:

```php
/**
 * Used to activate/deactivate single mail template.
 */
public function activate(User $user, MailTemplate $mailTemplate): bool
{
    return true;
}

/**
 * Used to activate/deactivate mail templates via bulk actions.
 */
public function activateAny(User $user): bool
{
    return true;
}
```

Register your policy:

```php
use App\Policies\MailTemplatePolicy;
use Illuminate\Support\Facades\Gate;
use MartinPetricko\LaravelDatabaseMail\Models\MailTemplate;

public function boot(): void
{
    Gate::policy(MailTemplate::class, MailTemplatePolicy::class);
}
```

## Extensibility

You can override MailTemplateResource to your liking. For example let's
implement [filamentphp/spatie-laravel-translatable-plugin](https://github.com/filamentphp/spatie-laravel-translatable-plugin).

#### Setup Plugin

Follow [installation instructions](https://github.com/filamentphp/spatie-laravel-translatable-plugin) for
filamentphp/spatie-laravel-translatable-plugin.

#### Prepare Your Translatable Model

```php
<?php

namespace App\Models;

use Spatie\Translatable\HasTranslations;

class MailTemplate extends \MartinPetricko\LaravelDatabaseMail\Models\MailTemplate
{
    use HasTranslations;

    /** @var string[] */
    public array $translatable = [
        'subject',
        'body',
        'meta',
    ];
}
```

Update migration file `database/migrations/xxxx_xx_xx_xxxxxx_create_mail_templates_table.php`:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class () extends Migration {
    public function up(): void
    {
        Schema::create('mail_templates', static function (Blueprint $table) {
            $table->id();
            $table->string('event')->index();
            $table->string('name');
            $table->json('subject');            // Changed type to json
            $table->json('body');               // Changed type to json
            $table->json('meta')->nullable();
            $table->json('recipients');
            $table->json('attachments');
            $table->string('delay')->nullable();
            $table->boolean('is_active');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('mail_templates');
    }
};
```

Update your `config/database-mail.php` file to use your model:

```php
'models' => [
    'mail_exception' => \MartinPetricko\LaravelDatabaseMail\Models\MailException::class,
    'mail_template' => \App\Models\MailTemplate::class,
],
```

#### Extend MailTemplateResource

Create `app/Filament/Admin/Resources/MailTemplateResource.php` file:

```php
namespace App\Filament\Admin\Resources;

use App\Filament\Admin\Resources\MailTemplateResource\Pages\CreateMailTemplate;
use App\Filament\Admin\Resources\MailTemplateResource\Pages\EditMailTemplate;
use App\Filament\Admin\Resources\MailTemplateResource\Pages\ViewMailTemplate;
use Filament\Resources\Concerns\Translatable;

class MailTemplateResource extends \MartinPetricko\FilamentDatabaseMail\Resources\MailTemplateResource
{
    use Translatable;

    public static function getPages(): array
    {
        return array_merge(parent::getPages(), [
            'create' => CreateMailTemplate::route('/create'),
            'view' => ViewMailTemplate::route('/{record}'),
            'edit' => EditMailTemplate::route('/{record}/edit'),
        ]);
    }
}
```

Create `app/Filament/Admin/Resources/MailTemplateResource/Pages/CreateMailTemplate.php` file:

```php
namespace App\Filament\Admin\Resources\MailTemplateResource\Pages;

use App\Filament\Admin\Resources\MailTemplateResource;
use Filament\Actions\LocaleSwitcher;
use Filament\Resources\Pages\CreateRecord\Concerns\Translatable;

class CreateMailTemplate extends \MartinPetricko\FilamentDatabaseMail\Resources\MailTemplateResource\Pages\CreateMailTemplate
{
    use Translatable;

    protected function getHeaderActions(): array
    {
        return array_merge(parent::getHeaderActions(), [
            LocaleSwitcher::make(),
        ]);
    }
}
```

Create `app/Filament/Admin/Resources/MailTemplateResource/Pages/EditMailTemplate.php` file:

```php
namespace App\Filament\Admin\Resources\MailTemplateResource\Pages;

use App\Filament\Admin\Resources\MailTemplateResource;
use Filament\Actions\LocaleSwitcher;
use Filament\Resources\Pages\EditRecord\Concerns\Translatable;

class EditMailTemplate extends \MartinPetricko\FilamentDatabaseMail\Resources\MailTemplateResource\Pages\EditMailTemplate
{
    use Translatable;

    protected function getHeaderActions(): array
    {
        return array_merge(parent::getHeaderActions(), [
            LocaleSwitcher::make(),
        ]);
    }
}
```

Create `app/Filament/Admin/Resources/MailTemplateResource/Pages/ViewMailTemplates.php` file:

```php
namespace App\Filament\Admin\Resources\MailTemplateResource\Pages;

use App\Filament\Admin\Resources\MailTemplateResource;
use Filament\Actions\LocaleSwitcher;
use Filament\Resources\Pages\ViewRecord\Concerns\Translatable;

class ViewMailTemplate extends \MartinPetricko\FilamentDatabaseMail\Resources\MailTemplateResource\Pages\ViewMailTemplate
{
    use Translatable;

    protected function getHeaderActions(): array
    {
        return array_merge(parent::getHeaderActions(), [
            LocaleSwitcher::make(),
        ]);
    }
}
```

#### Register Your MailTemplateResource to Plugin

```php
->plugins([
    FilamentDatabaseMailPlugin::make()
        ->mailTemplateResource(\App\Filament\Admin\Resources\MailTemplateResource::class),
])
```

> **Note:** Don't forget to set your
> users [preferred locale](https://laravel.com/docs/12.x/mail#user-preferred-locales), so sent mails are in the right
> language.

**Voilà!** Enjoy your translatable email templates that can be managed from filament panel.

## Changelog

Please see [CHANGELOG](https://github.com/MartinPetricko/filament-database-mail-docs/blob/main/CHANGELOG.md) for more
information on what has changed recently.

## Security Vulnerabilities

Please review [our security policy](https://github.com/MartinPetricko/filament-database-mail-docs/security/policy) on
how to report security vulnerabilities.
