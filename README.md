# OWASP CRS - Joomla Rule Exclusions Plugin
[![Integration tests](https://github.com/futsure0/joomla-rule-exclusions-plugin/actions/workflows/integration.yml/badge.svg?branch=main)](https://github.com/futsure0/joomla-rule-exclusions-plugin/actions/workflows/integration.yml?query=branch%3Amain)

## Description

This plugin contains rule exclusions for the administrator back end of
[Joomla](https://www.joomla.org/), so that editors can save articles, custom
HTML modules, categories and the offline message without being blocked by
OWASP CRS (CRS).

The exclusions are deliberately narrow:

* only `POST` requests to `<path>administrator/index.php`,
* only when a Joomla session cookie is present (a cookie whose name is a
  32 character hexadecimal hash and whose value looks like a session ID),
* only for the content field of the screen (`option` in the query string):

| Screen | `option` | Field |
|---|---|---|
| Articles | `com_content` | `jform[articletext]` |
| Modules (custom HTML) | `com_modules` | `jform[content]` |
| Categories | `com_categories` | `jform[description]` |
| Global configuration | `com_config` | `jform[offline_message]` |

* and only from the XSS rules (tag `attack-xss`) and rule 932130, which
  matches `>(` in ordinary HTML text (for example
  `<strong>Note</strong>(see below)`) as a Unix process substitution.

All other fields, all other rules (SQL injection, remote command execution,
PHP injection and so on), GET requests and the front end stay fully checked.

The session cookie condition is not a security boundary: Joomla checks the
session and the CSRF token itself. It only keeps the exclusions away from
requests that do not look like a logged-in editor. Because the cookie is
matched by its shape and not by its exact name, nothing has to be changed
when the Joomla `secret` changes or a site is added.

The plugin has been written for Joomla 3.x (tested with 3.10 / 3.17). Other
screens and third-party extensions (for example editors that post their own
fields) are not covered and must be tuned with
[custom rule exclusions](https://coreruleset.org/docs/concepts/false_positives_tuning/).

## Installation

For full and up to date instructions for the different available plugin
installation methods, refer to [How to Install a Plugin](https://coreruleset.org/docs/concepts/plugins/#how-to-install-a-plugin)
in the official CRS documentation.

### Joomla in a sub-directory

If Joomla is not installed at the document root, set
`tx.joomla-rule-exclusions-plugin_path` in `joomla-rule-exclusions-config.conf`
to the installation path. It must start and end with `/`:

```apache
SecAction \
    "id:9532020,\
    phase:1,\
    pass,\
    nolog,\
    setvar:'tx.joomla-rule-exclusions-plugin_path=/cms/'"
```

The config file also shows how to set the path per host.

### Conditionally enable plugins for multi-application environments

For full and up to date instructions on how to conditionally enable/disable
this plugin on a multisite environment, please refer to
[Conditionally enable plugins for multi-application environments](https://coreruleset.org/docs/concepts/plugins/#conditionally-enable-plugins-for-multi-application-environments)
in the official CRS documentation.

## Testing

The regression tests in `tests/regression` use the
[go-ftw](https://github.com/coreruleset/go-ftw) format and run with the
[CRS plugin test action](https://github.com/coreruleset/crs-plugin-test-action)
on Apache (ModSecurity v2) and nginx (libmodsecurity v3).

## License

Copyright (c) 2026 futsure0 and contributors.

The plugin is distributed under Apache Software License (ASL) version 2.
Please see the enclosed LICENSE file for full details.
