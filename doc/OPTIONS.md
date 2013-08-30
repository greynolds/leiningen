# Options

## Prelims

Leiningen uses some System properties and som env vars:

### System props

 * user.name
 * java.version

### Environment vars

## Options


### :profiles


From sample.project.clj:

  ;;; # Profiles
  ;; Each active profile gets merged into the project map. The :dev
  ;; and :user profiles are active by default, but the latter should be
  ;; looked up in ~/.lein/profiles.clj rather than set in project.clj.
  ;; Use the with-profiles higher-order task to run a task with a
  ;; different set of active profiles.
  ;; See `lein help profiles` for a detailed explanation.
  :profiles {:dev {:resource-paths ["dummy-data"]
                   :dependencies [[clj-stacktrace "0.2.4"]]}
             :debug {:debug true
                     :injections [(prn (into {} (System/getProperties)))]}
             :1.4 {:dependencies [[org.clojure/clojure "1.4.0"]]}
             :1.5 {:dependencies [[org.clojure/clojure "1.5.0"]]}}

leiningen/core/project.clj:

(when-not (or result (#{:provided :dev :user :test :base :default
	      	     		  :production :system :repl}
			profile))
	(println "Warning: profile" profile "not found."))


#### Predefined profiles

Note that "predefined" profile means it has some hard-coded options.

 * :default [:base :system :user :provided :dev]

default-profiles map keys:

 * :base {:resource-paths ["dev-resources"]
   	 :jvm-opts (with-meta tiered-jvm-opts
	 	   {:displace true})
	:test-selectors {:default (with-meta '(constantly true)
			{:displace true})}
	:dependencies '[[org.clojure/tools.nrepl "0.2.3"
		         :exclusions [org.clojure/clojure]]
			[clojure-complete "0.2.3"
		    	  :exclusions [org.clojure/clojure]]]
	:checkout-deps-shares [:source-paths
			      :test-paths
			      :resource-paths
			      :compile-path
			      #'classpath/checkout-deps-paths]}
 * :leiningen/test {:injections [hooke-injection]
   		   :test-selectors {:default (with-meta
		   		   '(constantly true)
				   {:displace true})}}
 * :uberjar {} ; TODO: use :aot :all here in 3.0
 * :update {:update :always}
 * :offline {:offline? true}
 * :debug {:debug true}}))

project.clj/read-profiles [project] merges default-profiles map with
system-profiles, user/profiles, :profiles passed in the project arg,
and proj:root:project.clj.  Semantics of `merge` says later keys
override earlier keys.  So the priority order is:

 project root/project.clj
   project:profiles  (where project is arg to read-profiles
     user/profiles  (~/.lein/profiles.clj, profiles.d)
       system-profiles  (etc/leiningen
         default-profiles (map of :default, :base, etc.)

(Here, upper levels override lower.  Each component is a map with
profile keys.  Except for project-root/project.clj?)

Other predefs:

 * :test
 * :production
 * :repl
 * :user
 * :system

:without-profiles?