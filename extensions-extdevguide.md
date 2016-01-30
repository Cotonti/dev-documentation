Introduction to extension development
=====================================

> Here we provide addition info for actualization original article with same name, located here:
https://www.cotonti.com/en/docs/ext/extensions/extdevguide

> Information consider «Setting up your extension» section, and will be actual  only in case of adoption `versioning` Cotonti package, described here
([см.описание тут](https://github.com/macik/cotonti-core-versioning)).
Also it would involves `Extension` API and `admin` module improvement.

Structure of setup file
-----------------------

Here we post only additions related to package requirements resolving theme.

### Current state

As for now (Siena `0.9.18`) Cotonti provide requirements checking only with its name (by `Requires_modules` or `Requires_plugins`) directive.

```
Requires_modules=page
```
However we get lack of these abilities: 
 - set certain version of needed Extension;
 - set and check required version of Cotonti core;
 - can not set requirement for certain PHP extension
 
### New Features

New `versioning` API proposed more flexible requirement conditions:
* define type of required Extension: `php`, `core`,`plugin`,`module`,`theme`,`admin_theme`,`system`
* define certain Extension version `1.2.3` 
* define boundary for version check: `>1.2.0` , `<=2.0.0`
* define range with `*` wildcards: `1.5.*`
* define «soft» requirement, when we check extension exists in the system but not required to be installed (see examples below)
* we can use short version tags or version tags with pre-release info: `1.7`, `1.*`, `1.2beta` 

### Примеры:
Here the list of examples to form requirement rules (it's not a real life cases but listed for demonstration purposes only):

```ini
# Extension required Cotonti v0.9.19 or newer.
Requires_core= >=0.9.19  
# shorthand for same rule
Requires= >=0.9.19  
```

```ini
# extension require Page module with version 1.1.1 (and not others)
Requires_page_module = 1.1.1
```

```ini
# PFS module at least 0.2.0 version should be in system but it installation is optional
Requires_pfs_module = >0.2.0?
```

```ini
# Wildcard use (suppose pm VERSION >=0.2.0, но <0.3.0 ) 
Requires_pm_module = 0.2.*
```

```ini
# Required `Nemesis` theme to be installed and selected as default
Requires_nemesis_theme = *
# `Skeletonti` theme must be located in themes folder (no matted selected or not)
Requires_skeletonti_theme = *?
```

```ini
# Extension needs PHP `curl`, version 7  
Requires_php_curl = 7.*
```

```ini
# Ext required `bcmath` extension
# as it PHP native extension without own version, we set `*` to allow any 
Requires_php_bcmath = *
```

```ini
# No comments — PHP 5.4 only
Requires_php = 5.4
```

```ini
# we required `anyext` extension
# If we do not define its type (module|plugin) so, both types will be checked for entire extension
Requires_anyext = 1.0
```

```ini
# we can use version tag with pre-release version 
Requires_some_ext3 = >=1.0.0-RC3
# in this case `1.0.0-beta` not satisfies requirements, but `1.0.0-RC4` or `1.0.0` should
```

```ini
# complex version tags also allowed
Requires_ckeditor = >1.0.0-3.5.7
# here `3.5.7` part treats as pre-release tag
```

```ini
# complex constraints combined several requirements can be defined with several lines
Requires_ckeditor = >=1.0.0
Requires_ckeditor = <1.5
```

