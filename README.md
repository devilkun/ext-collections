# ext-collections

[![Travis-CI](https://travis-ci.com/CismonX/ext-collections.svg?branch=master)](https://travis-ci.com/CismonX/ext-collections)
[![Codecov](https://codecov.io/gh/CismonX/ext-collections/branch/master/graphs/badge.svg)](https://codecov.io/gh/CismonX/ext-collections)
[![MIT license](https://img.shields.io/badge/licence-MIT-blue.svg)](LICENSE)

## 1. Introduction

This PHP extension provides a set of useful and convenient operations on PHP arrays, which makes working with arrays simple and scalable.

Method names and functionalities are inspired by [Kotlin.Collections](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/), having a slightly different style than that of [Laravel Collections](https://laravel.com/docs/5.8/collections).

Requires PHP version >= 7.1 and < 8.0 (master branch).

### 1.1 Example

Here is a simple example on how to work with arrays gracefully using this extension.

```php
$employees = [
    ['name' => 'Alice',    'gender' => 'female', 'age' => 35],
    ['name' => 'Bob',      'gender' => 'male',   'age' => 29],
    ['name' => 'David',    'gender' => 'male',   'age' => 40],
    ['name' => 'Benjamin', 'gender' => 'male',   'age' => 32]
];
// Trying to get an array of names of male employees,
// sorted by the descending order of their age.
$names = Collection::init($employees)
    ->filter(function ($value) {
        return $value['gender'] == 'male';
    })
    ->sortedByDescending(function ($value) {
        return $value['age'];
    })
    ->map(function ($value) {
        return $value['name'];
    })
    ->toArray();
// You got $names == ['David', 'Benjamin', 'Bob'].
```

## 2. Getting Started

### 2.1 Installation

Like other PHP extensions, ext-collections can be built and installed with a few commands:

```bash
phpize
./configure
make
sudo make install
```

Include it in your PHP configuration file to enable this extension:

```php.ini
extension=collections.so
```

Building on Windows is not as convenient, however, pre-built binaries for Windows are provided in the [releases](https://github.com/CismonX/ext-collections/releases). If you want to build it yourself, follow the [official PHP wiki](https://wiki.php.net/internals/windows/stepbystepbuild_sdk_2).

### 2.2 API Reference

See [stubs](stubs/) directory for signatures of all classes and methods of this extension, with PHPDoc. They can also serve as IDE helper.

### 2.3 PHP-style Access

The `Collection` class implements `ArrayAccess` and `Countable` interface internally, you can treat an instance of `Collection` as an `ArrayObject`.

* The `isset()`, `unset()` keywords can be used on elements of `Collection`.
* Elements can be accessed via property or bracket expression.
* `empty()`, `count()` can be used on instance of `Collection`.
* Elements can be traversed via `foreach()` keyword.

## 3. Notes

* The `Collection::xxxTo()` methods will preserve the original key-value pairs of destination `Collection` when keys collide.
* Some methods of `Collection` involves comparing two of its elements, which accepts `$flags` as one of its arguments. When these methods are being invoked, make sure all elements are of the same type (numeric/string/others), otherwise you're likely to get a segfault.

### 3.1 Copy-on-write Mechanism

Class `Collection` does not introduce new data structures internally. Instead, it only holds a pointer to a `zend_array`, and all its methods works directly on top of `zend_array`. Which means conversion between `Collection` and `array` does not involve copying, until write operation is performed on one of the duplicates.

```php
$foo = ['a', 'b'];              // arr0: refcount = 1
$bar = Collection::init($foo);  // arr0: refcount = 2, no copying of either `zend_array` or its elements
echo $bar->containsValue('a');  // arr0: refcount = 2, read operation, no copying
$bar->shuffle();                // arr0: refcount = 1, arr1: refcount = 1, write operation, `zend_array` is separated
$baz = $bar->toArray();         // arr0: refcount = 1, arr1: refcount = 2, no copying
```
