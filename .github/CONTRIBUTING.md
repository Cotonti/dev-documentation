Notes for contributors
======================

As an open source project, Cotonti welcomes contributions of many forms.
Do you have a new cool feature that you'd like to contribute to Cotonti? Or a fix for a bug? Great! The Cotonti project loves code contributions, improvements, and bugfixes, but we require that they follow a set of guidelines and that they are contributed in a specific way.
The rules below are not guidelines or recommendations, but strict rules. Contributions to Cotonti generally *may not be accepted* if they do not adhere to these rules.
Not all existing code follows these rules, but all new code is expected to

Common rules
------------

* **Learn** a [coding standards](https://www.cotonti.com/docs/devel/coding_style) should be used for our project
* Do not bundle commits that are unrelated to each other -- create separate pull requests instead.
* Do not submit a commit not attached to particular issue.
* Use commit description for brief issue info (english language only).
* Write a descriptive pull request message. Explain the advantages and disadvantages of your proposed changes.
* Before starting to work on a major contribution, discuss your idea with experienced Cotonti programmers (e.g., core team members) to avoid wasting time on things that have no chance of getting merged into Cotonti.

Contributing extension changes
------------------------------

* While you makes a change or bugfix to any extension (plugin, module, theme), please bump a version number of it manually.
* If you use minified version of some JavaScript code — include link to it's official page or source file itself.

Contributing or improving translations
--------------------------------------

Language files can be updated standard way via committing and pull requests. But preferred way is to use [Transifex](https://www.transifex.com/projects/p/cotonti/) instead. Notify a project manager for Cotonti on Transifex in case you need assistance.
Note: Since version `0.9.13` Cotonti language files are compatible with Transifex service. Here you can found [discussion](http://www.cotonti.com/forums?m=posts&q=7367) about it, and there are [documentation and how-to’s](http://www.cotonti.com/docs/ext/lang/transifex).

_Note_: In translations, please **do not**:
* Insert newlines in strings you are translating.
* Translate placeholder strings such as {$user_name}, {$count}, etc.. Leave those as they are.
