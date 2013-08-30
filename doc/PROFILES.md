# Profiles

## Disquisition:  Configuration & Customization (C&C)

Leiningen's `:profile` option is designed to address the problem of
customization.  The general idea is that every tool (in this case,
Leiningen) comes with a default configuration.  The default tool
configuration is effectively global - it is in effect by default
whenever and wherever the tool is used.  Different systems (or
"installations") may have different needs, so they need to be able to
customize the configuration by overriding default settings (e.g. by
adding a search path).  Within each system, different workgroups may
have different needs or policies, so they too want to be able to
refine the system settings.  Within workgroups, individual developers
(notoriously) may have their own ideas about proper Theology and
Geometry, so they want to set their own configuration.  Finally,
specific projects may call for customizations as well.  Per-project
configuration defaults may cut across organizational levels (multiple
organizations and/or groups may work on one project - e.g. Leiningen),
but since each developer works on private copies of projects,
per-developer per-project customization support may be needed as well.

But that's not all.  The various phases of development - dev, unit
test, system test, etc. - may also require distinct configurations;
these will need to apply across organization levels, and may also
require customization.  So we actually have two, orthogonal lines of
configuration and customization: one cuts across organizations and
individuals, and the other cuts across activity types.

For example, company policy might stipulate that all developers on all
projects use some kind of quality assurance tool (for example,
something that enforces code formatting rules); to enforce this, a
configuration option for Leiningen may be set at the system level.  A
particular workgroup might decide to use some sort of a tool or
configuration option that other groups do not want to use; this can be
supported by designating distinct (but shared within group)
configuration files for each group.  Individual developers will
commonly have their own preferences for certain things, which they can
specify using `~/.lein/profiles.clj`, which has the effect of enabling
them for every use of Leiningen by that user.  For a specific project,
a specific developer may need some custom configuration, which can be
set in the project.clj.  Each of these customizations may involve
overrides and/or extensions.  Furthermore distinct customization
structures may be called for for e.g. dev and test phases.

(We can add another "axis of configuration".  The organization-based
config/customize system described above is project-independent.  In
the end, the result of config/customize is a particular configuration
for a particular use of the tool; as described, this would be a
general config.  But we work on projects, so we need project-specific
configs as well.  The project.clj stored with the source in the
repository represents a global configuration.  It can be customized by
settings at the system and user levels, by setting options in config
files.  But those options would then apply across projects, since we
have no way of setting project-specific options except in project.clj.
But each developer works on a private copy of the project, so
per-user, per-project customization is also possible.  So in addition
to activity- and organization-based configuration and customization,
we also have project-based C&C.  Note that to support e.g. system,
workgroup, and user-level project-specific customizations we would
have to add some means of referring to specific projects in our
generic config files.  For example,
```clj
:projprofile {:proj "leiningen" :version "2.3.3-SNAPSHOT"
	     {:resource-paths ...}}
```

or something like that.  That way an individual developer could modify
the project config without mucking about in project.clj.  But maybe
there's a way to do this already in Leiningen?)

(In other words, version control systems clash with standard C&C
practice.  Normally, a config file in the current directory overrides
user settings specified in ~/.foo.  But when the config file in the
current directory is checked out from a repository, we might want to
override some of its settings - e.g. to experiment - but we don't want
to edit it, to avoid inadvertently pushing transient changes.  So in
this case it would be useful to have some means of using
~/.lein/profile to override project.clj settings.)

Leiningen customization using profiles follows a standard
customization strategy: to support organization-based customizations,
define a set of configuration files and an override or combination
mechanism such that, when the files are read and interpreted in the
right order, the various levels of customization outlined above are
possible.  To support customization based on activity type, Leiningen
provides a few predefined activity-based profiles (:dev, :test,
:production) and supports definition of user-defined profiles that can
be declared at any level of the organizational hierarchy.

(In other words, activity-based customization depends both on the
ability to name sets of customizations and the ability to set the
corresponding named "mode" for the tool.  Having a set of
customizations called :foo is useless without a way of also telling
the tool to use that set - i.e. to "run in foo mode".  That is what
Leiningen's `with-profile` is for.  By contrast, organization-based
customization is supported by read-and-override involving a hierarchy
of configuration files regardless of mode.)

(TODO: instead of "activity", e.g. :test is for the test activity or
phase, express this in terms of per-task customizations.  An example
of a per-task customization is the :test profile, which "applies" to
the `test` task only.  (Alternatively call them activities instead of
tasks.)  Other per-task profiles are :repl and uberjar (any others?).
Note that per-task profiles can be specified at any organization
level.)

Leiningen actually confuses this a bit by naming organization-based
configuration sets (i.e. :system and :user).  The same thing can be
accomplished just by means of an override mechanism.  E.g. whatever is
in my personal ~/.lein/profiles.clj overrides the system and global
stuff.  I just use the activity-based tags to set e.g. test config.  I
don't need a :user profile to do this, since by definition anything in
my personal ~/.lein/profiles.clj counts as part of my user profile.

But that's just an implementation strategy.  The way Leiningen does it
also works, it's just a little more confusing.

*TO ADD* compare Leiningen's way of merging maps with "standard" tools 
e.g. https://github.com/typesafehub/config, libconfig, libcfg, etc.

NB: we can define e.g. a "foo" profile at any level, then override it
just like any other profile.  So an org might define profile :foo in
etc/leiningen, a user might customize in ~/.lein/profiles.clj and in
<projroot>/project.clj.  Is this then org-based, activity-based, or
project-based?  Is that even a useful question?  Presumably one would
define profile :foo for a reason having to do with how one runs
something, hence activity-based.  But maybe not; maybe it's intended
for all activities.  In that case, you would not define :foo, you
would customize the :system profile.

### Configuration files

 * etc/leiningen

 * ~/.lein/profiles.clj -- contains profile definition clauses like
   `{:user {:dependencies ...}}`

 * ~/.lein/profiles.d/foo.clj -- by definition, defines profile ":foo"

 * project.clj  (globally, in repo; locally, in project root local to user)


### Predefined Profiles

In addition to the Big Five:

 * repl

 * test

 * uberjar

### Modes (Activity Types?)

Default mode ...

The repl, test, and uberjar tasks check for profiles of the same name,
so essentially each can be viewed as an execution mode.

# Profiles

In Leiningen 2.x you can change the configuration of your project by
applying various profiles. For instance, you may want to have a few
extra test data directories on the classpath during development
without including them in the jar, or you may want to have Slamhound
available in every project you hack on without modifying every single
project.clj you use.

Each Leiningen task uses a particular combination of profiles.  You
can think of each task as executing in a particular "mode", which is
characterized by the project map Leiningen assembles before launching
the task.  Most standard tasks run in what we'll call default mode.
The *test* task, in contrast, runs in test mode.  (We use the term
"mode" here as a convenient way of referring to the fact that
different tasks may select different combinations of profiles for use
in constructing the project map.)

Because support for profiles is implemented using ordinary project
options, Leiningen configurations are completely customizable.

The `with-profile` subtask allows you to specify the profile to be
used for a particular task.  You can define your own profiles for use
with `with-profile`.

*CAVEAT* Leiningen defines a small set of profiles whose use is
 described below, but it also defines a set of configuration files
 that are scanned in a particular order for profile (and other)
 declarations whenever Leiningen runs.  This provides a means of both
 setting and overriding values on a per-system, per-user, and
 per-project basis.  The result is that the effective project map
 depends not only on which profiles are defined but where they are
 defined.  Details below.

## Default Profile

All tasks use the following five profiles, which together are called
the `:default` profile:

 *  :base, :system, and :user support per-system and per-user customizations
 *  :dev supports customizations for the default activity
 *  :provided supports customizations driven by use of Java frameworks

The :base profile is a global profile designed to provide the basic
functionalities needed by every use of Leiningen.  This includes, for
example, adding `org.clojure/tools.nrepl` to the project
:dependencies.  The default value for :base is hard-coded in the
Leiningen codebase, but since it is an ordinary option it can be
overridden.

The `:system` and `:user` profiles are intended to allow workgroups
and individuals respectively to customize their work.  The intended
use pattern is that `:system` should be declared in `/etc/leiningen`,
which, on a properly configured system, would be shared by a group of
users.  Similarly, :user is intended for use in ~/.lein/profiles.clj,
since declarations found there apply to every Leiningen task the user
runs.  For example, you can specify that the `lein-pprint` plugin
should always be enabled.

The :dev profile is designed to support, development-phase
customization; it should be declared in the project.clj file.  Note
that and explicit :project profile is not needed, since the top-level
project options in project.clj effectively implicitly define such a
profile.  So :dev is an activity-based profile.

The :provided profile is designed to support a standard usage pattern
associated with Frameworks.  Many Java frameworks expect deployment of
a jar file or derived archive sub-format containing a subset of the
application's necessary dependencies.  The framework expects to
provide the missing dependencies itself at run-time.  Dependencies
which are provided by a framework in this fashion may be specified in
the `:provided` profile.  Such dependencies will be available during
compilation, testing, etc., but won't be included by default by the
`uberjar` task or plugin tasks intended to produce stable deployment
artifacts.

By default the `:dev`, `:provided`, `:user`, `:system`, and `:base`
profiles are activated for each task, but their settings are not
propagated downstream to projects that depend upon yours. Each profile
is defined as a map which gets merged into your project map.

You can place any arbitrary `defproject` entries into a given profile
and they will be merged into the project map when that profile is
active.

The example below adds a "dummy-data" resources directory during
development and a dependency upon "midje" that's only used for tests.

```clj
(defproject myproject "0.5.0-SNAPSHOT"
  :description "A project for doing things."
  :dependencies [[org.clojure/clojure "1.4.0"]]
  :profiles {:dev {:resource-paths ["dummy-data"]
                   :dependencies [[expectations "1.4.41"]]}})
```


## Declaring Profiles

In addition to `project.clj`, profiles also can be specified in `profiles.clj`
within the project root. Profiles specified in `profiles.clj` will override
profiles in `project.clj`, so this can be used for project-specific overrides
that you don't want committed in version control.

User-wide profiles can also be specified in `~/.lein/profiles.clj`. These will be
available in all projects managed by Leiningen, though those profiles will be
overridden by profiles of the same name specified in the project.
System-wide profiles can be placed in `/etc/leiningen`. They are treated
the same as user profiles, but with lower precedence.

You can also define user-wide profiles within `clj`-files inside
`~/.lein/profiles.d`. The semantics within such files differ slightly
from other profile files: rather than a map of maps, the profile map
is the top-level within the file, and the name of the profile comes
from the file itself (without the `.clj` part). Defining the same
user-wide profile in both `~/.lein/profiles.clj` and in
`~/.lein/profiles.d` is considered an error.

The `:user` profile is separate from `:dev`; the latter is intended to
be specified in the project itself. In order to avoid collisions, the
project should never define a `:user` profile, nor should a user-wide
`:dev` profile be defined. Use the `show-profiles` task to list them.

If you want to access dependencies during development time for any
project place them in your `:user` profile. Your
`~/.lein/profiles.clj` file could look something like this:

```clj
{:user {:plugins [[lein-pprint "1.1.1"]]
        :dependencies [[slamhound "1.3.1"]]}}
```

## Merging

Profiles are merged by taking each key in the project map or profile
map, combining the value if it's a collection and replacing it if it's
not. Profiles specified later take precedence when replacing, just
like the `clojure.core/merge` function. The dev profile takes
precedence over user by default. Maps are merged recursively, sets are
combined with `clojure.set/union`, and lists/vectors are
concatenated. You can add hints via metadata that a given value should
take precedence (`:replace`) or defer to values from a different
profile (`:displace`) if you want to override this logic:

```clj
{:profiles {:dev {:prep-tasks ^:replace ["clean" "compile"]
                  :aliases ^:displace {"launch" "run"}}}}
```

The exception to this merge logic is that `:plugins` and `:dependencies`
have custom de-duplication logic since they must be specified as
vectors even though they behave like maps (because it only makes sense
to have a single version of a given dependency present at once). The
replace/displace metadata hints still apply though.

Another use of profiles is to test against various sets of dependencies:

```clj
(defproject swank-clojure "1.5.0-SNAPSHOT"
  :description "Swank server connecting Clojure to Emacs SLIME"
  :dependencies [[org.clojure/clojure "1.2.1"]
                 [clj-stacktrace "0.2.4"]
                 [cdt "1.2.6.2"]]
  :profiles {:1.3 {:dependencies [[org.clojure/clojure "1.3.0"]]}
             :1.4 {:dependencies [[org.clojure/clojure "1.4.0-beta1"]]}})
```

## Activating Profiles

To activate a different set of profiles for a given task, use the
`with-profile` higher-order task:

    $ lein with-profile 1.3 test :database

Multiple profiles may be combined with commas:

    $ lein with-profile qa,user test :database

Multiple profiles may be executed in series with colons:

    $ lein with-profile 1.3:1.4 test :database

The above invocations activate the given profiles in place of the
defaults. To activate a profile in addition to the defaults, prepend
it with a `+`:

    $ lein with-profile +server run

You can also use `-` to deactivate a profile.

Activating different profiles will change your `:target-path` so that
elements which come from profiles don't spill over and contaminate
other profiles.

## Composite Profiles

Sometimes it is useful to define a profile as a combination of other
profiles. To do this, just use a vector instead of a map as the profile value.
This can be used to avoid duplication:

```clj
{:shared {:port 9229, :protocol \"https\"}
 :qa [:shared {:servers [\"qa.mycorp.com\"]}]
 :stage [:shared {:servers [\"stage.mycorp.com\"]}]
 :production [:shared {:servers [\"prod1.mycorp.com\", \"prod1.mycorp.com\"]}]}
```

Composite profiles are used by Leiningen internally for the `:default`
profile, which is the profile used if you don't change it using
`with-profile`. The `:default` profile is defined to be a composite of
`[:base :system :user :provided :dev]`, but you can change this in your
`project.clj` just like any other profile.

## Debugging

To see how a given profile affects your project map, use the
[lein-pprint](https://github.com/technomancy/leiningen/tree/stable/lein-pprint)
plugin:

    $ lein with-profile 1.4 pprint
    {:compile-path "/home/phil/src/leiningen/lein-pprint/classes",
     :group "lein-pprint",
     :source-path ("/home/phil/src/leiningen/lein-pprint/src"),
     :dependencies
     ([org.clojure/tools.nrepl "0.0.5" :exclusions [org.clojure/clojure]]
      [clojure-complete "0.1.4" :exclusions [org.clojure/clojure]]
      [org.thnetos/cd-client "0.3.3" :exclusions [org.clojure/clojure]]),
     :target-path "/home/phil/src/leiningen/lein-pprint/target",
     :name "lein-pprint",
     [...]
     :description "Pretty-print a representation of the project map."}

In order to prevent profile settings from being propagated to other
projects that depend upon yours, the `:default` profiles are removed
from your project when generating the pom, jar, and uberjar, and an
`:uberjar` profile, if present, is included when creating
uberjars. (This can be useful if you want to specify a `:main`
namespace for uberjar use without triggering AOT during regular
development.) Profiles activated through an explicit `with-profile`
invocation will be preserved.
