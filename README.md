# Introduction

`mkdot` is a Python script that generates user/system configuration files ("dotfiles") from Jinja2 templates. It is designed to work equally-well as both an interactive "on-demand" script with useful stdout/stderr messages and a hands-off automation-ready script with filesystem logging.

## Features

* Optional file logging.
* "Dry Run" support for generating dotfiles without saving changes.
* Color output (with option to disable).
* Backup of existing dotfiles prior to writing changes.
* Default behavior configuration via environment variables.
* Ability to quickly revert breaking changes.

## System Requirements

* Python 3
* [PyYAML](https://pyyaml.org/)
* [Jinja2](http://jinja.pocoo.org/docs/2.10/)
* `rsync` (used for transferring files locally)

## Important Notes

* `mkdot` runs the underlying `rsync` process _with_ the `--delete` flag when taking a backup of the output directory but _without_ the `--delete` flag when writing the newly-generated dotfiles to the output directory, _unless_ the `--delete` option is passed to the script.
* Running `mkdot` with `--dry-run` still writes the generated dotfiles to the specified or default working directory. This is so you can manually inspect the files afterward.

## Known Bugs & Potential Issues

* The `mkdot`-provided Jinja function `require` does not currently validate elements of variables more than one element deep. In other words, `require('foo.bar')` will work, but `require('foo.bar.baz')` will not.



----
# Usage

`mkdot` is invoked with two required arguments: a template configuration (YAML) file, and a template source directory. With regards to this repo, the simplest example would be:

```
$ ./mkdot example/example.yaml example/templates
```

If a directory is supplied as the first argument, then `mkdot` will automatically select a file called `mkdot.yaml` underneath that directory, if it exists. Otherwise, it will select the file whose name (without the `.yaml` extension) is closest to the hostname of the executing machine.

## CLI Arguments

The following table describes the remaining optional arguments:

| Argument(s)                 | Description                                                                                                                                                                                                    |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--block-end-string`        | Specifies the string marking the end of a Jinja template block.                                                                                                                                                |
| `--block-start-string`      | Specifies the string marking the start of a Jinja template block.                                                                                                                                              |
| `--command-end-string`      | Specifies the string marking the end of a Jinja template comment.                                                                                                                                              |
| `--comment-start-string`    | Specifies the string marking the start of a Jinja template comment.                                                                                                                                            |
| `--delete`                  | Specifies that the script should delete any files in the output directory that are not part of the generated dotfiles. +                                                                                       |
| `--dont-trim-blocks`        | Specifies that the first newline character after a Jinja block should NOT be removed.                                                                                                                          |
| `-d`, `--dry-run`           | Specifies that the script should only execute a dry-run, preventing the generated dotfiles from being copied from the working directory to the output directory.                                               |
| `--exclude`                 | Specifies an additional list of files or directories relative to the specified output directory that should be preserved on write (if `--delete` is supplied).                                                 |
| `--fqdn`                    | Specifies that the given string should be used to set the provided `fqdn` and `hostname` Jinja variables, instead of the FQDN of the executing machine.                                                        |
| `-h`, `--help`              | Displays help and usage information.                                                                                                                                                                           |
| `-f`, `--log-file`          | Specifies a log file to write to in addition to stdout/stderr.                                                                                                                                                 |
| `-l`, `--log-level`         | Specifies the log level of the script. This option is ignored if `--log-file` is not specified.                                                                                                                |
| `-m`, `--log-mode`          | Specifies whether to `append` or `overwrite` the specified log file. This option is ignored if `--log-file` is not specified.                                                                                  |
| `--no-backup`               | Specifies that the script should not perform a backup of the specified output directory prior to writing the generated dotfiles.                                                                               |
| `--no-color`                | Disables color output to stdout/stderr.                                                                                                                                                                        |
| `-o`, `--output`            | Specifies the output directory of the generated dotfiles.                                                                                                                                                      |
| `--revert`                  | Restores the contents of an existing backup directory relative to the specified output directory. If a path is provided alongside this flag, that location will instead be utilized as the restoration source. |
| `--rsync-executable`        | Specifies a file path to the rsync executable utilized for transferring directories.                                                                                                                           |
| `-r`, `--run`               | Specifies a command to run after the newly generated dotfiles are written to the output directory. This command is not run if `--dry-run` is specified.                                                        |
| `--variable-end-string`     | Specifies the string marking the end of a Jinja template variable.                                                                                                                                             |
| `--variable-start-string`   | Specifies the string marking the start of a Jinja template variable.                                                                                                                                           |
| `-w`, `--working-directory` | Specifies the working directory.                                                                                                                                                                               |

\+ Essentially this enables the `--delete` flag of the underlying rsync process used to sync the working and output directories.

Each of the above options has the following set of corresponding value types and default values:

| Argument(s)                 | Value Type / Possible Values | Default Value    |
|-----------------------------|------------------------------|------------------|
| `--block-end-string`        | Generic String               | `%}`             |
| `--block-start-string`      | Generic String               | `{%`             |
| `--command-end-string`      | Generic String               | `#}`             |
| `--comment-start-string`    | Generic String               | `{#`             |
| `--delete`                  |                              |                  |
| `-d`, `--dry-run`           |                              |                  |
| `--exclude`                 | Directory/File Paths         |                  |
| `--fqdn`                    | Fully-Qualified Domain Name  |                  |
| `-h`, `--help`              |                              |                  |
| `-f`, `--log-file`          | File Path                    |                  |
| `-l`, `--log-level`         | `info` or `debug`            | `info`           |
| `-m`, `--log-mode`          | `append` or `overwrite`      | `append`         |
| `--no-backup`               |                              |                  |
| `--no-color`                |                              |                  |
| `-o`, `--output`            | Directory Path               | `~/.config`      |
| `--revert`                  | Directory Path (Optional)    |                  |
| `--rsync-executable`        | File Path                    | `/usr/bin/rsync` |
| `-r`, `--run`               | Shell Command                |                  |
| `--variable-end-string`     | Generic String               | `}}`             |
| `--variable-start-string`   | Generic String               | `{{`             |
| `-w`, `--working-directory` | Directory Path               | `/tmp/mkdot`     |


## Script Output

The output of a typical run may look something like this on stderr/stdout:

```
~/projects/dotfiles/mkdot  >  mkdot yaml/mkdot-i3.yaml templates --no-color                                           
:: Validating working environment...
  --> Validating rsync executable path...
  --> Validating template source directory...
  --> Validating template configuration file...
  --> Validating working directory...
:: Loading template configuration file...
  --> Reading template configuration file...
  --> Parsing template configuration file...
  --> Validating template configuration...
:: Setting-up templating environment...
  --> Initializing loader...
  --> Initializing environment...
  --> Initializing extensions...
:: Copying additional files...
  --> misc/cursor.theme
  --> x/exports.sh
  --> misc/user-dirs.dirs
  --> zsh/exports.zsh
  --> zsh/prompt.zsh
  --> zsh/options.zsh
  --> zsh/aliases.zsh
  --> scripts/wal-set.sh
  --> scripts/scrot-full.sh
  --> scripts/rofi-network.sh
  --> scripts/rofi-exit.sh
  --> scripts/rofi-run.sh
  --> scripts/install-arch.sh
  --> scripts/lock.sh
  --> scripts/ssh-wrapper.sh
  --> scripts/i3-restart-wrapper.sh
  --> scripts/rofi-ssh.sh
  --> scripts/setup-monitors.sh
  --> scripts/rofi-lpass.sh
  --> scripts/scrot-select.sh
  --> scripts/polybar-start.sh
  --> scripts/mkdot-apply.sh
  --> scripts/i3-startup.sh
  --> wal/templates/colors-rofi.rasi
  --> wal/templates/colors-emacs.el
  --> wal/wallpapers/cups.jpg
  --> wal/wallpapers/hyper-light-drifter.png
  --> wal/wallpapers/yesteryear.png
  --> wal/wallpapers/red-landscape.jpg
  --> wal/wallpapers/firewatch.png
  --> wal/wallpapers/project-legion.jpg
  --> wal/wallpapers/jealousy.png
  --> wal/wallpapers/hexed.png
  --> wal/wallpapers/dock.jpg
  --> wal/wallpapers/cold-shoulder.png
  --> wal/wallpapers/clouds-fixed.jpg
  --> wal/wallpapers/cocoon.png
  --> wal/wallpapers/quarentine.png
  --> wal/wallpapers/stream.jpg
  --> wal/wallpapers/framework.png
  --> wal/wallpapers/pale-green-dot.png
  --> wal/wallpapers/mountains2.jpg
:: Translating templates...
  --> rofi/config
  --> trizen/trizen.conf
  --> dunst/dunstrc
  --> zsh/zshrc.sh
  --> polybar/config
  --> x/initrc.sh
  --> x/resources.conf
  --> i3/config
:: Finalizing configuration process...
  --> Creating backup of existing dotfiles...
  --> Writing dotfiles to output directory...
  --> Creating symlinks...
```

The corresponding log file for the above run looks like this:

```
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.main] Started configuration process.
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.validate_environment] Validating working environment...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.parse_yaml_config] Loading template configuration file...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.setup_jinja] Setting-up templating environment...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying additional files...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/misc/cursor.theme" to "/tmp/mkdot/misc/cursor.theme"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/x/exports.sh" to "/tmp/mkdot/x/exports.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/misc/user-dirs.dirs" to "/tmp/mkdot/misc/user-dirs.dirs"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/zsh/exports.zsh" to "/tmp/mkdot/zsh/exports.zsh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/zsh/prompt.zsh" to "/tmp/mkdot/zsh/prompt.zsh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/zsh/options.zsh" to "/tmp/mkdot/zsh/options.zsh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/zsh/aliases.zsh" to "/tmp/mkdot/zsh/aliases.zsh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/wal-set.sh" to "/tmp/mkdot/scripts/wal-set.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/scrot-full.sh" to "/tmp/mkdot/scripts/scrot-full.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/rofi-network.sh" to "/tmp/mkdot/scripts/rofi-network.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/rofi-exit.sh" to "/tmp/mkdot/scripts/rofi-exit.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/rofi-run.sh" to "/tmp/mkdot/scripts/rofi-run.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/install-arch.sh" to "/tmp/mkdot/scripts/install-arch.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/lock.sh" to "/tmp/mkdot/scripts/lock.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/ssh-wrapper.sh" to "/tmp/mkdot/scripts/ssh-wrapper.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/i3-restart-wrapper.sh" to "/tmp/mkdot/scripts/i3-restart-wrapper.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/rofi-ssh.sh" to "/tmp/mkdot/scripts/rofi-ssh.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/setup-monitors.sh" to "/tmp/mkdot/scripts/setup-monitors.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/rofi-lpass.sh" to "/tmp/mkdot/scripts/rofi-lpass.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/scrot-select.sh" to "/tmp/mkdot/scripts/scrot-select.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/polybar-start.sh" to "/tmp/mkdot/scripts/polybar-start.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/mkdot-apply.sh" to "/tmp/mkdot/scripts/mkdot-apply.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/scripts/i3-startup.sh" to "/tmp/mkdot/scripts/i3-startup.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/templates/colors-rofi.rasi" to "/tmp/mkdot/wal/templates/colors-rofi.rasi"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/templates/colors-emacs.el" to "/tmp/mkdot/wal/templates/colors-emacs.el"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/cups.jpg" to "/tmp/mkdot/wal/wallpapers/cups.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/hyper-light-drifter.png" to "/tmp/mkdot/wal/wallpapers/hyper-light-drifter.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/yesteryear.png" to "/tmp/mkdot/wal/wallpapers/yesteryear.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/red-landscape.jpg" to "/tmp/mkdot/wal/wallpapers/red-landscape.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/firewatch.png" to "/tmp/mkdot/wal/wallpapers/firewatch.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/project-legion.jpg" to "/tmp/mkdot/wal/wallpapers/project-legion.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/jealousy.png" to "/tmp/mkdot/wal/wallpapers/jealousy.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/hexed.png" to "/tmp/mkdot/wal/wallpapers/hexed.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/dock.jpg" to "/tmp/mkdot/wal/wallpapers/dock.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/cold-shoulder.png" to "/tmp/mkdot/wal/wallpapers/cold-shoulder.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/clouds-fixed.jpg" to "/tmp/mkdot/wal/wallpapers/clouds-fixed.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/cocoon.png" to "/tmp/mkdot/wal/wallpapers/cocoon.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/quarentine.png" to "/tmp/mkdot/wal/wallpapers/quarentine.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/stream.jpg" to "/tmp/mkdot/wal/wallpapers/stream.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/framework.png" to "/tmp/mkdot/wal/wallpapers/framework.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/pale-green-dot.png" to "/tmp/mkdot/wal/wallpapers/pale-green-dot.png"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.copy_files] Copying "templates/wal/wallpapers/mountains2.jpg" to "/tmp/mkdot/wal/wallpapers/mountains2.jpg"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating templates...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/rofi/config.template" into "/tmp/mkdot/rofi/config"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/trizen/trizen.template" into "/tmp/mkdot/trizen/trizen.conf"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/dunst/dunstrc.template" into "/tmp/mkdot/dunst/dunstrc"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/zsh/zshrc.template" into "/tmp/mkdot/zsh/zshrc.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/polybar/config.template" into "/tmp/mkdot/polybar/config"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/x/initrc.template" into "/tmp/mkdot/x/initrc.sh"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/x/resources.template" into "/tmp/mkdot/x/resources.conf"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.translate_templates] Translating "templates/i3/config.template" into "/tmp/mkdot/i3/config"...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.write_output] Finalizing configuration process...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.write_output] Creating backup of existing dotfiles...
[INF] [12/04/2018 11:15:26 AM] [10013] [mkdot.write_output] Writing dotfiles to output directory...
[INF] [12/04/2018 11:15:27 AM] [10013] [mkdot.write_output] Creating symlinks...
[INF] [12/04/2018 11:15:27 AM] [10013] [mkdot.main] Configuration process complete.
```

Notice that the log file follows the following format:

```
[LOG_LEVEL] [TIMESTAMP] [PROCESS_ID] [MODULE.FUNCTION] MESSAGE
```


## Exit Codes

The script not only returns non-zero exit codes on fatal errors, but even broadly categorizes them:

| Code | Description                                                                                                    |
|------|----------------------------------------------------------------------------------------------------------------|
| 0    | Script successfully ran, although perhaps with warnings.                                                       |
| 1    | Generic issue prior to the environment validation step (invalid arguments, import exceptions, etc).            |
| 2    | Issue during the environment validation step.                                                                  |
| 3    | Issue with loading, parsing, or validating the template configuration file.                                    |
| 4    | Issue with instantiating or configuring the Jinja templating environment.                                      |
| 5    | Issue with translating the source templates or saving them to the working directory.                           |
| 6    | Issue with performing the backup of the existing dotfiles or writing the new dotfiles to the output directory. |
| 7    | Issue occurring during a `--revert` operation.                                                                 |
| 100  | Script was interrupted via CTRL+C or CTRL+D.                                                                   |


## Environment Variables

The default behavior of the `mkdot` script may also be configured via environment variables. Each environment variable has an associated command line argument, as described in the table below:

| Environment Variable      | Corresponding CLI Argument |
|---------------------------|----------------------------|
| `MKDOT_BLOCK_END_STR`     | `--block-end-string`       |
| `MKDOT_BLOCK_START_STR`   | `--block-start-string`     |
| `MKDOT_COMMENT_END_STR`   | `--comment-end-string`     |
| `MKDOT_COMMENT_START_STR` | `--comment-start-string`   |
| `MKDOT_EXCLUDE`           | `--exclude`                |
| `MKDOT_FQDN`              | `--fqdn`                   |
| `MKDOT_LOG_FILE`          | `--log-file`               |
| `MKDOT_LOG_LVL`           | `--log-level`              |
| `MKDOT_LOG_MODE`          | `--log-mode`               |
| `MKDOT_OUTPUT`            | `--output`                 |
| `MKDOT_RSYNC_PATH`        | `--rsync-executable`       |
| `MKDOT_RUN`               | `--run`                    |
| `MKDOT_VAR_END_STR`       | `--variable-end-string`    |
| `MKDOT_VAR_START_STR`     | `--variable-start-string`  |
| `MKDOT_WORKING_DIR`       | `--working-directory`      |

As an example, if `MKDOT_WORKING_DIR` was set to `/tmp/foo` but `-w /tmp/bar` was passed to `mkdot` via command-line, then the script would use the value of `/tmp/bar` for the working directory.


## Handling Failure Cases

In the case that a falure occurs prior to the finalization step (in other words, a failure case that does not produce an exit code of `6`), it can be assumed that the existing dotfiles have not been corrupted and thus no restoration of a backup is necessary. However in the event that `mkdot` encounters a critical issue during the finalization step, the user may quickly revert any changes by restoring the dedicated backup directory by executing:

```
$ mkdot --revert
```

In additon to restoring dotfiles from the dedicated backup directory, the `--revert` option can also be specified in conjection with a specific source directory. The following example would restore dotfiles from `/foo/bar/2018.backup`:

```
$ mkdot --revert /foo/bar/2018.backup
```



----
# Templating Layout

See `TEMPLATING.md` within this repo for further information on designing dotfile templates to work with `mkdot`.
