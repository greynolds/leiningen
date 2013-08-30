# Leiningen Features

## Configuration & Customization

## Tasks

## Checkouts

Sometimes you need to develop two projects in parallel, where one
depends on the other; for example, you're developing a library and an
application that uses the library.  Normally, for every change you
make in the library you would need to run `lein install`, then switch
to the app and restart the repl.  This is annoying.

The :checkouts option allows you to define a dependency on a library
such that leiningen will look in the dependency's project tree for
resources rather than ~/.m2/repository or other repo.

This option is a little different than most of the other options,
because it also requires that you create a `checkouts` subdirectory in
the root directory of the dependent project.

The name "checkouts" is thus a bit of a misnomer.  Think of it as a
kind of :local-dependencies option.

## Plugins

## Templates

