# sbt-1.3.0-bridge-download-regression

A repository to demonstrate a sbt 1.3.0-RC1 regression with downloading the bridge sources JAR.

To reproduce the problem simply enter the sbt interactive shell and type `compile`:

```
 sbt:broken-bridge-download> compile
[info] Compiling 1 Scala source to /Users/mirco/Projects/purgatorium/broken-bridge-download/target/scala-2.12/classes ...
[info] Attempting to fetch com.triplequote:hydra-bridge_1_0:2.1.4.
[info] Updating 
[info] Resolved  dependencies
[error] ## Exception when compiling 1 sources to /Users/mirco/Projects/purgatorium/broken-bridge-download/target/scala-2.12/classes
[error] sbt.internal.inc.InvalidComponent: The compiler bridge sources CoursierModuleDescriptor ...
```

This only occurs with sbt 1.3.0-RC1, as it now uses Coursier to manage dependencies. If the sbt version is downgraded, the bridge is successfully fetched (compilation will still fail because there are missing dependencies, but that's ok and expected here).

In a nutshell, the configured `hydra-bridge` sources JAR (see `MyPlugin.scala` inside the `project` directory) is not found because the "Triplequote Maven Releases" resolver is not used by Coursier, while it is used when Ivy is the dependency manager.

One thing I noticed in the `sbt/Defaults.scala` source is that the `IvyConfiguration` is created using `fullResolvers` (https://github.com/sbt/sbt/blob/develop/main/src/main/scala/sbt/Defaults.scala#L3153), and that includes the Triplequote Maven Releases repo. My understanding is that `IvyConfiguration` instance is used to resolve the bridge component when using Ivy. I don't have a full grasp yet on what happens when Coursier is used instead of Ivy, but I suspect `fullResolvers` is not used.

It would be desiderable that sbt uses the same resolvers to download the bridge component with both Ivy and Coursier.
