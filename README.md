Jenkins Pipeline for AOSP
=========================

*"This repository is a [Jenkins shared library](https://jenkins.io/doc/book/pipeline/shared-libraries/#dynamic-retrieval)"*

# How to use it ?

Importing and using this pipeline (or shared library) can be performed from
a script which will be interpreted by Jenkins.
That script can be written in two ways:

- written in a [web textarea](https://jenkins.io/doc/book/pipeline/getting-started/#through-the-classic-ui) from a job configuration page
- written in a [Jenkinsfile](https://jenkins.io/doc/book/pipeline/getting-started/#defining-a-pipeline-in-scm)

It is generally considered a best practice to create a Jenkinsfile and
check the file into the source control repository.

## Import the shared library

To import this shared library, write the following piece of code
at the beginning of the script (or before using the pipeline):

```groovy
library identifier: '@test', retriever: modernSCM([
    $class: 'GitSCMSource',
    credentialsId: '<private-key-id>',
    remote: 'https://git.smile.fr/ecs-ci/jenkins-pipeline-aosp.git',
])
```

The `credentialsId` parameter is an ID which refers to one of your credentials
in Jenkins whose users need permissions to use it (see
the credential table in the web UI: **Jenkins > Credentials**).

## Use the pipeline

Add the following code to the script and complete it:

```groovy
env = [
    "NODE_LABEL=<node-label>"
]

withEnv(env) {
    aospPipeline {
        doClean         = true
        manifestUrl     = "<git@git.domain.com:user/repository>"
        repoBranch      = "<repo-branch-name>"
        targetProduct   = "<target-name>" // ex: "aosp_x86_64"
        buildVariant    = "eng"           // ex: "user" or "userdebug"
        ccacheEnabled   = true
        ccacheSize      = "50G"
        jobCpus         = 4
    }
}
```

The `withEnv` expression injects the `env`
[Map](http://groovy-lang.org/groovy-dev-kit.html#Collections-Maps)'s variables into
the Jenkins environment which enables the pipeline to read them. The pipeline
requires the `NODE_LABEL` environment variable which will instruct Jenkins
to execute the pipeline on a specific agent/node.

## See also

For more details about arguments and options, look at the:

* [Sources](./vars/aospPipeline.groovy)
* [Jenkinsfile as example](./samples/Jenkinsfile)

# Advanced

## Pipeline's stages

1. Preparation (pre-pipeline);
1. SCM - downloading the sources with *repo*;
1. Build - building the AOSP;
1. Static Analysis - only on the building's output not the sources;
1. Unit Tests - using the Android CTS;
1. SonarQube - send static analysis, unit testing and coverage results;
1. Post (post-pipeline).

## Debugging

* Skip some stages as you please, using the `skipStages` option (ex: skip
"SCM" and "Build" stages):

```groovy
aospPipeline {
...
    skipStages = [
        scm   : true,
        build : true
    ]
}
```