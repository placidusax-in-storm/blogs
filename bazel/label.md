# Label

The name of a [[target]] is called its label.

Every label uniquely identifies a target.

A typical label in canonical form looks like:
`@myrepo//my/app/main:app_binary`

`@repository` (`myrepo//`) + ` un-qualified package` (`my/app/main`) + un-qualified target-name (:app_binary).

The name of a file target in a subdirectory of the package is the file's path
relative to the package root (the directory containing the BUILD file).
