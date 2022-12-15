# Template Variable Substitution

When doing a `stacker build`, the behavior of stacker is specified by the yaml
directives below. In addition to these, stacker allows variable substitions of
several forms. For example, a line like:

    $ONE ${{TWO}} ${{THREE:3}}

When run with `stacker build --substitute ONE=1 --substitute TWO=2` is
processed in stacker as:

    1 2 3

That is, variables of the form `$FOO` or `${FOO}` are supported, and variables
with `${FOO:default}` a default value will evaluate to their default if not
specified on the command line. It is an error to specify a `${FOO}` style
without a default; to make the default an empty string, use `${FOO:}`.
