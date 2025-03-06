# Filament Database Mail

This package allows you to store email templates in your database, assign them to events and send them when the event is
fired. It also comes with Unlayer editor to manage your templates.

## Installation

You can install the package via composer:

```bash
composer require martinpetricko/filament-database-mail
```

You can publish and run the migrations with:

```bash
php artisan vendor:publish --tag="filament-database-mail-migrations"
php artisan migrate
```

You can publish the config file with:

```bash
php artisan vendor:publish --tag="filament-database-mail-config"
```

Optionally, you can publish the views using

```bash
php artisan vendor:publish --tag="filament-database-mail-views"
```

This is the contents of the published config file:

```php
return [
];
```

## Usage

```php
$filamentDatabaseMail = new MartinPetricko\FilamentDatabaseMail();
echo $filamentDatabaseMail->echoPhrase('Hello, MartinPetricko!');
```

## Testing

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [Martin Petricko](https://github.com/MartinPetricko)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
