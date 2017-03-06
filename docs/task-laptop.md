# Task laptop

## Configuration notes

Configuration of experiments currently happens in a few different places for
PyEPL-based experiments. For every task type (e.g., FR), there is a main
configuration file (`config.py`) and an additional version-specific file (e.g.,
`FR1_config.py`) which can override variables set in the main `config.py` or
define new ones. Other options are given on the command line and passed along to
the experiment via the `RAM_CONFIG` environment variable.

!!! note

    When migrating away from PyEPL, configuration should be streamlined to be
    less spread out!
