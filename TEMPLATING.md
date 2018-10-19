# Templating Layout

----
## Provided Jinja Expressions

`mkdot` provides several useful additional variables and functions that are available to all dotfile templates.

### Provided Variables

| Variable   | Description                                                     |
|------------|-----------------------------------------------------------------|
| `file`     | The name of the current template _output file_.                 |
| `fqdn`     | The fully-qualified domain name of the machine running `mkdot`. |
| `hostname` | The hostname of the machine running `mkdot`.                    |

For all templates, the `this` variable is set to the relevant `templates` list item (see _Template Configuration File_). As a side note, since `this.file` has the same value as `file`, it's probably shorter to just use `file` everywhere.

### Provided Functions

| Function        | Description                                                                                                    |
|-----------------|----------------------------------------------------------------------------------------------------------------|
| `domain_join`   | Joins each argument as part of a domain. Calls `return '.'.join([x.strip('.') for x in args])` under the hood. |
| `file_ext`      | Returns the full file extension of the specified path.                                                         |
| `file_name`     | Returns the value of `os.path.basename` but without the file extension.                                        |
| `get_host`      | Returns the reverse record for the specified IP address.                                                       |
| `git_ip`        | Returns the IPv4 address associated with the specified host string.                                            |
| `get_output`    | Returns the output of the specified command (both STDOUT and STDERR).                                          |
| `path_basename` | Calls `os.path.basename` on the specified string.                                                              |
| `path_dirname`  | Calls `os.path.dirname` on the specified string.                                                               |
| `path_join`     | Calls `os.path.join` on the specified strings.                                                                 |
| `print`         | Prints the specified string during the templating process.                                                     |
| `raise`         | Raises an exception during the templating process with the specified string.                                   |
| `require`       | Raises an exception if any of the arguments are not present as variables in the current context.               |


----
## Template Configuration File


The template configuration file (`example/mkdot.yaml`) is a YAML file which contains the global variables accessible to the various configuration templates. In addition to being utilized by the Jinja templates themselves, `mkdot` also directly acts on certain required variables within the file. Currently, this amounts to the `templates` variable.

### `templates` Variable

The `templates` variable defines a list of dictionaries specifying the input-output mapping of template files to render. The name of each output file is specified by the required `file` key. By default, the `file` key assumes that the input file path (relative to the template source directory) is the same as the output file path (relative to the output directory). However, if the optional sibling `template` key is specified, then the input file is taken to be the value of the `template` key, and the output file is taken to be the value of the `file` key. Note that the value of the `file` and `this.file` Jinja variables will always be the value of the `file` key in the `templates` specification. Basic globbing for the value of the `file` key is supported by Python's `glob` module, however globbing is not supported if a specification supplies values to both the `file` and `template` keys. An example `templates` specification is given below:

```
templates:
    # Translate "i3/conf" from the source directory into "i3/conf" in the output
    # directory.
    - file: "i3/conf"

    # Include all ".sh" files within the "bash" subdirectory of the specified
    # template directory.
    - file: "bash/*.sh"
    
    # Import foo/baz.template and save it as foo/bar.conf
    - file: "foo/bar.conf"
      template: "foo/baz.template"
```
