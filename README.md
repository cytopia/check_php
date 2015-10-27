# check_php

[![Build Status](https://travis-ci.org/cytopia/check_php.svg?branch=master)](https://travis-ci.org/cytopia/check_php)
[![Latest Stable Version](https://poser.pugx.org/cytopia/check_php/v/stable)](https://packagist.org/packages/cytopia/check_php) [![Total Downloads](https://poser.pugx.org/cytopia/check_php/downloads)](https://packagist.org/packages/cytopia/check_php) [![Latest Unstable Version](https://poser.pugx.org/cytopia/check_php/v/unstable)](https://packagist.org/packages/cytopia/check_php) [![License](https://poser.pugx.org/cytopia/check_php/license)](http://opensource.org/licenses/MIT)
[![POSIX](https://img.shields.io/badge/posix-100%25-brightgreen.svg)](https://en.wikipedia.org/?title=POSIX)
[![Type](https://img.shields.io/badge/type-%2Fbin%2Fsh-red.svg)](https://en.wikipedia.org/?title=Bourne_shell)

Check_php is a POSIX compliant nagios plugin that will check for PHP startup errors, missing PHP modules and misconfigured directives in php.ini.


## Usage

```shell
Usage: check_php [-s <w|e>] [-m <module>] [-c <conf> <val>] [-v]
       check_php -h
       check_php -V

Nagios plugin that will check for PHP startup errors,
missing modules and misconfigured directives.

  -s <w|e>           Check for PHP startup errors and display
                     nagios warning or error if any exists.
                     Warning:  -s w
                     Error:    -s e
                     (Default: -s e)

  -m <module>        Require compiled PHP module and display
                     nagios error if the module was not compiled against PHP.
                     Use multiple times to check against multiple modules.
                     Example: -m "mysql" -m "mysqli"

  -c <conf> <val>    Check for misconfigured directives in php.ini and display
                     nagios error if the configuration does not match.
                     Use multiple times to check against multiple configurations.
                     Example: -c "date.timezone" "Europe/Berlin"

  -v                 Be verbose (Show PHP Version and Zend Engine Version)

  -h                 Display help

  -V                 Display version
```


## Examples

Checking against prefered timezone and compiled module `mysql`

```shell
$ check_php -c "date.timezone" "Europe/Berlin" -m mysql
[ERR] PHP Errors detected. | OK'=0;;;; 'Errors'=0;;;; 'Warnings'=1;;;; 'Unknown'=0;;;;
[ERR]  Module: "mysql" not available
[OK]   Config "date.timezone" = "Europe/Berlin"
```

Checking for PHP startup errors

```shell
$ check_php -s w
[WARN] PHP Warnings detected. | OK'=0;;;; 'Errors'=0;;;; 'Warnings'=1;;;; 'Unknown'=0;;;;
[WARN] PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test' - dlopen(/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test, 9): image not found in Unknown on line 0
```

Combine multiple module checks

```shell
$ check_php -m mysql -m mysqli -m mbstring
[OK] No PHP Errors detected. | 'OK'=1;;;; 'Errors'=0;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]   Module: "mysql" available
[OK]   Module: "mysqli" available
[OK]   Module: "mbstring" available
```
