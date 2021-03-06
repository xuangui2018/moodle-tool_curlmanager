# moodle-tool_curlmanager

* [What is this?](#what-is-this)
* [How does it work?](#how-does-it-work)
* [What does it do?](#what-does-it-do)
* [Branches](#branches)
* [Installation](#installation)
* [References](#references)


What is this?
-------------

Moodle comes with a built in 'security helper' which is what enforces the $CFG->curlsecurityblockedhosts and related settings. This plugin augments this existing security helper and adds additional features such as reporting on what urls are being curled to help inform policy decisions.


How does it work?
-----------------

This relies on backporting this tracker:

Allow plugins to augment the curl security helper via callback

https://tracker.moodle.org/browse/MDL-70649

A function tool_curlmanager_security_helper is defined in admin/tool/curlmanager/lib.php which will be called back.

A curlmanager_security_helper object will be returned from the above function and url_is_blocked method in curlmanager_security_helper will be triggered before making each curl request.

What does it do?
-----------------

1. Allow curl requests only on ```List of allow hosts``` specified in plugin settings if ```Allowed hosts``` setting is enabled.

2. Log all curl requests made by moodle curl (new curl()) or functions that uses moodle curl (e.g. download_file_content) irrespective an url is blocked or not.

3. Report on the curl requests - summary report.

4. Report on the curl requests - domain aggregation report.

5. Please note URL will be treated as blocked if the url is specified in ```List of allow hosts``` and included in ```$CFG->curlsecurityblockedhosts```.


Caveats
-------

Not all outgoing traffic will be logged, there are some known edge cases:

* All Moodle code and plugins which use the Moodle curl libraries should use the security helper.
  However a plugin can pass in 'ignoresecurity'. In general this should only be done for internal
  services and not for traffic outbound the internet.
* Some Moodle plugins do not user the Moodle curl libraries, in particular Guzzle is a very common
  library in use. These will not use the security helper, but if they are being used for general
  internet traffic then they *should* use the Moodle proxy settings.
* Code which uses curl inside a DB transaction which gets rolled back. In this case the security
  helper will be used, but the logging may not happen.

Branches
--------

| Moodle verion     |  Totara version          | Branch      | PHP        | Backports  |
| ----------------- | ------------------------ |------------ | ---------  | -----------|
| 3.9               |                          | VERSION1    | 7.4+       | MDL-70649  |
|                   |  12                      | VERSION1    | 7.0+       | MDL-70649  |

Installation
------------
Checkout or download the plugin source code into folder `admin\tool_curlmanager` of your Moodle installation.

```sh
git clone git@github.com:catalyst/moodle-tool_curlmanager.git admin\tool\curlmanager
```
or
```sh
wget https://github.com/catalyst/moodle-tool_curlmanager/archive/VERSION1.zip
mkdir -p admin\tool\curlmanager
unzip VERSION1.zip -d admin\tool\curlmanager
```
Then go to your Moodle admin interface and complete installation and configuration.

Testing
-------

Fire of a quick curl through the Moodle curl library:

```sh
php -r "define('CLI_SCRIPT', 1); require('config.php'); ((new curl())->get('https://catalyst-au.net'));"
```

You should now see this appear in the domain report:

/admin/tool/curlmanager/domain_report.php

References
----------

See also:

Tracker: Allow an alternate curl security helper

https://tracker.moodle.org/browse/MDL-70649

This plugin was developed by Catalyst IT Australia:

https://www.catalyst-au.net/

<img alt="Catalyst IT" src="https://cdn.rawgit.com/CatalystIT-AU/moodle-auth_saml2/master/pix/catalyst-logo.svg" width="400">
