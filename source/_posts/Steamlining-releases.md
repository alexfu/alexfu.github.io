title: Streamline releases with Android
date: 2015-07-16 21:03:18
categories: Android
tags:
  - Android
  - Gradle
comments: true
---
When it comes to releasing a build, it usually goes without saying that there are a few housekeeping tasks that need accomplished before an actual release goes out. Things like tagging and pushing your current branch to your remote repository could be automated so that you can focus on what’s important.

Having to do this repeatedly for every beta build of our [app](https://www.everseat.com/) became a chore and was something I didn’t want to really think about. I should be able to simply enter a single command and boom, tag and push the current branch. I eventually came up with the following:

```groovy
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'org.ajoberstar:gradle-git:1.2.0'
  }
}

apply plugin: 'org.ajoberstar.grgit'

android {
  ...
}

task releaseBeta << {
  // Ex: v1.2.3.45
  def tagName = "v${android.defaultConfig.versionName}.${android.defaultConfig.versionCode}"
  grgit.tag.add(name: tagName, message: "Beta release ${tagName}")
  grgit.push(refsOrSpecs: [tagName])
}
```

Running `./gradlew releaseBeta` will now tag my current branch and push it up for me automatically. As an added bonus, If I forgot to update the `versionName` or `versionCode`, the build will simply fail.

Simple and handy, but can we do more with this? I sought out to automate the entire versioning system, within reason. When it comes to versioning, you still would want the option to have a hand in some decisions such as when to bump the major/minor version number.

In the following examples, I will be using a very simple versioning scheme of

```bash
<major>.<minor>.<revision>.<build_number>

versionName = <major>.<minor>.<revision>
versionCode = <build_number>
```

Where we **increment** the `build_number` each time we release a beta build and **increment** both `build_number` and `revision` when we release a production build. We leave `major` and `minor` alone since we want to decide when those numbers are updated.

The first step is to decouple versioning from the Android Gradle plugin. Typically in an Android project we would specify the `versionCode` and `versionName` with hardcoded values, but since we want to be able to modify those values from a build task, we need to be able to provide those values from another source (i.e. a file).

With some elbow grease and refactoring, the following...

```groovy
android {
  defaultConfig {
    applicationId "com.myapp"
    versionCode 1
    versionName "1.2.3"
  }
}
```

Now becomes...

```groovy
android {
  defaultConfig {
    applicationId "com.myapp"
    versionCode project.ext.versionCode
    versionName project.ext.versionName
  }
}
```

Where both `project.ext.versionCode` and `project.ext.versionCode` references the `versionCode` and `versionName`. We dynamically generate `versionCode` and `versionName` from a JSON file that looks like this:

```json
{  
  "major": 1,
  "minor": 2,
  "revision": 3,
  "buildNumber": 1
}
```

Now we can work on building a Gradle task that will increment the version for our beta and production releases. First thing that needs to be done is to read in the above JSON file into an object...

```groovy
def versionFile = file("/path/to/version/file")
def versionJSON = getJSON(versionFile)
def versionCode = versionJSON.buildNumber
def versionName = "${versionJSON.major}.${versionJSON.minor}.${versionJSON.revision}"

// Expose as extra properties at project level
ext.versionCode = versionCode
ext.versionName = versionName

def getJSON(file) {
  return new JsonSlurper().parseText(file.text)
}
```

Then we can build out our task...

```groovy
task prepareBetaRelease << {
  // Ensure our working copy is clean first
  if (!grgit.status().isClean()) {
    throw new GradleException("You must NOT have any changes in your working copy!")
  }

  // Update version code
  versionJSON.buildNumber += 1
  versionCode = versionJSON.buildNumber
  versionFile.write(new JsonBuilder(versionJSON).toPrettyString())

  // Apply version code to all variants. This is necessary
  // so when we build the APK, it gets the updated values
  android.applicationVariants.all { variant ->
    variant.mergedFlavor.versionCode = versionCode
  }

  // Add changes
  def changes = grgit.status().unstaged.getAllChanges()
  grgit.add(update: true, patterns: changes)

  // Commit
  grgit.commit(message: 'Prepare for beta release')

  // Push
  grgit.push()

  // Tag
  def tagName = "v${versionName}.${versionCode}"
  grgit.tag.add(name: tagName, message: "Beta release ${tagName}")

  // Push
  grgit.push(refsOrSpecs: [tagName])
}
```

We can then take the above task, and hook it into something like HockeyApp with their Gradle plugin for distributing APKs...

```groovy
// NOTE: The order in which the tasks appear in the array does not mean
// that is the order they will run in.
task releaseBeta(dependsOn: ['prepareBetaRelease', 'uploadBetaToHockeyApp'])

// Here we explicitly specify the order in which the two tasks should run in.
// We use 'whenTaskAdded' because 'uploadBetaToHockeyApp' isn't created until
// a build starts.
tasks.whenTaskAdded { task ->
  if (task.name.equals("uploadBetaToHockeyApp")) {
    task.mustRunAfter prepareBetaRelease
  }
}
```

Now when we run `./gradlew releaseBeta`, this will do the following:

1. Increment our build number
2. Commit changed build number
3. Tag and push our current branch
4. Push the committed changes
5. Release to HockeyApp

To accomplish this for production builds, we follow the same exact procedure, except we increment the `build_number` and `revision`.
