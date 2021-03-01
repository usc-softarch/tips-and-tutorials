# Maven

The Maven documentation, as well as its community, drive me nuts. Either it's over-specified or under-specified, and half the time I get holier-than-thou responses on why I shouldn't do what I want to do in the first place, rather than helping me solve the damn problem. So, here is a very brief explanation of the basic Maven information that I consider necessary to get someone started on it, followed by some snippet solutions that'll do what I want without wasting my time.

## Basics

Maven is essentially an extensible build system with a dependency solver. It is based on a lifecycle, a sequence of phases that are each made up of smaller tasks (https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html). The ones that matter for ARCADE are `clean`, `build`, `test` and `package`.

### mvn clean

The `mvn clean` command essentially resets the system to its initial state, deleting all artefacts produced by Maven until this point. Effectivelly, what this means is it recursively deletes the `target` directory. In addition to this, all systems I (Marcelo) manage bind a task to the `clean` phase which installs any "local" requirements to the machine, so that they may be accessible by later phases. For example, if a project is using a local library located at `libs/local_lib.jar`, then `clean` will install that library to the machine's `.m2` repository (`.m2` is Maven's local memory). What this means is you should __always__ run `mvn clean` the first time you clone a repository that I manage, or else other Maven phases may not work.

For more on how this works, [look here](#using-local-libraries).

### mvn build

`mvn build` will compile the source code into binary files. It will _not_ compile test sources, and will _not_ generate any sort of packages like `.jar` files. For the most part, I do not use `build` and so have not bound anything to it. However, it can be useful to run a quick succession of `mvn clean` and `mvn build` if your IDE's syntax highlighting or linter are acting strangely: for most IDEs, these functions work off of the compiler's output, so recompiling everything can fix erratic behavior. If it doesn't, it probably means your code is, in fact, wrong.

Also worth noting is that `mvn build`, unless explicitly told otherwise in the `pom.xml`, works off of the default source directory: that is, it will compile using `src/main/java` as its root.

### mvn test

`mvn test` serves to both compile and execute test code. When run without any parameters, it will compile everything in the default test directory (`src/test/java`) and execute all `@Test`s defined inside of `*Test.java` files. For more information on how this works, see http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html, specifically `<testSourceDirectory>` and `<includes>`. Remember that for a test class to be identified as such by Maven, it MUST follow the regular expression `*Test.java` or one of the other base regexps, that is, the class name must end in `Test`.

Two options of `mvn test` are important: `-Dtest` and `Dmaven.test.skip`.

`-Dtest` can be used to tell Maven exactly which tests to execute: for example, `mvn -Dtest=ClassTest test` will run all `@Test`s inside of `ClassTest`, and `mvn -Dtest=ClassTest#mytest test` will run the `mytest` `@Test` inside of `ClassTest`. For more information, see https://maven.apache.org/surefire/maven-surefire-plugin/examples/single-test.html.

`-Dmaven.test.skip` is the option that determines whether or not to execute this phase. This option is important because `test` is actually bound to later phases, in particular to `package`. By specifying `-Dmaven.test.skip=true`, you can tell Maven to skip this phase entirely. Remember that this means the tests will neither be executed nor even compiled. This option is useful when you are making small changes in quick succession and do not want to run a full battery of tests every time.

### mvn package

Finally, `mvn package` will execute both `build` and `test`, as well as further tasks related to packaging the binaries into a distributable file, such as a `.jar`. In ARCADE, two particular tasks are bound to `package`, one to generate a regular `.jar` and one to generate a `fat.jar`. The former contains only ARCADE's binaries, whereas the latter also contains the binaries from all of ARCADE's dependencies. For more information, look [here](#jar-with-dependencies), [here](#packaging-the-jar) and [here](#building-multiple-jars).

### Extra information

Maven does not work properly from inside VS Code's terminal, and likely has similar issues in other IDE terminals. IDE terminals, generally speaking, are a wrapper over an existing OS terminal, and sometimes that wrapper parses commands incorrectly. In Maven's case with VS Code, options that include a `.` will just fail. In practice, this means you should always use maven commands from a native terminal.

## Snippets

These are snippets to solve specific problems that I've encountered in the past. Many of these have examples inside of ARCADE's `pom.xml`, although some are not used in ARCADE.

### Template

Sometimes I forget the structure of the XML. This is it.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion></modelVersion>
  <groupId></groupId>
  <artifactId></artifactId>
  <packaging></packaging>
  <version></version>
  <name></name>
  <url></url>
  <dependencies>
    <dependency>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
      </plugin>
    </plugins>
  </build>
</project>
```

### Compiler Version

If the Java compiler acts up, this seems to fix it. Sometimes it's not necessary. I don't actually know why. Theoretically, the default Maven Java compiler uses version 1.5 for some ungodly reason, or at least that's what I found online. Either way, if javac whines, add this plugin. Obviously, replace the version for whatever you're using.

```
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

Another option is to add the following before your dependencies:

```
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

You should add this anyway so maven stops giving irrelevant warnings. I don't know why the archetype generator doesn't just add it automatically, and honestly I'm afraid to even ask.

### Linking plugins to phases

To run a plugin automatically during a particular phase, add this to it at the end. GOAL depends on the plugin, each plugin has a number of goals.

```
<executions>
  <execution>
    <id>make-assembly</id>
    <phase>package</phase>
    <goals>
      <goal>GOAL</goal>
    </goals>
  </execution>
</executions>
```

### Jar with dependencies

Because there are 317 different assembly plugins, if you search how to make a jar with dependencies you'll get some 318 different answers. This is how I've been doing it, and it works okay. This will make sure that the `.jar` output will contain all of the necessary dependencies (will be a "fat jar").

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>3.1.1</version>
  <executions>
    <execution>
      <id>copy-dependencies</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>copy-dependencies</goal>
      </goals>
      <configuration>
        <outputDirectory>${project.build.directory}/lib</outputDirectory>
        <overWriteReleases>false</overWriteReleases>
        <overWriteSnapshots>false</overWriteSnapshots>
        <overWriteIfNewer>true</overWriteIfNewer>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### Using local libraries

Google how to use a local library and at least 60% of the answers will be mobs with pitchforks and torches trying to burn you at the stake. Just add this plugin. You're an adult, you know what you're doing.

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-install-plugin</artifactId>
  <version>3.0.0-M1</version>
  <executions>
    <execution>
      <id>hack-binary</id>
      <phase>clean</phase>
      <configuration>
        <file>${basedir}/lib/[[[YOUR_LIB]]].jar</file>
        <repositoryLayout>default</repositoryLayout>
        <groupId>[[[LIB_GROUP_ID]]]</groupId>
        <artifactId>[[[LIB_ARTIFACT_ID]]]</artifactId>
        <version>[[[LIB_VERSION]]]</version>
        <packaging>jar</packaging>
        <generatePom>true</generatePom>
      </configuration>
      <goals>
        <goal>install-file</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

To install multiple ones, just add extra executions. To use them, add them as a dependency, same as any other.

```
<dependency>
  <groupId>[[[LIB_GROUP_ID]]]</groupId>
  <artifactId>[[[LIB_ARTIFACT_ID]]]</artifactId>
  <version>[[[LIB_VERSION]]]</version>
</dependency>
```

### Packaging the jar

This just builds an executable `.jar`. The classpath prefix makes it access your local libs (that were packaged into the jar with maven-dependency-plugin). For more information, see http://maven.apache.org/shared/maven-archiver/examples/classpath.html.

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.0.2</version>
  <configuration>
    <archive>
    <manifest>
    <addClasspath>true</addClasspath>
    <classpathPrefix>lib/</classpathPrefix>
    <mainClass>[[[FULLY_QUALIFIED_CLASSNAME]]]</mainClass>
    </manifest>
    </archive>
  </configuration>
</plugin>
```

### Building multiple jars

Sometimes a project can have several different packagings. You can specify them with this.

```
<plugin>
  <artifactId>maven-assembly-plugin</artifactId>
  <version>2.5.5</version>
  <executions>
    <execution>
      <id>[[[EXECUTION_ID]]]</id>
      <configuration>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <finalName>[[[OUTPUT_JAR_NAME]]]</finalName>
        <archive>
          <manifest>
            <mainClass>[[[FULLY_QUALIFIED_CLASSNAME]]]</mainClass>
          </manifest>
        </archive>
      </configuration>
      <phase>package</phase>
      <goals>
        <goal>single</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Again, to add more packagings, add more executions. All packages will contain everything, the only difference is the main class. There is a way to specify exactly what you want in the package, but I don't remember the details on how to do that.