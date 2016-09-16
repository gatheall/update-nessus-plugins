## Introduction

[Nessus](http://www.tenable.com/products/nessus-vulnerability-scanner) is a high-quality vulnerability scanner.  One of its main advantages is its extensive and continually evolving plugin database of vulnerability checks.  Nessus comes with a shell script to retrieve the latest set of plugins from a central repository -- *nessus-update-plugins* -- and I would imagine most Nessus users run this fairly often to keep their plugins up-to-date.

While the script does at fine job, it does have a couple of limitations in my opinion: it does not backup existing plugins, it doesn't attempt to validate new or changed plugins, and it doesn't provide much information about changes.  I designed *update-nessus-plugins* to address these limitations and to call *nessus-update-plugins* to do the actual updates so as to avoid any problem of compatibility.

*update-nessus-plugins* is written in Perl and calls *nessus-update-plugins* to update the plugins as well as [describe-nessus-plugin](https://github.com/gatheall/describe-nessus-plugin/) to obtain descriptive information about new / changed plugins.  It should work on any unix-like system with Perl 5 or better (Perl 5.005 if you choose to generate summary reports).  It also requires the following Perl modules:

* `Algorithm::Diff`
* `Archive::Tar`
* `Carp`
* `Digest::SHA`
* `File::Find`
* `Getopt::Long`
* `IO::Zlib` (used by `Archive::Tar` to output file in compressed form)
* `POSIX`

If your system does not have these modules installed already, visit [CPAN](http://search.cpan.org/). Note that `Algorithm::Diff`, `Archive::Tar`, and `IO::Zlib` are not included with Perl distributions and that `Digest::SHA` is not included with Perl distributions prior to 5.8.0 so you may need to install them yourself.


## Installation</H3>

* Retrieve [the script](update-nessus-plugins) and save it locally.
* Verify ownership and permissions on the script - it will need to run as root to run *nessus-update-plugins*.
* You may wish to edit the script to adjust the location of the Perl interpreter in the first line and to initialize variables such as `$lang` and `$plugins_dir` to suit your environment and tastes.
* Ensure you have working versions of *nessus-update-plugins* and, if you wish to do summary reports, [describe-nessus-plugin](https://github.com/gatheall/describe-nessus-plugin/).


## Use</H3>

There are several commandline arguments you can use to override variables defined in the script itself:

| Option | Meaning |
| ------ | ------- |
| -b, --backup | Create a backup of existing plugins first. |
| -d, --debug | Display debugging messages while running.  *Note: this will still update the plugins!* |
| -i, --ignores | Ignore the specified files found in the plugins directory from the summary report and parse check, overriding `@ignores`. |
| -p, --parse | Parse new / changed plugins and report whether errors exist. |
| -s, --summary | Print a summary report of new / changed plugins. |


Notes:

* Commandline arguments take precedence over variables defined in the script.
* Enabling the debug option will not prevent plugins from being updated.
* Specifying a file either in `@ignores` or with the `-i` option won't prevent it from being updated by *nessus-update-plugins*, just from being included in any summary report and parse check. It will, however, be included in the backup, if one is made.
* Multiple files to ignore can be specified on the commandline either by a comma-delimited string or by multiple argument pairs. For example, `-i "MD5,README"` is equivalent to `-c MD5 -c README`.


Examples:

| Invocation | Meaning |
| ---------- | ------- |
| `update-nessus-plugins` | updates nessus plugins. Note: this does nothing more than call *nessus-update-plugins*. |
| `update-nessus-plugins -b` | creates a backup of old plugins and updates plugins. |
| `update-nessus-plugins -bs` | creates a backup, updates plugins, and prints a summary of new / changed plugins. |
| `update-nessus-plugins -p` | updates plugins and parses new / changed plugins for errors. |


## Known Bugs and Caveats</H3>

Currently, I am not aware of any bugs in this script.

You need a working version of *nessus-update-plugins* to use this script.

If you wish to generate summary reports of new / changed plugins, you also need [describe-nessus-plugin](https://github.com/gatheall/describe-nessus-plugin/) version 2.01 or better.

If you encounter an error similar to `Insecure dependency in chdir while running with -T switch at /usr/lib/perl/5.00503/File/Find.pm line 133` when trying to run *update-nessus-plugins*, it's likely that you're using an older version of the Perl module `File::Find`.  Versions distributed with Perl versions prior to 5.6.0 don't support the `no_chdir` option, which is used in this script to avoid problems with taint checks.  The solution is to either upgrade `File::Find`, upgrade Perl itself, or disable taint checks (ie, remove the `-T` option on the first line of the script).

The option for parsing new / changed plugins requires that the NASL interpreter support the `-p` option, which was introduced with Nessus version 2.0.7.

If you encounter a problem with this script, I encourage you to rerun it in debug mode (eg, add `-d` to your commandline) and examine the resulting output before contacting me.  Often, this will enable you to resolve the problem by yourself.


## Copyright and License

Copyright (c) 2003-2016, George A. Theall.
All rights reserved.

This script is free software; you can redistribute it and/or modify it under the same terms as Perl itself.
