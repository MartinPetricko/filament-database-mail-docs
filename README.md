## Introduction

Are you tired of writing email templates in laravel, managing all translations and editing them always when something
changes? Well not anymore! Let your clients manage them in filament panel and call it a feature!

This package allows you to store email templates in your database, assign them to events and send them when the event is
fired. It also comes with Unlayer editor to manage your templates. All events properties are shared to email and you
can use them just like you would in standard [blade views](https://laravel.com/docs/12.x/blade).

## Features

- Store email templates in database
- Assign email templates to standard laravel events
- Send email templates when the event is fired
- Access all of event public properties in email templates
- Define possible mail recipients for each event
- Define possible mail attachments for each event
- Use Unlayer editor to manage your templates
- Use blade syntax in mail templates
- Delay emails sending for time interval after the event is fired
- Load templates from localy created designs or from your Unlayer account
- See any exceptions that occured while sending email due to badly formated mail templates
- Easy to extend

## Screenshots

#### List of Email Templates

![list](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/list.png)

#### Email Detail

![view](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/view.png)

#### Email Creation

![edit](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/edit.png)

#### UI For Available Priperties

![merge-tags](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tags.png)

#### Built-in Conditions and Loops

![merge-tag-rules-detail](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tag-rules-detail.png)

#### Custom Blade Syntax

![custom-blade](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/custom-blade.png)

#### Load Templates

![load-template](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/load-template.png)

#### Email Formating Excepitons

![exception](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/exception.png)

#### Received Email

![mail](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/mail.png)

## Installation

You can install the package via composer:

```bash
composer require martinpetricko/filament-database-mail
```

Publish and run the migrations with:

```bash
php artisan vendor:publish --tag="database-mail-migrations"
php artisan migrate
```

Publish the config file with:

```bash
php artisan vendor:publish --tag="database-mail-config"
```

This is the contents of the published config file:

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
     * This property definitions can be later shown to user as available
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

Optionaly you can publish other config file:

```bash
php artisan vendor:publish --tag="filament-mail-config"
```

This is the contents of the published config file:

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

Register excepitons reporting in `bootstrap/app.php`:

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
     * List of possible recipients that can receive the mail.
     * MailTemplate stores keys of recipients that will
     * receive the mail when event is fired.
     *
     * @return array<string, Recipient>
     */
    public static function getRecipients(): array
    {
        return [
            'user' => new Recipient('Registered User', fn (Registered $event) => $event->user),
            'inviter-user' => new Recipient('Inviter user', fn (Registered $event) => $event->inviter),
        ];
    }

    /**
     * List of possible attachments that can be attached to the mail.
     * MailTemplate stores keys of attachments that will be
     * attached to the mail when event is fired.
     *
     * @return array<string, Attachment>
     */
    public static function getAttachments(): array
    {
        return [
            'terms-of-service' => new Attachment('Terms of Services', fn (Registered $event) => [
                \Illuminate\Mail\Mailables\Attachment::fromUrl('https://my-project.com/tos')->as('tos.pdf')
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

Add list of events to your published `config/database-mail.php` file:

```php
'events' => [
    \App\Events\Registered::class,
],
```

### Create Mail Template

![create](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/create.png)

> **Note:** Don't forget to activate email after you created the template, as by default it is deactivated.

### Dispatch Event

Dispatch event as you would any other laravel event with it's parameters.

```php
use App\Events\Registered;

// ... your bussiness logic

Registered::dispatch($registeredUser, $registeredUserEmailVerificationUrl);
```

## Extensibility

You can override MailTemplateResource to you liking. For example let's
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
    'mail_template' => App\Models\MailTemplate::class,
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

    protected static string $resource = MailTemplateResource::class;

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

    protected static string $resource = MailTemplateResource::class;

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

    protected static string $resource = MailTemplateResource::class;

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

> **Note:** Don't forget to set your users [preferred locale](https://laravel.com/docs/12.x/mail#user-preferred-locales), so sent mails are in the right language.

**Voilà!** Enjoy your translatable email templates that can be managed from filament panel.

## Changelog

Please see [CHANGELOG](https://github.com/MartinPetricko/filament-database-mail-docs/blob/main/CHANGELOG.md) for more
information on what has changed recently.

## Security Vulnerabilities

Please review [our security policy](https://github.com/MartinPetricko/filament-database-mail-docs/security/policy) on
how to report security vulnerabilities.
