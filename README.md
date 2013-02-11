Jenkins update center generator
===============================

This project is primarily used to generate the jenkins-ci.org update center laying out the files to and generating
summary files about the core [aka the wars] and the plugins.

With a few modifications it could easily be used to generate your corporate update center as well.

The generator
-------------

The generator can:

* generate a static navigation site for the update center site including
    * index.html files for the core and the plugins. Each index.html points to the latest release and all known versions identified by their number
    * .htaccess files containing 302 redirects for managing latest versions of the core and the plugins
    * a txt file containing the latest version of the core
    * json files:
        * the core release history
        * the plugins release history
    * symlinks
* download the hpis and wars
* sign the json files

Extra features:

* allow to blacklist plugins to ignore. See src/main/resources/artifact-ignores.properties
* temporary override of the plugins wiki page. src/main/resources/wiki-overrides.properties

The generator pulls information from:

* a nexus index site
* remote and local maven repositories
* confluence

The generator doesn't have a full usage page yet. Meanwhile you can read the code
of [the arg4js annotated Main class](backend-update-center2/blob/master/src/main/java/org/jvnet/hudson/update_center/Main.java "the Main class")

Standard www layout (without hpis and wars)
-------------------------------------------

    www/latestCore.txt
    www/release-history.json
    www/update-center.json
    www/latest/
    www/latest/.htaccess
    www/download
    www/download/plugins/
    www/download/plugins/checkstyle/
    www/download/plugins/checkstyle/index.html
    www/download/plugins/cccc/
    www/download/plugins/cccc/index.html
    [...]
    www/download/war/
    www/download/war/index.html

Running the generator
---------------------

The project various artifacts to be used on a site hosting a jenkins update center
The project produces a jar and a zip file containing all the required dependencies to run the generator.

If you want to run the generator from within your development environment,
you can try to use the appassembler plugin as described below. The exec:java plugin won't work.

    # to generate the files in a standard layout
    # warning this may take quite a bit of time, so you might want to add the -maxPlugins 1 option
    mvn package appassembler:assemble
    sh target/appassembler/bin/app -id com.example.jenkins -www www

Arguments
---------
* -id
	* Required
	* Uniquely identifies this update center.
	* We recommend you use a dot-separated name like "com.sun.wts.jenkins".
	* This value is not exposed to users, but instead internally used by Jenkins.
	* Used as "id" field in update-center.json
* -repository
	* Alternate repository for plugins.
	* If not specified, http://repo.jenkins-ci.org/public/ is used.
	* Default: null
* -repositoryName
	* Name of repository. This is a value for n opition of nexus-indexer-cli.
	* If not specified, "public" is used.
	* Default: null
* -remoteIndex
	* Nexus index file in repository.
	* If not specified, .index/nexus-maven-repository-index.gz is used.
	* Default: null
* -download
	* Build download server layout
	* Default: null
* -www
	* Built jenkins-ci.org layout
	* Default: null
* -r
	* release history JSON file
	* When -www is specified, $(www)/release-history.json is used.
	* Default: release-history.json
* -h
	* htaccess file
	* When -www is specified, $(www)/latest/.htaccess is used.
	* Default: .htaccess
* -o
	* json file(update-center.json)
	* When -www is specified, $(www)/update-center.json is used.
	* Default: output.json
* -index.html
	* Update the version number of the latest jenkins.war in jenkins-ci.org/index.html
	* When -www is specified, $(www)/index.html is used.
	* Default: null
* -latestCore.txt
	* Update the version number of the latest jenkins.war in latestCore.txt
	* When -www is specified, $(www)/latestCore.txt is used. 
	* Default: null
* -maxPlugins
	* For testing purposes.
	* Limit the number of plugins managed to the specified number.
	* Default: null
* -connectionCheckUrl
	* Specify an URL of the 'always up' server for performing connection check.
	* Used as "connectionCheckUrl" field in update-center.json
	* Default: null
* -pretty
	* Pretty-print the result
	* Default: false
* -cap
	* Cap the version number and only report data that's compatible with

How to work with windows
------------------------
In windows, app.bat results in following fucking output:
	input line is too long

So, run in following steps...

1. Flatten all jar files to a directory.
	mkdir target\appassembler\repo.flat
	for /f %A in ('dir /S /b target\appassembler\repo\*.jar') do copy /Y "%A" target\appassembler\repo.flat
2. Modify app.bat as following:
	1. Modify "set CLASSPATH=..." to set empty.
		Don't comment out, for even it results in "input line is too long"...REMOVE IT.
	2. set EXTRA_JVM_ARGUMENTS as following
		set JRE_HOME=%JAVA_HOME%
		if exist "%JAVA_HOME%\jre" set JRE_HOME=%JAVA_HOME%\jre
		set EXTRA_JVM_ARGUMENTS="-Djava.ext.dirs=%REPO%.flat;%JRE_HOME%\lib\ext"
