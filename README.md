## Introduction

Are you tired of writing email templates in laravel, managing all translations and editing them always when something
changes? Well not anymore! Let your clients manage them in filament panel and call it a feature!

This package allows you to store email templates in your database, assign them to events and send them when the event is
fired. It also comes with Unlayer editor to manage your templates. All events properties are shared to email and you
can use them just like you would in classic blade views.

## Features

- Store email templates in database
- Assign email templates to standard laravel events
- Send email templates when the event is fired
- Use all of event public properties in email templates
- Define pasible mail recipients for each event
- Define pasible mail attachments for each event
- Use Unlayer editor to manage your templates
- Access event properties automatically in mail templates
- Use any blade directives in mail templates
- Delay emails sending for time interval after the event is fired
- Load templates from localy created designs or from you Unlayer project
- See any exceptions that occured while sending email due to badly formated mail templates

## Screenshots

List of email templates

![list](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/list.png)

Email detail

![view](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/view.png)

Email creation

![edit](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/edit.png)

UI for available priperties

![merge-tags](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tags.png)

Built in conditions and loops 

![merge-tag-rules-detail](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/merge-tag-rules-detail.png)

Custom blade

![custom-blade](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/custom-blade.png)

Load templates

![load-template](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/load-template.png)

Email formating excepitons

![exception](https://raw.githubusercontent.com/MartinPetricko/filament-database-mail-docs/refs/heads/main/assets/screenshots/exception.png)

Received email

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
        'mail_template' => \MartinPetricko\LaravelDatabaseMail\Models\MailTemplate::class,
        'mail_exception' => \MartinPetricko\LaravelDatabaseMail\Models\MailException::class,
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

## Changelog

Please see [CHANGELOG](https://github.com/MartinPetricko/filament-database-mail-docs/blob/main/CHANGELOG.md) for more information on what has changed recently.

## Security Vulnerabilities

Please review [our security policy](https://github.com/MartinPetricko/filament-database-mail-docs/security/policy) on how to report security vulnerabilities.
