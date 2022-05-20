# DB Kit

### PHP functions to handle database connection and CRUD operations with PDO.


# Installation
Easy install with composer:
```
composer require hxgf/dbkit
```
```php
use DBkit\db;
require __DIR__ . '/vendor/autoload.php';
```

# Usage
## db::init($settings)
Initializes the database connection. The output from this function should be assigned to a global 'database' variable.
```php
$GLOBALS['database'] = db::init([
  'driver' => 'mysql', // optional, defaults to 'mysql'
  'host' => 'localhost',
  'name' => 'database_name',
  'user' => 'username',
  'password' => 'password'
]);
```

## db::insert($table, $input)
Sanitizes parameters and inserts an array of data into a specific table. <br />
Returns the `id` field of the record created.
```php
$new_id = db::insert("celestial_bodies", [
  'name' => 'Luna',
  'classification' => 'moon',
  'comment' => 'Earth\'s moon, also commonly referred to as "the moon"'
]);

echo $new_id;
```

## db::find($table, $criteria, $options)
Sanitizes parameters and retrieves a specific record(/s) from a specific table, building a query with `SELECT *`. <br />
Returns an array with the the data and total number of records.
```php
$planets = db::find("celestial_bodies", "classification = 'planet' ORDER BY title ASC LIMIT 8");

foreach ($planets['data'] as $p){
  echo $p['title'];
  echo $p['classification'];
  // etc, etc
}

echo $planets['total'];  // 8
```
`db::get()` and `db::fetch()` are also available as aliases to `db::find()`, and behave the exact same way.

### Raw Queries
Raw SQL queries can be used by sending a `raw` parameter like this:
```php
$space_objects = db::find("", "SELECT title, classification FROM celestial_bodies WHERE id IS NOT NULL", [
  'raw' => true
]);
```
<sub>\* NOTE: Be careful when writing raw queries, as none of these parameters are sanitized.</sub>


## db::update($table, $input, $criteria)
Sanitizes parameters and updates an array of data for a specific record(/s).
```php
db::update("celestial_bodies", [
  'name' => 'Mars',
  'comment' => 'Research "The Phobos Incident" -- we are not alone'
], "name='Marz'");
```

## db::delete($table, $criteria)
Sanitizes parameters and deletes a specific record(/s).
```php
db::delete("celestial_bodies", "name='venice'");
```

## db::where_placeholders($criteria)
Creates placeholders and sanitizes data for query building. This function is used to sanitize parameters for all functions.

It returns an array with a string of generated placeholders (`where`), and an array of the actual data to be used in the query (`data`).

An example of how it is used to prepare and execute a `db::find()` query:
```php
$wd = db::db_where_placeholders($where);
try {
  $query = "SELECT * FROM $table WHERE " . $wd['where'];
  $a = $GLOBALS['database']->prepare($query);
  $a->execute($wd['data']);
  $a->setFetchMode(PDO::FETCH_ASSOC);
}
catch(PDOException $e) {
  echo $e->getMessage();
}
```

