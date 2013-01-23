## Binks
Allows jar and war files that contain nested jar dependencies to be
run using `java -jar archive.jar`.

## Design Goals
* Dependencies do not need to be unpacked or modified in order to be used
* Temporary files are not be written in order to load nested jars
* Minimal memory consumption when working with packaged files
* Support the generation of executable war files (similar to Jenkins)
* Small codebase with no external dependencies
* Programmatic discovery of dependencies (i.e. no huge manifest)
* Works with Java 1.6 or above
* Apache 2.0 licensed code with no requirements to distribute copyright notices

## Structuring Files

### JAR Files
Jar files should be structured in the following way:

	example.jar
	 |
	 +-META-INF
	 |  +-MANIFEST.MF
	 +-org
	 |  +-springframework
	 |  |  +-<binks classes>
	 |  +-yourcorp
	 |     +-YouClasses.class
	 +-lib
	    +-dependency1.jar
	    +-dependency1.jar

Dependencies should be placed in a nested `lib` directory. Binks classes should be
unpacked in the root of the jar. The `MANIFEST.MF` file should contain the following:

	Start-Class: <your main class>
	Main-Class: org.springframework.binks.JarLauncher

See [binks-executable-jar sample](https://github.com/philwebb/binks/tree/master/binks-samples/binks-executable-jar).

### WAR Files
Jar files should be structured in the following way:

	example.jar
	 |
	 +-META-INF
	 |  +-MANIFEST.MF
	 +-org
	 |  +-springframework
	 |     +-<binks classes>
	 +-WEB-INF
	    +-classes
	    |  +-org
	    |     +-yourcorp
	    |        +-YouClasses.class
	    +-lib
	    |  +-dependency1.jar
	    |  +-dependency1.jar
	    +-lib-provided
	       +-servlet-api.jar
	       +-dependency3.jar

Dependencies should be placed in a nested `WEB-INF/lib` directory. Any dependencies
that are required when running embedded but are not required when deploying to
a traditional web container should be placed in `WEB-INF/lib-provided`. Binks
classes should be unpacked in the root of the jar. The 'MANIFEST.MF' file should contain
the following:

	Start-Class: <your main class>
	Main-Class: org.springframework.binks.WarLauncher

See [binks-executable-war sample](https://github.com/philwebb/binks/tree/master/binks-samples/binks-executable-war).

## Constraints

### Double compression
Zip entries that refer to nested jar files must not be compressed.  This is required so
that we can seek directly to individual content within the nested jar.  The content of
the nested jar file itself can still be compressed. Archives generated using Apache
Maven are compatible.

### Jetty Annotation Scanning
The annotation scanning method used by Jetty 8 is not compatible with nested jars.

## Alternatives
* [Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)
* [JarClassLoader](http://www.jdotsoft.com/JarClassLoader.php)
* [OneJar](http://one-jar.sourceforge.net)
