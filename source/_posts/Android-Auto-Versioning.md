title: Android Auto-Versioning
date: 2015-11-09 13:51:16
categories: Android
tags:
  - Android
  - Gradle
comments: true
---
This post is a continuation of my [previous post](/2015/07/16/Steamlining-releases) about using Gradle to streamline versioning into the build process so you can spend more time on what's important: **building your product**. The basic idea is to automate a versioning process that follows [semantic versioning](http://semver.org/).

After some time of using my home grown solution for auto-versioning, I've decided to make a plugin out of it so others can easily implement this into their Android projects. My main goal for the plugin was to make it simple and let the developer be in control.

The intended way for using this plugin is to not use the tasks it generates directly. Instead, you write your own tasks that depend on the versioning task. That way you can run other things before/after the versioning task executes. For example, the four tasks that you get out of the box are:

 - prepareBuildRelease - Increments the build number
 - preparePatchRelease - Increments the patch number
 - prepareMinorRelease - Increments the minor number and resets the patch number
 - prepareMajorRelease - Increments the major number and resets the patch and minor number

And here is a sample task for creating a major release:

```Groovy
project.task('releaseMajor', dependsOn: ['prepareMajorRelease', 'assembleRelease'])

// To ensure assembleRelease runs after prepareMajorRelease
project.tasks.whenTaskAdded { task ->
  if (task.name.equals("assembleRelease")) {
    task.mustRunAfter 'prepareMajorRelease'
  }
}
```

The above task will update the version file and then generate a release APK. You might be asking: "What version file?". In order for the plugin to know what the current version is, a basic JSON file is required. The contents of the JSON file must look like the following:

```JSON
{
  "buildNumber": 65,
  "major": 1,
  "minor": 6,
  "patch": 6
}
```

Where you place this file is up to you since it's location is configurable...

```Groovy
apply plugin: VersioningPlugin

versioning {
  versionFile = file("${project.rootDir}/config/version")
}
```

Lets take a look at more complicated example:

```Groovy
project.task('releasePatch', dependsOn: ['preparePatchRelease', 'assembleRelease']) << {
  // Add changes
  grgit.add(update: true, patterns: ['config/version']

  // Commit
  grgit.commit(message: 'Update version')
  grgit.push()

  // Tag
  def tagName = "${versioning.getVersionName()}"
  grgit.tag(name: tagName, message: 'Patch release')
  grgit.push(refsOrSpecs: [tagName])
}
```

Whoa, what's going on here? In this task, we are updating the patch version, generating a release APK, committing the updated version file and then finally, tagging the current branch... All with a single command.

Sweet! Where can I get this plugin? You can get it [here](https://gist.github.com/alexfu/e5523a4276d24c320a3a). Simply copy the file somewhere in your project and then add the following to your main `build.gradle` file:

```Groovy
apply from: 'path/to/versioning.gradle'
apply plugin: VersioningPlugin

android {
...
}

versioning {
  versionFile = file("path/to/version_file")
}

// Project specific versioning tasks go here
```

The git gradle plugin used in my example can be found [here](https://github.com/ajoberstar/grgit).
