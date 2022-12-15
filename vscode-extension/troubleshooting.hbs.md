# Troubleshoot Tanzu Developer Tools for VS Code

This topic describes what to do when encountering issues with Tanzu Developer Tools for VS Code.

## <a id='cannot-view-workloads'></a> Unable to view workloads on the panel when connected to GKE cluster

{{> 'partials/ext-tshoot/cannot-view-workloads' }}

## <a id='lu-not-working-classversion'></a> Live update fails with `UnsupportedClassVersionError`

### Symptom

After live-update has synchronized changes you made locally to the running workload, the workload pods
start failing with an error message similar to the following:

```console
Caused by: org.springframework.beans.factory.CannotLoadBeanClassException: Error loading class
[com.example.springboot.HelloController] for bean with name 'helloController' defined in file
[/workspace/BOOT-INF/classes/com/example/springboot/HelloController.class]: problem with class file
or dependent class; nested exception is
java.lang.UnsupportedClassVersionError: com/example/springboot/HelloController has been compiled by
a more recent version of the Java Runtime (class file version 61.0), this version of the
Java Runtime only recognizes class file versions up to 55.0
```

### Cause

The classes produced locally on your machine are compiled to target a newer Java virtual machine (JVM).
The error message mentions `class file version 61.0`, which corresponds to Java 17.
The buildpack, however, is set up to run the application with an older JVM.
The error message mentions `class file versions up to 55.0`, which corresponds to Java 11.

The root cause of this is a misconfiguration of the Java compiler that VS Code uses.
This issue seems to be caused by a suspected bug in the VS Code Java tooling, which sometimes fails
to properly configure the compiler source and target compatibility-level from information in the
Maven POM.

For example, in the `tanzu-java-web-app` sample application the POM contains the following:

```java
<properties>
        <java.version>11</java.version>
        ...
</properties>
```

This correctly specifies that the app must be compiled for Java 11 compatibility.
However, the VS Code Java tooling sometimes fails to take this information into account.

### Solution

Force the VS Code Java tooling to re-read and synchronize information from the POM:

1. Right-click on the `pom.xml` file.
2. Select **Reload Projects**.

This causes the internal compiler level to be set correctly based on the information from `pom.xml`.
For example, Java 11 in `tanzu-java-web-app`.

## <a id="live-update-timeout"></a> Timeout error when Live Updating

### Sympton
When a user attempts to Live Update their workload, they may get the following error in the logs: 

`ERROR: Build Failed: apply command timed out after 30s - see }}{{https://docs.tilt.dev/api.html#api.update_settings{{ for how to increase}}`

### Cause

Kubernetes times out on upserts over 30 seconds.

### Solution

Add `update_settings (k8s_upsert_timeout_secs = 300)` to the Tiltfile. See Tiltfile [docs](https://docs.tilt.dev/api.html#api.update_settings).

## <a id="deprecated-task"></a> Task related error when using launch configs

### Sympton
When a user attempts to use a launch config, they may get a task related error: 

`Could not find the task 'tanzuManagement: Kill Port Forward my-app`

### Cause

Some previous tasks are no longer supported.

### Solution

Delete that task in the launch config.