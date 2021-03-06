# check_php

check_php is a POSIX compliant nagios plugin that will check for PHP startup errors (`-s`), missing PHP modules (`-m`), misconfigured directives in php.ini (`-c`) and for available PHP updates (`-u`). This plugin supports performance data (error and warning counts over time) and long output (exact detail about all problems).

[![Build Status](https://travis-ci.org/cytopia/check_php.svg?branch=master)](https://travis-ci.org/cytopia/check_php)
[![Latest Stable Version](https://poser.pugx.org/cytopia/check_php/v/stable)](https://packagist.org/packages/cytopia/check_php)
[![Total Downloads](https://poser.pugx.org/cytopia/check_php/downloads)](https://packagist.org/packages/cytopia/check_php)
[![Latest Unstable Version](https://poser.pugx.org/cytopia/check_php/v/unstable)](https://packagist.org/packages/cytopia/check_php)
[![License](https://poser.pugx.org/cytopia/check_php/license)](http://opensource.org/licenses/MIT)
[![POSIX](https://img.shields.io/badge/posix-100%25-brightgreen.svg)](https://en.wikipedia.org/?title=POSIX)
[![Type](https://img.shields.io/badge/type-%2Fbin%2Fsh-red.svg)](https://en.wikipedia.org/?title=Bourne_shell)

[Nagios Configuration](#1-nagios-configuration) |
[Icinga2 Configuration](#2-icinga2-configuration) |
[Usage](#3-usage) |
[Examples](#4-examples) |
[License](#5-license) |
[Contributors](#6-contributors) |
[Awesome](#7-awesome)

---

| [![Awesome-Nagios-Plugins](https://raw.githubusercontent.com/cytopia/awesome-nagios-plugins/master/doc/img/awesome-nagios.png)](https://github.com/cytopia/awesome-nagios-plugins) | Find more plugins at [Awesome Nagios](https://github.com/cytopia/awesome-nagios-plugins) |
|---|---|
| [![Icinga Exchange](https://raw.githubusercontent.com/cytopia/awesome-nagios-plugins/master/doc/img/icinga.png)](https://exchange.icinga.com/cytopia) | **Find more plugins at [Icinga Exchange](https://exchange.icinga.com/cytopia)** |
| [![Nagios Exchange](https://raw.githubusercontent.com/cytopia/awesome-nagios-plugins/master/doc/img/nagios.png)](https://exchange.nagios.org/directory/Owner/cytopia/1) | **Find more plugins at [Nagios Exchange](https://exchange.nagios.org/directory/Owner/cytopia/1)** |

---

##### Requirements
| Program  | Required | Description |
| ------------- | ------------- | -------- |
| bourne shell (sh)  | yes  | The whole script is written in pure bourne shell (sh) and is 100% Posix compliant |
| [check_by_ssh](https://www.monitoring-plugins.org/doc/man/check_by_ssh.html)  | yes  | This nagios plugin is used as a wrapper to check on remote hosts |
| wget, curl or fetch | Optional  | Either one of them is required if you want to check against PHP updates. (`-u`) |

##### Features
* Check for PHP startup errors
* Check for PHP updates (inside minor versions and patch levels)
* Check for missing PHP modules
* Check for blacklisted PHP modules and throw err/warn if they are compiled in
* Check for expected php.ini config directives (e.g.: date.timezone must be "Europe/Berlin", etc)
* Each check can specify its own severity (warning or error)

##### Motivation
If you have to take care about many servers which have PHP installed you can use this plugin to make sure that all servers or all groups of server use the same configuration with the same compiled modules and are always up to date.


## 1. Nagios Configuration

### 1.1 Command definition
In order to check php on remote servers you will need to make use of `check_by_ssh`.
```bash
name:    check_by_ssh_php
command: $USER1$/check_by_ssh -H $HOSTADDRESS$ -t 60 -l "$USER17$" -C "$USER22$/check_php $ARG1$"
```
### 1.2 Service definition
In the above command definition there is only one argument variable assigned to `check_php`: `$ARG1`. So you can easily assign all required arguments to this single variable in the service definition:
```bash
check command: check_by_ssh_php
$ARG1$:        -s e -u w -m curl e -m gettext e -m openssl e -m json e
```

## 2. Icinga2 Configuration

### 2.1 Command definition
```javascript
/*
 * Check PHP health
 */
object CheckCommand "php" {
  import "plugin-check-command"

  command = [ PluginDir + "/check_php" ]

  arguments = {
    "-s" = {
      value = "$php_startup$"
      description = "Check for PHP startup errors and display warning or error if any exists. Allowed values are 'w' for warnings and 'e' for critical errors."
    }
    "-u" = {
      value = "$php_updates$"
      description = "Check for updated PHP version online. Allowed values are 'w' for warnings and 'e' for critical errors. This check requires wget, curl or fetch."
    }
    "-p" = {
      value = "$php_binary$"
      description = "Optional path to PHP binary. This argument allows to define a certain PHP binary to be checked. If none is defined, the default PHP version will be used."
    }
    "-d" = {
      value = "$php_delimiter$"
      description = "Delimiter for multi-value arguments."
    }
    "-m" = {
      value = "$php_modules$"
      description = "Check for required modules."
      repeat_key = true
    }
    "-b" = {
      value = "$php_blacklist$"
      description = "Check for blacklisted modules."
      repeat_key = true
    }
    "-c" = {
      value = "$php_config$"
      description = "Check PHP setting directives that diverge from the given default value."
      repeat_key = true
    }
    "-v" = {
      set_if = "$php_verbose$"
      description = "Enable verbose mode."
    }
  }
}
```

### 2.2 Service definition example (using apply)
```javascript
/*
 * PHP Health
 */
apply Service "php-" for (php => config in host.vars.php) {
  check_command = "php"
  // Assuming your PHP setup doesn't change too often, we don't
  // bother to check twice a day only.
  check_interval = 12h

  display_name = "PHP " + php
  notes = "Checks currently installed PHP " + php + " health."

  // Service variables from php definition.
  vars += config
  vars.php_delimiter = "|"
  if ( config.php_modules ) {
    vars.php_modules = []
    for (key => value in config.php_modules) {
      vars.php_modules += [ key + vars.php_delimiter + value ]
    }
  }
  if ( config.php_blacklist ) {
    vars.php_blacklist = []
    for (key => value in config.php_blacklist) {
      vars.php_blacklist += [ key + vars.php_delimiter + value ]
    }
  }
  if ( config.php_config ) {
    vars.php_config = []
    for (key => value in config.php_config) {
      vars.php_config += [ key + vars.php_delimiter + value["default"] + vars.php_delimiter + value["severity"] ]
    }
  }

  // Application rules.
  assign where host.name = NodeName && host.vars.php
}
```

### 2.3 Host object definition
```javascript
object Host "node.example.com" {

  // Other settings.
  vars.php[ "7.1" ] = {
    php_binary = "/opt/php/7.1/bin/php"
    php_updates = "w"
    php_startup = "e"
    php_modules = {
      intl = "w"
      mbstring = "w"
      soap = "w"
      apcu = "w"
      memcached = "w"
      geoip = "w"
      mongodb = "w"
      imagick = "w"
      redis = "w"
      openssl = "w"
      xml = "w"
      json = "w"
      curl = "w"
    }
    php_blacklist = {
      mcrypt = "w"
    }
    php_config = {
      "date.timezone" = {
        "default" = "Europe/Berlin"
        "severity" = "w"
      }
    }
  }
}
```


## 3. Usage

Each argument allows you to specify which severity should be triggered (`<w|e>`), where `w` triggers a warning and `e` triggers an error.
Arguments that can be used multiple times (`-m` and `-c`) can of course use different severities each time. All severities will be aggregated and the highest severity (error > warning) will determine the final state.

```bash
Usage: check_php [-s <w|e>] [-u <w|e>] [-m <module> <w|e>] [-b <module> <w|e> [-c <conf> <val> <w|e>] [-p <path>] [-d <delimiter>] [-v]
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

  -b <module> <w|e>      [multiple] Check PHP for modules that should not be compiled in and display
                         nagios warning/error if the module is compiled against PHP.
                         Use multiple times to check for multiple blacklisted modules.
                         Example: -b "imagick" w -b "tidy" e

  -c <conf> <val> <w|e>  [multiple] Check for misconfigured directives in php.ini and display
                         nagios warning/error if the configuration does not match.
                         Use multiple times to check against multiple configurations.
                         Example: -c "date.timezone" "Europe/Berlin" e

  -p <path>              [optional] Define the path to the PHP binary that shall be used.  
                         If no value is given, the current user's default PHP version will be checked.
                         Example: -p "/usr/bin/php"

  -d <delimiter>         [optional] Delimiter used to concatenate arguments of the above options
                         that require multiple values.
                         Example: -d "|" -m "mysql|w" -b "mcrypt|w" -c "date.timezone|Europe/Berlin|e"

  -v                     Be verbose (Show PHP Version and Zend Engine Version)

  -h                     Display help

  -V                     Display version
```


## 4. Examples

Checking against prefered timezone and compiled module `mysql`

```bash
$ check_php -c "date.timezone" "Europe/Berlin" e -m mysql e
[ERR] PHP 5.6.16 has errors: Missing module(s) | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[CRITICAL] Module: "mysql" not available
[OK]       Config "date.timezone" = "Europe/Berlin"
```

Checking for PHP startup errors

```bash
$ check_php -s w
[WARN] PHP 5.6.16 has warning: Startup errors | 'OK'=0;;;; 'Errors'=0;;;; 'Warnings'=1;;;; 'Unknown'=0;;;;
[WARNING]  PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test' - dlopen(/usr/local/Cellar/php56/5.6.14/lib/php/extensions/no-debug-non-zts-20131226/test, 9): image not found in Unknown on line 0
```

Combine multiple module checks

```bash
$ check_php -m mysql e -m mysqli w -m mbstring w
[OK] PHP 5.6.16 is healthy | 'OK'=1;;;; 'Errors'=0;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       Module: "mysql" available
[OK]       Module: "mysqli" available
[OK]       Module: "mbstring" available
```

Checking for PHP Updates (OK)
```bash
$ check_php -u e
[OK] PHP 5.6.16 is healthy | 'OK'=1;;;; 'Errors'=0;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[OK]       PHP Version 5.6.14 up to date.
```

Checking for PHP Updates (Updates available)
```bash
$ check_php -u e
[ERR] PHP 5.6.13 has errors: Updates available | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=-;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[CRITICAL] PHP Version 5.6.13 too old. Latest: 5.6.14.
```

Checking for PHP Updates (Able to differentiate between PHP 5.4, 5.5 and 5.6)
```bash
$ check_php -u e
[ERR] PHP 5.5.1 has errors: Updates available | 'OK'=0;;;; 'Errors'=1;;;; 'Warnings'=0;;;; 'Unknown'=0;;;;
[OK]       No PHP startup errors
[CRITICAL] PHP Version 5.5.1 too old. Latest: 5.5.30.
```

A lot of options combined
```bash
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


## 5. License

[![license](https://poser.pugx.org/cytopia/check_php/license)](http://opensource.org/licenses/mit)


## 6. Contributors

* [cytopia](https://github.com/cytopia)
* [MarioSteinitz](https://github.com/MarioSteinitz)


## 7. Awesome

Added by the following [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/sindresorhus/awesome) lists:

* [awesome-nagios-plugins](https://github.com/cytopia/awesome-nagios-plugins)
