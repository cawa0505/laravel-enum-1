# Laravel support for spatie/enum

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/laravel-enum.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-enum)
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/spatie/laravel-enum/run-tests?label=tests&style=flat-square)](https://github.com/spatie/laravel-enum/actions?query=workflow%3Arun-tests)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/laravel-enum.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/laravel-enum)
[![Code Coverage](https://img.shields.io/coveralls/github/spatie/laravel-enum.svg?style=flat-square)](https://coveralls.io/github/spatie/laravel-enum)
[![StyleCI](https://github.styleci.io/repos/180570906/shield?branch=master)](https://github.styleci.io/repos/180570906)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/laravel-enum.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-enum)

This package provides extended support for our [spatie/enum](https://github.com/spatie/enum) package in Laravel.

## Installation

You can install the package via composer:

```bash
composer require spatie/laravel-enum
```

## Support us	

Learn how to create a package like this one, by watching our premium video course:	

[![Laravel Package training](https://spatie.be/github/package-training.jpg)](https://laravelpackage.training)	

We invest a lot of resources into creating [best in class open source packages](https://spatie.be/open-source). You can support us by [buying one of our paid products](https://spatie.be/open-source/support-us).	

We highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using. You'll find our address on [our contact page](https://spatie.be/about-us). We publish all received postcards on [our virtual postcard wall](https://spatie.be/open-source/postcards).

## Usage

### Model Attribute casting

Chances are that if you're working in a Laravel project, you'll want to use enums within your models.
This package provides two custom casts and the `\Spatie\Enum\Laravel\Enum` also implements the `\Illuminate\Contracts\Database\Eloquent\Castable` interface.

```php
use Illuminate\Database\Eloquent\Model;
use Spatie\Enum\Laravel\Casts\EnumCast;
use Spatie\Enum\Laravel\Casts\EnumCollectionCast;

class TestModel extends Model
{
    protected $casts = [
        'status' => StatusEnum::class,
        'nullable_enum' => EnumCast::class.':'.StatusEnum::class.',nullable',
        'array_of_enums' => EnumCollectionCast::class.':'.StatusEnum::class,
        'nullable_array_of_enums' => EnumCollectionCast::class.':'.StatusEnum::class.',nullable',
    ];
}
```

By using the casts the casted attribute will always be an instance of the given enum.

```php
$model = new TestModel();
$model->status = StatusEnum::DRAFT();
$model->status->equals(StatusEnum::DRAFT());
```

### Validation Rule

This package provides a validation rule to validate your request data against a given enumerable.

```php
use Spatie\Enum\Laravel\Rules\EnumRule;

$rules = [
    'status' => new EnumRule(StatusEnum::class),
];
```

This rule validates that the value of `status` is any possible representation of the `StatusEnum`.

But you can also use the simple string validation rule definition:

```php
$rules = [
    'status' => [
        'enum:'.StatusEnum::class,
    ],
];
```

If you want to customize the failed validation messages you can publish the translation file.

```bash
php artisan vendor:publish --provider="Spatie\Enum\Laravel\EnumServiceProvider" --tag="translation"
```

We pass several replacements to the translation key which you can use.

-   `attribute` - the name of the validated attribute
-   `value` - the actual value that's validated
-   `enum` - the full class name of the wanted enumerable
-   `other` - a comma separated list of all possible values - they are translated via the `enums` array in the translation file

### Request Data Transformation

A common scenario is that you receive an enumerable value as part of yor request data.
To let you work with it as a real enum object you can transform request data to an enum in a similar way to the model attribute casting.

#### Request macro

There is a request macro available which is the base for the other possible ways to cast request data to an enumerable.

```php
$request->transformEnums($enumCastRules);
```

This is an example definition of all possible request enum castings.
There are three predefined keys available as constants on `Spatie\Enum\Laravel\Http\EnumRequest` to cast enums only in specific request data sets.
All other keys will be treated as independent enum casts and are applied to the combined request data set.

```php
use Spatie\Enum\Laravel\Http\EnumRequest;

$enums = [
    // cast the status key independent of it's data set
    'status' => StatusEnum::class,
    // cast the status only in the request query params
    EnumRequest::REQUEST_QUERY => [
        'status' => StatusEnum::class,
    ],
    // cast the status only in the request post data
    EnumRequest::REQUEST_REQUEST => [
        'status' => StatusEnum::class,
    ],
    // cast the status only in the request route params
    EnumRequest::REQUEST_ROUTE => [
        'status' => StatusEnum::class,
    ],
];
```

You can call this macro yourself in every part of your code with access to a request instance.
Most commonly you will do this in your controller action if you don't want to use one of the other two ways.

#### Form Requests

Form requests are the easiest way to cast the data to an enum.

```php
use Illuminate\Foundation\Http\FormRequest;
use Spatie\Enum\Laravel\Http\Requests\TransformsEnums;
use Spatie\Enum\Laravel\Rules\EnumRule;

class StatusFormRequest extends FormRequest
{
    use TransformsEnums;

    public function rules(): array
    {
        return [
            'status' => new EnumRule(StatusEnum::class),
        ];
    }

    public function enums(): array
    {
        return [
            'status' => StatusEnum::class,
        ];
    }
}
```

The request data transformation is done after validation via the `FormRequest::passedValidation()` method. If you define your own `passedValidation()` method you have to call the request macro `transformEnums()` yourself.

```php
protected function passedValidation()
{
    $this->transformEnums($this->enums());

    // ...
}
```

#### Middleware

You can also use the middleware to transform enums in a more general way and for requests without a form request.

```php
use Spatie\Enum\Laravel\Http\Middleware\TransformEnums;

new TransformEnums([
    'status' => StatusEnum::class,
]);
```

### Enum Make Command

We provide an artisan make command which allows you to quickly create new enumerables.

```bash
php artisan make:enum StatusEnum
```

You can use `--method` option to predefine some enum values - you can use them several times.

## Testing

```bash
composer test
composer test-coverage
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

-   [Brent Roose](https://github.com/brendt)
-   [Tom Witkowski](https://github.com/Gummibeer)
-   [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
