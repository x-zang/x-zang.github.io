---
layout: post
giscus_comments: true
related_posts: false
title: 'Nextflow StackOverflowError due to undefined workflow metadata'
date: 2025-08-24
tags:
  - nextflow
category:
  - troubleshoot
---

I was developing a *nextflow* workflow and wanted to get the directory where my `main.nf` is located inside the workflow. I need to run a few Python scripts from the same directory (without hard-coding the absolute path) as part of the workflow.

Cursor suggested a simple variable `workflow.scriptDir` to retrieve this metadata.

```groovy
// test_wrong.nf
workflow {
    println "Starting workflow"
    project_dir = workflow.scriptDir
    println "Project directory: ${project_dir}"
}
```

However, running this simple workflow resulted in a stack overflow error:

```sh
ERROR ~ Unexpected error [StackOverflowError]
```

This is quite strange. I was just trying to get the metadata. It does not even involve any computation. 

The `.nextflow.log` does not help either. It repeated the following error messages hundreds of times before overflowing:

```
at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.base/java.lang.reflect.Method.invoke(Method.java:569)
at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:343)
at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:328)
at groovy.lang.MetaClassImpl.getProperty(MetaClassImpl.java:1989)
at groovy.lang.MetaClassImpl.getProperty(MetaClassImpl.java:3898)
at groovy.lang.GroovyObject.getProperty(GroovyObject.java:50)
at org.codehaus.groovy.runtime.InvokerHelper.getProperty(InvokerHelper.java:167)
at nextflow. script.WorkflowMetadata.get(WorkflowMetadata.groovy:357)
at jdk.internal.reflect.GeneratedMethodAccessor15.invoke(Unknown Source)
```

## Fixing...

I just went ahead and asked Cursor to fix it for me. It suggested several solutions (spoiler: none worked), such as: 

- moving the variable to a process/ saving values to a file
- updating/uninstall/re-install java/openjdk
- updating/uninstall/re-install nextflow

I tried for a few hours before I found a related discussion in [github issue](https://github.com/nextflow-io/nextflow/issues/5986). It turned out that I should use `projectDir` rather than `scriptDir`. By default, `workflow.projectDir` returns the directory of `main.nf`. So here is a correct version.

```groovy
// test_correct.nf
workflow {
    println "Starting workflow"
    project_dir = workflow.projectDir
    println "Project directory: ${project_dir}"
}
```

It is unclear why accessing an invalid variable will invoke a stack overflow. A better (or correct) error message should be `Variable not found`.

## Take-home + TL;DR

Only 4 constants are *globally* available (retrieved from [doc](https://www.nextflow.io/docs/latest/config.html#constants) in Aug 2025).

- `baseDir: Path`

  *Deprecated since version 20.04.0.*

  Alias for `projectDir`.

- `launchDir: Path`

  The directory where the workflow was launched.

- `projectDir: Path`

  The directory where the main script is located.

- `secrets: Map<String,String>`

  Map of pipeline secrets. See [Secrets](https://www.nextflow.io/docs/latest/secrets.html#secrets-page) for more information.

More runtime metadata is documented at <https://dokk.org/documentation/nextflow/v23.04.4/metadata/>

Attempting access to undefined `workflow.metadata_variable` will give you a stack overflow. 

## Software versions 

```sh
# java -version
openjdk version "17.0.15" 2025-04-15
OpenJDK Runtime Environment Homebrew (build 17.0.15+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.15+0, mixed mode, sharing)
# nextflow -v
nextflow version 25.04.6.5954
```

