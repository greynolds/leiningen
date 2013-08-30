# Tasks

## Task-specific options

NB: The Leiningen source sometimes uses "key" instead of "option".
I.e. configuration *options* are implemented as Clojure map *keys*.

Syntax: context:option

Terminology:  key = option

### Task: compile

 * project:source-paths
 * project:compile-path
 * project:java-source-paths
 * project:test-paths
 * project:test-selectors
 * project:test-selectors:default
 * project:monkypatch-clojure-test
 * project:clean-non-project-classes
 * project:class-file-whitelist
 * project:prep-tasks

 * project:profiles:leiningent/test
 * project:profiles:test

### install

 * project:install-releases?
 * project:local-repo
 * project:group
 * project:name
 * project:version

### Task: jar

(def whitelist-keys
  "Project keys which don't affect the production of the jar should be
propagated to the compilation phase and not stripped out."
  [:offline? :local-repo :certificates :warn-on-reflection :mirrors])

 * project:root
 * project:manifest
 * project:main
 * project:jar-exclusions
 * project:group
 * project:name
 * project:filespecs
 * project:source-paths
 * project:java-source-paths
 * project:version
 * project:jar-name
 * project:uberjar-name
 * project:aot
 * project:standalone
 * project:profiles

 * profile:default

 * (meta (:main project)):skip-aot

 * :path?
 * :bytes?
 * :classifier
 * :extension

Task jar looks for readme and license files using regexes:

 * #"^(?i)readme"
 * #"^(?i)license"

#### filespecs

I don't understand how this works.

:filespecs =  ;; Add arbitrary jar entries. Supports :path, :paths, :bytes, and :fn types.

filespec fn seems to concat map containing :paths of the following forms:

 * "META-INF/maven/<group>/<name>pom.xml"
 * "META-INF/maven/<group>/<name>pom.xml"
 * "META-INF/maven/<group>/<name>/pom.properties"
 * "META-INF/leiningen/<group>/<name>/project.clj"

It concats these to project:filespecs.

### Task repl, trampoline-repl

### Task: uberjar

 * project:standalone
 * project:jar-inclusions
 * project:uberjar-inclusions
 * project:dependencies
 * profile:uberjar
 * profile:default

jar/whitelist-keys?

"META-INF/plexus/components.xml"???