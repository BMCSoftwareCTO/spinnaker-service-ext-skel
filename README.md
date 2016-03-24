# spinnaker-service-ext-skel

## Summary

This is skeleton of a Spinnaker service extension repository. This is used to create private changes to any Spinnaker service besides deck. The buildDeb Gradle task will create a Debian package that can be installed onto an instance where the target Spinnaker service is installed to augment it's JARs in the CLASSPATH and the yaml configuration.

The below steps will use clouddriver as an example.

## Creating a Spinnaker Service Extension Repo

1. Create Spinnaker service extension repo called clouddriver-bmc
2. git clone the skeleton repo (a.k.a spinnaker-service-ext-skel)
3. `rm -fr .git`
4. `git init`
3. Change the remote of the clone to point to the created Spinnaker service extension repo
```
   git remote add origin git@github.com:BMCSoftwareCTO/clouddriver-bmc.git
```
4. `rm -fr README.md` or update it to reflect the service extension details

## Dependencies
For each of the subprojects like clouddriver-web, etc, the Gradle build script applies a plugin called nebula.provided-base:
```
plugins {
  id 'nebula.provided-base' version '3.0.3' // if you want provided-base
}
```

This plugin allows a new dependency qualifier called `provided`
Example:

```
dependencies {
   provided 'com.netflix.spinnaker.clouddriver:clouddriver-core:1.108.0'
}
```

What this specifies is that the dependency will be compiled against for this Spinnaker service project extension build, but the JARs will not be included in the Debian package. This is because these `provided` dependencies are already provided by the Spinnaker service main installation and hence they do not need to be packaged and installed by this extension.

## Edit build.gradle

1. Change ```group = "com.bmc.spinnaker.spinnakersvc"``` to ```group = "com.bmc.spinnaker.clouddriver"```

## Edit settings.gradle

1. Change ```rootProject.name="spinnakersvc"``` to ```rootProject.name="clouddriver"```
2. For ```include 'spinnakersvc-web', 'spinnakersvc-security'```, change ```spinnakersvc``` to ```clouddriver``` and add/remove to this list to match the number of subprojects that will be in this extension.

## spinnakersvc-web

1. Rename the diretory to clouddriver-web
2. Rename config/spinnakersvc.yml to config/clouddriver.yml
3. Rename spinnakersvc-web.gradle to clouddriver-web.gradle
4. Edit clouddriver-web.gradle
  1. In the dependencies block, add/remove lines like ```compile project(':clouddriver-security')``` to match the subprojects in this extension
  2. In the dependencies block, add/remove lines like ```provided 'com.netflix.spinnaker.clouddriver:clouddriver-core:1.108.0'``` to match the core dependencies that this extension has on the service that it is augmenting.
5. Rename ```src/main/groovy/com/netflix/spinnaker/spinnakersvc``` to ```src/main/groovy/com/netflix/spinnaker/clouddriver``` or add/remove source directories like this as necessary.

## spinnakersvc-security

This represents each additional subproject that is being augmented.

1. Add/remove directories for each subproject like ```clouddriver-security```
2. Create a named Gradle file like ```clouddriver-security.gradle```
  1. Put this block at the top
  ```
  plugins {
    id 'nebula.provided-base' version '3.0.3' // if you want provided-base
  }
  ```
3. Add dependencies to the corresponding Spinnaker service subproject
```
  dependencies {
   provided 'com.netflix.spinnaker.clouddriver:clouddriver-security:1.108.0'
  }
```

## Build

You will need to do an initial commit locally: `git commit -m "Initial commit"`. Later you can add to this local commit before pushing with the command: `git commit --amend`

To create the Debian package run from the repo root path:
`./gradlew buildDeb`

The Debian package is located in: `clouddriver-web/build/distributions` and named something like `spinnaker-clouddriver-bmc_0.1.0-4_all.deb`
