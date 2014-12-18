# Poignant

Eloquent on steroids, implements RoR like ActiveRecord features as traits, including in-model validations,
user stamping, declarative relations, cascaded operations on relations (save, delete).

The DeclarativeRelations is extracted from [Ardent](https://github.com/laravelbook/ardent) by Max Ehsan (see LICENSE_Ardent).

Copyright (C) 2014 Pascal Hurni <[https://github.com/phurni](https://github.com/phurni)>

Licensed under the [MIT License](http://opensource.org/licenses/MIT).

## Installation

Add `mortimer/poignant` as a requirement to `composer.json`:

```javascript
{
    "require": {
        "mortimer/poignant": "dev-master"
    }
}
```

Update your packages with `composer update` or install with `composer install`.

The master branch works for both Laravel 4.2 and 5.0

## Getting Started

`Poignant` is a collection of PHP traits for Eloquent models. You are free to use any or many of those traits by simply
adding the `use` statement in your model, like this:

```php
use Mortimer\Poignant\UserStamping;

class MyModel extends Eloquent {
  use UserStamping;
}
```

You may also import all of them by inheriting from the `Model` class:

```php
use Mortimer\Poignant\Model;

class MyModel extends Model {
}
```

## Documentation

Here is the list of traits with their behaviours, you'll find the detailed documentation in the next chapters.

 * UserStamping
   Provides automatic filling of `create_by_id`, `updated_by_id` and `deleted_by_id` attributes.

 * DeclarativeRelations
   Eases the declaration of relationships with a property array.

### UserStamping

Provides automatic filling of `create_by_id`, `updated_by_id` and `deleted_by_id` attributes.
These are filled only if the columns exists on the table, so no need to worry about using or not the trait,
simply use and forget it.

This trait stamps the user by using its `id` not a name string. So you may add relations on your models to
associate them correctly.

Every column name may be customized by defining them in your model:

```php
use Mortimer\Poignant\UserStamping;

class MyModel extends Eloquent {
  use UserStamping;
  
  protected static $CREATED_BY = 'FK_created_by';
}
```

You may also override the value stored in those columns:

```php
use Mortimer\Poignant\UserStamping;

class MyModel extends Eloquent {
  use UserStamping;
  
  public function getUserStampValue()
  {
      // This is the default value, return what you desire here to override the default behaviour
      return \Auth::user()->getKey();
  }
}
```

### DeclarativeRelations

This one is extracted from Ardent which itself picked the idea from the Yii framework.

Can be used to ease declaration of relationships in Eloquent models.
Follows closely the behavior of the relation methods used by Eloquent, but packing them into an indexed array
with relation constants make the code less cluttered.

It should be declared with camel-cased keys as the relation name, and value being a mixed array with the
relation constant being the first (0) value, the second (1) being the classname and the next ones (optionals)
having named keys indicating the other arguments of the original methods: 'foreignKey' (belongsTo, hasOne,
belongsToMany and hasMany); 'table' and 'otherKey' (belongsToMany only); 'name', 'type' and 'id' (specific for
morphTo, morphOne and morphMany).
Exceptionally, the relation type MORPH_TO does not include a classname, following the method declaration of
`\Illuminate\Database\Eloquent\Model::morphTo`.

Example:

```php
use Mortimer\Poignant\DeclarativeRelations;
use Mortimer\Poignant\DeclarativeRelationsTypes as DRT;

class Order extends Eloquent {
    use DeclarativeRelations;
    
    protected static $relationsData = [
        'items'    => [DRT::HAS_MANY, 'Item'],
        'owner'    => [DRT::HAS_ONE, 'User', 'foreignKey' => 'user_id'],
        'pictures' => [DRT::MORPH_MANY, 'Picture', 'name' => 'imageable']
    ];
}
```

Or by extending the base model (no more needs of DRT):

```php
use Mortimer\Poignant\Model;

class Order extends Model {
    use DeclarativeRelations;
    
    protected static $relationsData = [
        'items'    => [self::HAS_MANY, 'Item'],
        'owner'    => [self::HAS_ONE, 'User', 'foreignKey' => 'user_id'],
        'pictures' => [self::MORPH_MANY, 'Picture', 'name' => 'imageable']
    ];
}
```

For fellow developers that want to pick the relations declaration from anywhere else than the `$relationsData` property,
you simply have to override these three methods to accomodate for this:

  * `protected static function getDeclaredRelations()`
  * `protected static function getDeclaredRelationOptions($relationName)`
  * `protected static function hasDeclaredRelation($relationName)`
