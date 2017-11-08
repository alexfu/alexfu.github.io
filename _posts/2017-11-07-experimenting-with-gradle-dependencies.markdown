---
layout: post
title: "Experimenting with Gradle dependencies"
date: 2017-11-07
categories: android
---
Today, I opened my `app/build.gradle` file and cringed at the sight of my `dependencies` block. So unorganized and just hard to look at.

This is something that's been bothering me for sometime but I never really had the time to figure out a better way, until now.

I started out researching what others were doing with their build file. One of the most common patterns I've been seeing recently looks something like this:

```
dependencies {
  implementation deps.libraryA
  implementation deps.libraryB
}
```

Where `deps` is defined like so:

```
ext.deps = [
  libraryA: "com.example:library-a:1.0",
  libraryB: "com.example:library-b:1.0"
]
```

Although this cleans up the dependencies block a bit, I find the need to map the dependency string to a map key slightly redundant. I then thought, "What if I could organize my dependencies by feature?", similarly to how you would organize packages by feature.

My dependencies block already had comments, creating sections like UI, Data, etc., so it seemed like a good direction to go in.

```
dependencies {
  // UI
  implementation "com.android.support:appcompat-v7:27.0.0"
  implementation "com.android.support.constraint:constraint-layout:1.1.0-beta3"
  implementation "com.android.support:design:27.0.0"

  // Network
  implementation "com.squareup.moshi:moshi-adapters:1.5.0"
  implementation "com.squareup.retrofit2:retrofit:2.3.0"
}
```

With this in mind, I thought about what this would look like and I ended up with the following:

```
dependencies {
  ui()
  network()
}
```

Looks great! But, how do we get there? What does `ui()` look like?

```
def ui() {
  implementation "com.android.support:appcompat-v7:27.0.0"
  implementation "com.android.support.constraint:constraint-layout:1.1.0-beta3"
  implementation "com.android.support:design:27.0.0"
}
```

The above method doesn't work because `implementation` is scoped to the `dependencies` block. So the question becomes, how do we get access to `implementation` outside of the `dependencies` block?

Looking at the Gradle DSL documentation, we see that the [`dependencies`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:dependencies(groovy.lang.Closure)) block delegates to [`DependencyHandler`](https://docs.gradle.org/current/javadoc/org/gradle/api/artifacts/dsl/DependencyHandler.html) which we can get access to from [`Project.getDependencies()`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:dependencies). So with that knowledge, we can modify our ui method to look like...

```
def ui() {
  getDependencies().add("implementation", "com.android.support:appcompat-v7:27.0.0")
}
```

So far so good, but it's still hard to look at. We can refactor the above to conform to Groovy conventions.

```
def ui() {
  dependencies.add("implementation", "com.android.support:appcompat-v7:27.0.0")
}
```

A little better, but ideally we should be able to declare the dependency as if it were inside the `dependencies` block.

If we're able to access the `DependencyHandler` APIs from the `dependencies` block, then we should be able to access the same API from `getDependencies()`. So we rewrite our `ui` method again like so:

```
def ui() {
  dependencies.implementation "com.android.support:appcompat-v7:27.0.0"
}
```

And... we see that everything compiles successfully! So now, our build file looks something like this:

```
def ui() {
  dependencies.implementation "com.android.support:appcompat-v7:27.0.0"
  dependencies.implementation "com.android.support.constraint:constraint-layout:1.1.0-beta3"
  dependencies.implementation "com.android.support:design:27.0.0"
}

def network() {
  dependencies.implementation "com.squareup.moshi:moshi-adapters:1.5.0"
  dependencies.implementation "com.squareup.retrofit2:retrofit:2.3.0"
}

dependencies {
  ui()
  network()
}
```

To summarize, we took a standard `dependencies` block and grouped the dependencies by feature. We then moved the declarations out into methods and leveraged the Gradle API to be able to access the same `DependencyHandler` API.

**DISCLAIMER**: The method outlined above is completely experimental and may or may not be the best way, but it works pretty well for my purposes.
