# Baseline Java code quality plugins

[![CircleCI Build Status](https://circleci.com/gh/palantir/gradle-baseline/tree/develop.svg?style=shield)](https://circleci.com/gh/palantir/gradle-baseline)
[![Bintray Release](https://api.bintray.com/packages/palantir/releases/gradle-baseline/images/download.svg) ](https://bintray.com/palantir/releases/gradle-baseline/_latestVersion)

Baseline Java is a collection of Gradle plugins for configuring code quality tools in builds and generated
Eclipse/IntelliJ projects. It configures [Checkstyle](http://checkstyle.sourceforge.net) and
[error-prone](https://errorprone.info) for style and formatting checks, and Eclipse/IntelliJ code style and formatting
configurations consistent with the
[Baseline Java Style Guide and Best Practices](https://github.com/palantir/gradle-baseline/tree/develop/docs)

The Baseline plugins are compatible with Gradle 4.0.0 and above.






## Quick start
- Add the Baseline plugins to the `build.gradle` configuration of the Gradle project:

```Gradle
buildscript {
    repositories {
        maven { url  "http://palantir.bintray.com/releases" }
    }

    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

repositories {
    maven { url  "http://palantir.bintray.com/releases" }
}

dependencies {
    // Adds a dependency on the Baseline configuration files. Typically use
    // the same version as the plugin itself.
    baseline "com.palantir.baseline:gradle-baseline-java-config:<version>@zip"
}

apply plugin: 'java'
apply plugin: 'org.inferred.processors'  // installs the "processor" configuration needed for baseline-error-prone
apply plugin: 'com.palantir.baseline'

// All plugins apart from class-uniqueness are applied by default by baseline plugin.
apply plugin: 'com.palantir.baseline-class-uniqueness'
```

- Run ``./gradlew baselineUpdateConfig`` to download the config files
referenced in the `dependencies.baseline` configuration and extract them to .baseline/
- Any subsequent ``./gradlew build`` invokes Checkstyle as part of the build and test tasks (if the
respective baseline-xyz plugins are applied).
- The ``eclipse`` and ``idea`` Gradle tasks generate projects pre-configured with Baseline settings:

   - Code style and code formatting rules conforming with Baseline style
   - Checkstyle configuration

  Note that the Checkstyle-IDEA plugin is required to run the Baseline Checkstyle within IntelliJ.




## Development environment
Tests are run with `./gradlew publishToMavenLocal build`, the publishing step is required in order to make Baseline
artifacts available to the tests. Note that some of the tests only work when run for the first time since they assume
particular directory structures that are unavailable when re-running tests.

IDE configurations can be generated with `./gradlew idea eclipse`.

Generally, you should check generated `.baseline` folder into version control. Though it is not compulsory, since it can be recreated using the `baselineUpdateConfig` task, checking it into git enables your project to customize its rules and share them across developers.



## Plugin Architecture Overview

The Baseline plugins `com.palantir.baseline-checkstyle`, `com.palantir.baseline-eclipse`,
`com.palantir.baseline-idea`, `com.palantir.baseline-error-prone` apply the configuration present in `.baseline` to the
respective Gradle tasks. For example, any Gradle Checkstyle tasks uses the Checkstyle configuration in
`.baseline/checkstyle/checkstyle.xml`, and any IntelliJ/Eclipse project generated by `./gradlew eclipse idea` is
configured with Baseline code formatting and Checkstyle rules. Note that each of these plugins automatically applies the
underlying Gradle plugin: `com.palantir.baseline-checkstyle` applies `checkstyle`, `com.palantir.baseline-eclipse`
applies `eclipse`, `com.palantir.baseline-error-prone` applies `net.ltgt.errorprone`, etc.





## Configuration

The standard Gradle configuration options for the underlying plugins (Eclipse, IntelliJ, Checkstyle) can be
used, with the following exception:

- `checkstyle.configFile` - not compatible with Baseline since the file location is hard-coded to
`.baseline/checkstyle/checkstyle.xml`






## Advanced usage

### Multiple-project builds

All `com.palantir.baseline-xyz` plugins can be applied selectively to subprojects. For example:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

apply plugin: 'com.palantir.baseline-idea'

subprojects {
    apply plugin: 'java'
    apply plugin: 'com.palantir.baseline-checkstyle'
    apply plugin: 'com.palantir.baseline-idea'
}
```

Depending on the Gradle setup, you may need to edit `gradle/shared.gradle` (or similar) instead. Feel free to contact
the Baseline mailing list for troubleshooting.


### Applying Baseline plugins selectively or all at once

The `com.palantir.baseline` plugin applies all `com.palantir.baseline-xyz` plugins to all projects. In order to
use only Checkstyle and IntelliJ support from Baseline, apply the required plugins selectively, e.g.:

```Gradle
buildscript {
    dependencies {
        classpath 'com.palantir.baseline:gradle-baseline-java:<version>'
    }
}

apply plugin: 'com.palantir.baseline-idea'
subprojects {
    apply plugin: 'com.palantir.baseline-idea' // Applies all com.palantir.baseline-xyz plugins
}
```





### Checkstyle Plugin (com.palantir.baseline-checkstyle)

Checkstyle rules can be suppressed on a per-line or per-block basis. (It is good practice to first consider formatting
the code block in question according to the project's style guidelines before adding suppression statements.) To
suppress a particular check, say `MagicNumberCheck`, from an entire class or method, annotate the class or method with
the lowercase check name without the "Check" suffix:

```Java
@SuppressWarnings("checkstyle:magicnumber")
```

Checkstyle rules can also be suppressed using comments, which is useful for checks such as `IllegalImport` where
annotations cannot be used to suppress the violation. To suppress checks for particular lines, add the comment
`// CHECKSTYLE:OFF` before the first line to suppress and add the comment `// CHECKSTYLE:ON` after the last line.

To disable certain checks for an entire file, apply [custom suppressions](http://checkstyle.sourceforge.net/config.html)
in `.baseline/checkstyle/checkstyle-suppressions`.


### Eclipse Plugin (com.palantir.baseline-eclipse)

Run `./gradlew eclipse` to repopulate projects from the templates in `.baseline`.

The `com.palantir.baseline-eclipse` plugin automatically applies the `eclipse` plugin, but not the `java` plugin. The
`com.palantir.baseline-eclipse` plugin has no effects if the `java` plugin is not applied.

If set, `sourceCompatibility` is used to configure the Eclipse project settings and the Eclipse JDK version. Note
that `targetCompatibility` is also honored and defaults to `sourceCompatibility`.

Generated Eclipse projects have default per-project code formatting rules as well as Checkstyle configuration.

The Eclipse plugin is compatible with the following versions: Checkstyle 7.5+, JDK 1.7, 1.8


### IntelliJ Plugin (com.palantir.baseline-idea)

Run `./gradlew idea` to (re-) generate IntelliJ project and module files from the templates in `.baseline`. The
generated project is pre-configured with Baseline code style settings and support for the Checkstyle-IDEA plugin.

The `com.palantir.baseline-idea` plugin automatically applies the `idea` plugin.

Generated IntelliJ projects have default per-project code formatting rules as well as Checkstyle configuration. The JDK
and Java language level settings are picked up from the Gradle `sourceCompatibility` property on a per-module basis.

### Error-prone

The `com.palantir.baseline-error-prone` plugin brings in the `net.ltgt.errorprone` plugin and adds an annotation-processor dependency on Baseline-specific error-checks (see below). We recommend applying the `org.inferred.processors` plugin in order to configure an appropriate `processor` configuration. The minimal setup is as follows:


```groovy
buildscript {
    dependencies {
        classpath 'gradle.plugin.org.inferred:gradle-processors:1.2.3'
    }
}

apply plugin: 'org.inferred.processors'
apply plugin: 'com.palantir.baseline-error-prone'
```

The version of the error-prone library defaults to "latest" and can be adjusted via the `errorprone` configuration, see
[gradle-errorprone-plugin](https://github.com/tbroyer/gradle-errorprone-plugin) for details.

```groovy
dependencies {
    errorprone 'com.google.errorprone:error_prone_core:2.0.19'  // update version as desired
}
```

#### Error-prone suppressions

Error-prone rules can be suppressed on a per-line or per-block basis just like Checkstyle rules:

```Java
@SuppressWarnings("Slf4jConstantLogMessage")
```

Rules can be suppressed at the project level, or have their severity modified, by adding the following to the project's `build.gradle`:

```groovy
tasks.withType(JavaCompile) {
    options.compilerArgs += ['-Xep:Slf4jLogsafeArgs:OFF']
}
```
Warnings on generated code can be suppressed as follows:

```groovy
tasks.withType(JavaCompile) {
    options.compilerArgs += ['-XepDisableWarningsInGeneratedCode']
}
```

More information on error-prone severity handling can be found at [errorprone.info/docs/flags](http://errorprone.info/docs/flags).

#### Baseline error-prone checks
Baseline configures the following checks in addition to the [error-prone's out-of-the-box
checks](https://errorprone.info):

- Slf4jConstantLogMessage: Allow only compile-time constant slf4j log message strings.
- Slf4jLogsafeArgs: Allow only com.palantir.logsafe.Arg types as parameter inputs to slf4j log messages. More information on
Safe Logging can be found at [github.com/palantir/safe-logging](https://github.com/palantir/safe-logging).


### Class Uniqueness Plugin (com.palantir.baseline-class-uniqueness)

Run `./gradlew checkClassUniqueness` to scan all jars on the `runtime` classpath for identically named classes.
This task will run automatically as part of `./gradlew build`.

If you discover multiple jars on your classpath contain clashing classes, you should ideally try to fix them upstream and then depend on the fixed version.  If this is not feasible, you may be able to tell Gradle to [use a substituted dependency instead](https://docs.gradle.org/current/userguide/customizing_dependency_resolution_behavior.html#sec:module_substitution):

```gradle
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.name == 'log4j') {
            details.useTarget group: 'org.slf4j', name: 'log4j-over-slf4j', version: '1.7.10'
            details.because "prefer 'log4j-over-slf4j' over any version of 'log4j'"
        }
    }
}
```

### CircleCi Plugin (com.palantir.baseline-circleci)

Automatically applies the following plugins:

- [`com.palantir.circle.style`](https://github.com/palantir/gradle-circle-style) - this configures checkstyle xml output to be written to the `$CIRCLE_TEST_REPORTS` directory.
- [`com.palantir.configuration-resolver`](https://github.com/palantir/gradle-configuration-resolver-plugin) - this adds a `./gradlew resolveConfigurations` task which is useful for caching on CI.

Also, the plugin:

1. stores the HTML output of tests in `$CIRCLE_ARTIFACTS/junit`
1. stores the HTML reports from `--profile` into `$CIRCLE_ARTIFACTS/reports`


## com.palantir.baseline-versions

Sources version numbers from a root level `versions.props` file.  This plugin should be applied in an `allprojects` block. It is effectively a shorthand for the following:

```gradle
buildscript {
    dependencies {
        classpath 'com.netflix.nebula:nebula-dependency-recommender:x.y.z'
    }
}

allprojects {
    apply plugin: 'nebula.dependency-recommender'
    dependencyRecommendations {
        strategy OverrideTransitives // use ConflictResolved to undo this
        propertiesFile file: project.rootProject.file('versions.props')
        if (file('versions.props').exists()) {
            propertiesFile file: project.file('versions.props')
        }
    }
}
```

Features from [nebula.dependency-recommender](https://github.com/nebula-plugins/nebula-dependency-recommender-plugin) are still available (for now), so you can configure BOMs:

```gradle
dependencyRecommendations {
    mavenBom module: 'com.palantir.product:your-bom'
}
```

### Copyright Checks

By default Baseline enforces Palantir copyright at the beginning of files. To change this, edit the template copyright
in `.baseline/copyright/*.txt` and the RegexpHeader checkstyle configuration in `.baseline/checkstyle/checkstyle.xml`
Gradle Circle Style
===================

A plugin for Gradle that summarizes failed Gradle builds in [CircleCI], with special handling for [Checkstyle] and [FindBugs] failures.

[Checkstyle]: https://docs.gradle.org/current/userguide/checkstyle_plugin.html
[CircleCI]: https://circleci.com/
[FindBugs]: https://docs.gradle.org/current/userguide/findbugs_plugin.html

[![Gradle plugins page](https://img.shields.io/github/release/palantir/gradle-circle-style.svg?maxAge=60)](https://plugins.gradle.org/plugin/com.palantir.circle.style)
[![CircleCI](https://img.shields.io/circleci/project/github/palantir/gradle-circle-style/master.svg?maxAge=60)](https://circleci.com/gh/palantir/gradle-circle-style/tree/master)
[![Apache 2.0 License](https://img.shields.io/github/license/palantir/gradle-circle-style.svg?maxAge=2592000)](http://www.apache.org/licenses/LICENSE-2.0)

Quickstart
----------

Add the following to your project's top-level build.gradle file:

```gradle

plugins {
  id 'com.palantir.circle.style' version '1.1.2'
}
```

And now your CircleCI builds will fail with nice summaries:

![CHECKSTYLE — 1 FAILURE](images/checkstyle-circle-failure.png?raw=true "CircleCI failure image")

Details
-------

This plugin is enabled by the `CIRCLE_TEST_REPORTS` environment variable, set automatically on CircleCI builds. It then automatically enables XML output for Checkstyle and FindBugs plugins, and adds a finalizer task that collates their results (and any other Gradle build step failures) using the JUnit XML output that CircleCI expects.

Note that FindBugs does not support generating both HTML and XML output, so HTML output will be disabled on CircleCI builds. (Checkstyle does not have this limitation.)
