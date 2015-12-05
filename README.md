# check_php

[Nagios Configuration](https://github.com/cytopia/check_php#1-nagios-configuration) |
[Usage](https://github.com/cytopia/check_php#2-usage) |
[Examples](https://github.com/cytopia/check_php#3-examples) |
[License](https://github.com/cytopia/check_php#4-license) |
[Awesome](https://github.com/cytopia/check_php#5-awesome)

[![Build Status](https://travis-ci.org/cytopia/check_php.svg?branch=master)](https://travis-ci.org/cytopia/check_php)
[![Latest Stable Version](https://poser.pugx.org/cytopia/check_php/v/stable)](https://packagist.org/packages/cytopia/check_php) [![Total Downloads](https://poser.pugx.org/cytopia/check_php/downloads)](https://packagist.org/packages/cytopia/check_php) [![Latest Unstable Version](https://poser.pugx.org/cytopia/check_php/v/unstable)](https://packagist.org/packages/cytopia/check_php) [![License](https://poser.pugx.org/cytopia/check_php/license)](http://opensource.org/licenses/MIT)
[![POSIX](https://img.shields.io/badge/posix-100%25-brightgreen.svg)](https://en.wikipedia.org/?title=POSIX)
[![Type](https://img.shields.io/badge/type-%2Fbin%2Fsh-red.svg)](https://en.wikipedia.org/?title=Bourne_shell)

Check_php is a POSIX compliant nagios plugin that will check for PHP startup errors (`-s`), missing PHP modules (`-m`), misconfigured directives in php.ini (`-c`) and for available PHP updates (`-u`). This plugin supports performance data (error and warning counts over time) and long output (exact detail about all problems).

##### Requirements
| Program  | Required | Description |
| ------------- | ------------- | -------- |
| bourne shell (sh)  | yes  | The whole script is written in pure bourne shell (sh) and is 100% Posix compliant |
| [check_by_ssh](https://www.monitoring-plugins.org/doc/man/check_by_ssh.html)  | yes  | This nagios plugin is used as a wrapper to check on remote hosts |
| wget, curl or fetch | Optional  | Either one of them is required if you want to check against PHP updates. (`-u`) |



## 1. Nagios Configuration
### Command definition
In order to check php on remote servers you will need to make use of `check_by_ssh`.
```shell
name:    check_by_ssh_php
command: $USER1$/check_by_ssh -H $HOSTADDRESS$ -t 60 -l "$USER17$" -C "$USER22$/check_php $ARG1$"
```
### Service definition
In the above command definition there is only one argument variable assigned to `check_php`: `$ARG1`. So you can easily assign all required arguments to this single variable in the service definition:
```shell
check command: check_by_ssh_php
$ARG1$:        -s e -u w -m curl e -m gettext e -m openssl e -m json e
```

## 2. Usage

Each argument allows you to specify which severity should be triggered (`<w|e>`), where `w` triggers a warning and `e` triggers an error.
Arguments that can be used multiple times (`-m` and `-c`) can of course use different severities each time. All severities will be aggregated and the highest severity (error > warning) will determine the final state.

```shell
Usage: check_php [-s <w|e>] [-u <w|e>] [-m <module> <w|e>] [-c <conf> <val> <w|e>] [-v]
       check_php -h
       check_php -V

Nagios plugin that will check if PHP exists, for PHP startup errors,
missing modules, misconfigured directives and available updates.

  -s <w|e>               [single] Check for PHP startup errors and display
                         nagios warning or error if any exists.
                         Warning:  -s w
                         Error:    -s e

  -u <w|e>               [single] Check for updated PHP version online. (requires wget, curl or fetch)
                         Will only check for patch updates and will not notify if your current version
                         PHP 5.5 and there is already PHP 5.6 out there.

  -m <module> <w|e>      [multiple] Require compiled PHP module and display
                         nagios warning/error if the module was not compiled against PHP.
                         Use multiple times to check against multiple modules.
                         Example: -m "mysql" w -m "mysqli" e

  -c <conf> <val> <w|e>  [multiple] Check for misconfigured directives in php.ini and display
                         nagios warning/error if the configuration does not match.
                         Use multiple times to check against multiple configurations.
                         Example: -c "date.timezone" w -c "Europe/Berlin" e

  -v                     Be verbose (Show PHP Version and Zend Engine Version)

  -h                     Display help

  -V                     Display version
```


## 3. Examples

Checking against prefered timezone and compiled module `mysql`

```shell
$ check_php -c "date.timezone" "Europe/Berlin" e -m mysql e
[ERR] PHP 5.6.16 has errors: Missing module(s) | OK'=0;;;; 'Errors'=1;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[CRITICAL] Module: "mysql" not available
[OK]       Config "date.timezone" = "Europe/Berlin"
```

Checking for PHP startup errors

```shell
$ check_php -s w
[WARN] PHP 5.6.16 has warning: Startup errors | OK'=0;;;; 'Errors'=0;;;; 'Warnings'=1;;;; 'Unknown'=0;;;;
[WARNING]  PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test' - dlopen(/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test, 9): image not found in Unknown on line 0
```

Combine multiple module checks

```shell
$ check_php -m mysql e -m mysqli w -m mbstring w
[OK] PHP 5.6.16 is healthy | 'OK'=1;;;; 'Errors'=0;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       Module: "mysql" available
[OK]       Module: "mysqli" available
[OK]       Module: "mbstring" available
```

Checking for PHP Updates (OK)
```shell
$ check_php -u e
[OK] PHP 5.6.16 is healthy | 'OK'=1;;;; 'Errors'=0;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[OK]       PHP Version 5.6.14 up to date.
```

Checking for PHP Updates (Updates available)
```shell
$ check_php -u e
[ERR] PHP 5.6.13 has errors: Updates available | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=-;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[CRITICAL] PHP Version 5.6.13 too old. Latest: 5.6.14.
```

Checking for PHP Updates (Able to differentiate between PHP 5.4, 5.5 and 5.6)
```shell
$ check_php -u e
[ERR] PHP 5.5.1 has errors: Updates available | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[CRITICAL] PHP Version 5.5.1 too old. Latest: 5.5.30.
```

A lot of options combined
```shell
$ check_php -s w -m mysql e -m mbstring e -m xml e -c date.timezone 'Europe/Berlin' e -c session.cookie_secure "On" e -u e -v
[ERR] PHP 5.6.14 has errors: Wrong config | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[OK]       PHP Version 5.6.14 up to date.
[OK]       Module: "mysql" available
[OK]       Module: "mbstring" available
[OK]       Module: "xml" available
[OK]       Config "date.timezone" = "Europe/Berlin"
[CRITICAL] Config "session.cookie_secure" = "Off", excpected: "On"
PHP 5.6.14
Zend Engine v2.6.0
```

## 4. License
[![license](https://poser.pugx.org/cytopia/check_php/license)](http://opensource.org/licenses/mit)

## 5. Awesome

Added by the following [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/sindresorhus/awesome) lists:

* [awesome-nagios-plugins](https://github.com/cytopia/awesome-nagios-plugins)
