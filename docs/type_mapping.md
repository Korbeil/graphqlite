---
id: type_mapping
title: Type mapping
sidebar_label: Type mapping
---

As explained in the [queries](queries.md) section, the job of GraphQLite is to create GraphQL types from PHP types.

## Scalar mapping

Scalar PHP types can be type-hinted to the corresponding GraphQL types:

* `string`
* `int`
* `bool`
* `float`

For instance:

```php
namespace App\Controller;

use TheCodingMachine\GraphQLite\Annotations\Query;

class MyController
{
    /**
     * @Query
     */
    public function hello(string $name): string
    {
        return 'Hello ' . $name;
    }
}
```

## Class mapping

When returning a PHP class in a query, you must annotate this class using `@Type` and `@Field` annotations:

```php
namespace App\Entities;

use TheCodingMachine\GraphQLite\Annotations\Field;
use TheCodingMachine\GraphQLite\Annotations\Type;

/**
 * @Type()
 */
class Product
{
    // ...

    /**
     * @Field()
     */
    public function getName(): string
    {
        return $this->name;
    }

    /**
     * @Field()
     */
    public function getPrice(): ?float
    {
        return $this->price;
    }
}
```

## Array mapping

You can type-hint against arrays (or iterators) as long as you add a detailed `@return` statement in the PHPDoc.

```php
/**
 * @Query
 * @return User[] <=== we specify that the array is an array of User objects.
 */
public function users(int $limit, int $offset): array
{
    // Some code that returns an array of "users".
}
```

## ID mapping

GraphQL comes with a native `ID` type. PHP has no such type.

There are two ways with GraphQLite to handle such type.

### Force the outputType

```php
/**
 * @Field(outputType="ID")
 */
public function getId(): string
{
    // ...
}
```

Using the `outputType` attribute of the `@Field` annotation, you can force the output type to `ID`.

You can learn more about forcing output types in the [custom types section](custom_types.md).

### ID class

```php
use TheCodingMachine\GraphQLite\Types\ID;

/**
 * @Field
 */
public function getId(): ID
{
    // ...
}
```

Note that you can also use the `ID` class as an input type:

```php
use TheCodingMachine\GraphQLite\Types\ID;

/**
 * @Mutation
 */
public function save(ID $id, string $name): Product
{
    // ...
}
```

## Date mapping

Out of the box, GraphQL does not have a `DateTime` type, but we took the liberty to add one, with sensible defaults.

When used as an output type, `DateTimeImmutable` or `DateTimeInterface` PHP classes are 
automatically mapped to this `DateTime` GraphQL type.

```php
/**
 * @Field
 */
public function getDate(): \DateTimeInterface
{
    return $this->date;
}
```

The `date` field will be of type `DateTime`. In the returned JSON response to a query, the date is formatted as a string
in the **ISO8601** format (aka ATOM format).

<div class="alert alert-error">
    PHP <code>DateTime</code> type is not supported.
</div>

## Union types

You can create a GraphQL union type *on the fly* using the pipe `|` operator in the PHPDoc:

```php
/**
 * @Query
 * @return Company|Contact <== can return a company OR a contact.
 */
public function companyOrContact(int $id)
{
    // Some code that returns a company or a contact.
}
```

## Enum types

<small>Available in GraphQLite 4.0+</small>

PHP has no native support for enum types. Hopefully, there are a number of PHP libraries that emulate enums in PHP.
The most commonly used library is [myclabs/php-enum](https://github.com/myclabs/php-enum) and GraphQLite comes with
native support for it.

You will first need to install [myclabs/php-enum](https://github.com/myclabs/php-enum):

```bash
$ composer require myclabs/php-enum
```

Now, any class extending the `MyCLabs\Enum\Enum` class will be mapped to a GraphQL enum:

```php
use MyCLabs\Enum\Enum;

class StatusEnum extends Enum
{
    private const ON = 'on';
    private const OFF = 'off';
    private const PENDING = 'pending';
}
```

```php
/**
 * @Query
 * @return User[]
 */
public function users(StatusEnum $status): array
{
    if ($status === StatusEum::ON()) {
        // ...
    }
    // ...
}
```

<div class="alert alert-info">There are many enumeration library in PHP and you might be using another library.
If you want to add support for your own library, this is not extremely difficult to do. You need to register a custom
"RootTypeMapper" with GraphQLite. You can learn more about <em>type mappers</em> in the <a href="internals.md">"internals" documentation</a>
and <a href="https://github.com/thecodingmachine/graphqlite/blob/master/src/Mappers/Root/MyCLabsEnumTypeMapper.php">copy/paste and adapt the root type mapper used for myclabs/php-enum</a>.</div>

## More scalar types

<small>Available in GraphQLite 4.0+</small>

GraphQL supports "custom" scalar types. GraphQLite supports adding more GraphQL scalar types.

If you need more types, you can check the [GraphQLite Misc. Types library](https://github.com/thecodingmachine/graphqlite-misc-types).
It adds support for more scalar types out of the box in GraphQLite.

Or if you have some special needs, [you can develop your own scalar types](custom-types.md#registering-a-custom-scalar-type-advanced).
