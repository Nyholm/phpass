PHPass: Portable PHP password hashing framework
===============================================

This library is a reimplementation of the [PHPass](http://openwall.com/phpass/) class. The output is 100% compatible with the original, the library is simply updated to be more extensible.

The PHPass library uses two main methods to generate secure password hashes. The first is simply to use salts and high-quality one-way hash functions. The use of cryptographic salts prevents an attacker from using [rainbow tables](http://en.wikipedia.org/wiki/Rainbow_table) for password lookup. The only way for an attacker to determine the original password is to use either [dictionary](http://en.wikipedia.org/wiki/Dictionary_attack) or [brute-force](http://en.wikipedia.org/wiki/Brute_force_attack) methods.

The second method is to utilize [key stretching](http://en.wikipedia.org/wiki/Key_stretching). Key stretching makes hash generation exponentially more computationally expensive. This means that instead of being able to try 100 passwords per second, an attacker might only be able to try 10.

These methods, combined with a good (yet sane!) password policy will create password hashes which are highly improbable to crack.

Requirements
------------

You will need to be using PHP 5.3+ in order to use this library. It should not have any other external dependencies.

Installation
------------

Installation can be done via PEAR.

```bash
pear channel-discover rchouinard.github.com/pear
pear install rych/PHPass-1.0.0RC1
```

Alternatively, you may choose to clone the git repository and add the library folder to your include path.

```bash
git clone git://github.com/rchouinard/phpass.git
```

Quick Start
-----------

The main class is Phpass, and uses two main methods: `hashPassword()` and `checkPassword()`, shown in the example below. By default, the class will create an instance of the blowfish adapter if an adapter is not otherwise given.

```php
<?php
$phpass = new Phpass;

// Returns a hash which can be stored in a database
$hash = $phpass->hashPassword($myPassword);

// Checks a password against a stored hash, probably from a database
if ($phpass->checkPassword($myPassword, $hash) {
    echo 'Password matches!';
} else {
    echo 'Password does not match!';
}
```

Configuration
-------------

The Phpass class can be given either a pre-configured Adapter instance, or an array of configuration options. Currently, the only supported options key is "adapter", which is expected to be either an Adapter instance or an array with the keys "type" and "options."

### Passing an adapter to the constructor

```php
<?php
$adapter = new Phpass\Adapter\Portable;
$phpass = new Phpass($adapter);
```

### Passing an array of options to the constructor

```php
<?php
$options = array (
    'adapter' => array (
        'type' => 'extdes', // One of blowfish, extdes, or portable
        'options' => array ( // Options array passed to adapter constructor
            'iterationCountLog2' => 12
        )
    )
);
$phpass = new Phpass($options);
```

Adapters
--------

The library includes three adapters which wrap different cipher methods.

#### Common Options

<dt>iterationCountLog2</dt>
  <dd>_Integer_; This number is the base-2 logarithm representation of the iteration count used for key stretching. The final iteration count can be calculated with 2^x, so a value of 8 will yield 2^8, or 256, iterations. A value of 12 will yield 2^12, or 4,096 iterations.</dd>

### Blowfish

This adapter uses the blowfish cipher to create one-way hashes.

```php
<?php
$adapterOptions = array (
    'iterationCountLog2' => 8
);

$adapter = new Phpass\Adapter\Blowfish($adapterOptions);

// $2a$08$xZQ8G2a1XLjxr14Rc0zOP.
$salt = $adapter->genSalt();

// $2a$08$xZQ8G2a1XLjxr14Rc0zOP.X8atMFBx8J6EaFhniNvujgcqs17TgGC
$hash = $adapter->crypt('password', $salt);
```

### Extended DES

This adapter uses the extended DES cipher to create one-way hashes.

```php
<?php
$adapterOptions = array (
    'iterationCountLog2' => 8
);

$adapter = new Phpass\Adapter\ExtDes($adapterOptions);

// _zzD.d84Z
$salt = $adapter->genSalt();

// _zzD.d84ZhoAfE8.PYgQ
$hash = $adapter->crypt('password', $salt);
```

### Portable

This adapter uses a custom cipher to create one-way hashes. It is compatible with phpBB passwords.

```php
<?php
$adapterOptions = array (
    'iterationCountLog2' => 8
);

$adapter = new Phpass\Adapter\Portable($adapterOptions);

// $P$BItJrOG/2
$salt = $adapter->genSalt();

// $P$BItJrOG/2OwpzlDFMRNPR8vUIlcBbi/
$hash = $adapter->crypt('password', $salt);
```

License
-------

Since US law does not allow directly publishing works to the public domain, I'm licensing this library under the terms of the [MIT license](http://www.opensource.org/licenses/mit-license.html).