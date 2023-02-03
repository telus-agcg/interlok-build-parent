[![OWASP SCA](https://github.com/telus-agcg/interlok-build-parent/actions/workflows/owasp.yaml/badge.svg)](https://github.com/telus-agcg/interlok-build-parent/actions/workflows/owasp.yaml) [![Test](https://github.com/telus-agcg/interlok-build-parent/actions/workflows/test.yaml/badge.svg)](https://github.com/telus-agcg/interlok-build-parent/actions/workflows/test.yaml) ![license](https://img.shields.io/github/license/telus-agcg/interlok-build-parent.svg)

> This is forked from https://github.com/adaptris/interlok-build-parent and is customised for use by Telus Agriculture & Consumer Goods. That doesn't mean you can't you use it, but we will move on in slightly different ways because we intend to be in full control of the underlying Interlok version.

- `develop` is considered the trunk branch and we will mostly be referencing the develop branch.
- If we tag then the tag will match the baseline Interlok release (e.g. the next tag might be 4.6.0 to match the corresponding interlok release).
- ~~We may introduce our own OWASP exclusion file based on our own attitude to those dependencies~~, but for now will rely mostly on the exclusion file made available in [the interlok project](https://github.com/adaptris/interlok/blob/develop/gradle/owasp-exclude.xml).
  - There is now [an exclusion file](https://raw.githubusercontent.com/telus-agcg/interlok-build-parent/develop/owasp-exclude.xml) used in addition to the default
- v3/build.gradle will not be actively worked on since we aren't using it but we'll let dependabot do its thing and merge if the associated github action passes.
- v4/build.gradle is actively being worked on and We will try to track the latest stable gradle release (right now gradle-7.5.1) with it as well.
- We'll endeavour to be in a situation where we can sync with upstream, but we know that this might not always be possible.

---
# interlok-build-parent
[![GitHub tag](https://img.shields.io/github/tag/adaptris/interlok-build-parent.svg)](https://github.com/adaptris/interlok-build-parent/tags) ![license](https://img.shields.io/github/license/adaptris/interlok-build-parent.svg)

This is a gradle file that can be applied to your gradle file to simplify things bootstrapping your Interlok project. Gradle 5.x+ is required. Note that `mavenCentral()` doesn't appear to be available if we execute using gradle-5.2, but is available on gradle-5.6.3. We have tested with gradle-5.6.3 so YMMV to be honest vis-a-vis the gradle version.

## v3 vs v4

Since Interlok 3.x differs from Interlok 4.x there are separate gradle files that support their respective versions. We don't anticipate making very many more changes to the `v3/` variant and most of the work is going to be focussed on the `v4/` variant.

## Usage

```
// build.gradle
ext {
  interlokParentGradle = "https://raw.githubusercontent.com/adaptris/interlok-build-parent/main/v3/build.gradle"
}

allprojects {
    apply from: "${interlokParentGradle}"
}

dependencies {
    interlokRuntime ("com.adaptris:interlok-json:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-filesystem:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-csv-json:$interlokVersion") { changing=true }
}
```

Or you can override version:

```
// build.gradle
ext {
  interlokVersion = '3.9.2-RELEASE'
  interlokUiVersion = interlokVersion
  interlokParentGradle = "https://raw.githubusercontent.com/adaptris/interlok-build-parent/main/v3/build.gradle"
}

allprojects {
    apply from: "${interlokParentGradle}"
}

dependencies {
    interlokRuntime ("com.adaptris:interlok-json:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-filesystem:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-csv-json:$interlokVersion") { changing=true }
}
```

A full example with configuration is here : [build-parent-json-csv](https://github.com/adaptris-labs/build-parent-json-csv)

## Gradle Tasks

* `gradle clean check` will verify the configuration unmarshalls; execute service-tester if `src/test/interlok/service-test.xml` exists; and finally runs a version report.
* `gradle clean assemble` will build your distribution, you end up with an Interlok installed into `./build/distribution`
* `gradle clean build` will execute both the steps above
* `gradle dependencyCheckAnalyze` will execute the OWASP dependency check analysis; this isn't part of the normal build chain.

## Next steps

```
cd ./build/distribution
java -jar lib/interlok-boot.jar
```

## The bit you probably won't read; but you should.

* You probably want to run clean every time before a build because of the way gradle detects up-to-date files.
* `assemble` is a sync task which means it will delete everything in your distribution directory (i.e. it will delete the UI database if you're running from _build/distribution_)


### Customising your build

There is support for build environments; you can pass in a gradle property to specify the build environment `gradle -PbuildEnv=myEnv clean check`. What this does is to copy the file `variables-local-{buildenv}.properties` to variables-local.properties so that you can potentially override various settings on a per build basis.

* __If the `buildEnv` is set to the _dev_ then the UI is copied into the distribution__.
* __If the `buildEnv` is set to the _dev_ then the service tester jars are copied into the distribution in the expectation that you will be using the UI service tester page__.


The same is possible with `log4j2.xml` by creating a file with the following pattern `log4j2.xml.{buildEnv}`.

If you don't want to assemble into `./build/distribution` then you can override that location by defining a `interlokDistDirectory=` in your gradle properties (or on the commandline). We generally discourage this, unless you are only running the assemble task.

If you have a local repository that you want to use (e.g. you have custom components not in our artifact repository), then you can specify an additional property `localInterlokRepo` to point to your artifact repository e.g. `gradle -PlocalInterlokRepo=http://my.artifact.repo/groups/interlok assemble`. Any _url_ supported by gradle may be used here.

### Additional Build Specific Configuration

If you need additional build specific configuration, these can be added by setting the follow properties `additionalTemplatedConfiguration` or `additionalTemplatedProperties` in your build.gradle (in the `ext{}` block) :

```
additionalTemplatedConfiguration = [
  'jetty.xml'
]

additionalTemplatedProperties = [
  'kinesis-local'
]
```

Template files are named slightly differently to property files since the expectation is that property files may need to be maintained by the UI so it needs to fit that default convention. `log42j.xml.{buildEnv}` and `variables-local-{buildEnv}.properties` are always automagically copied and overridden; additional files may be requested by explicitly setting them.

| Setting | BuildEnv | Behaviour |
|----|----|----|
| - | dev | If `src/main/interlok/config/log4j2.xml.dev` exists then this is used; otherwise log4j2.xml is the file used during `assemble` |
| - | dev | If `src/main/interlok/config/variables-local-dev.properties` exists then this is used; otherwise variables-local.properties is the file used during `assemble` |
| additionalTemplatedConfiguration = ['jetty.xml', 'ApplicationInsights.xml' ] | dev | If `src/main/interlok/config/jetty.xml.dev` exists then this is used; otherwise jetty.xml is the file used during `assemble`. The same behaviour is applied for `src/main/interlok/config/ApplicationInsights.xml.dev` |
| additionalTemplatedProperties = ['aws' ] | dev | If `src/main/interlok/config/aws-dev.properties` exists then this is used; otherwise aws.properties is the file used during `assemble` |


### Overriding system properties / environment variables during InterlokVerify

If you are expecting environment variables and system properties to be injected at runtime by your build pipeline, then you can spoof those things by setting two additional variables in your build.gradle, `interlokVerifyEnvironmentProperties` & `interlokVerifySystemProperties` respectively; this will makes `check` work w/o you defining additional properties.

```
interlokVerifyEnvironmentProperties = [
  DB_HOST: "localhost",
  DB_PORT: "3306",
  DB_NAME: "mydatabase",
  DB_USER: "root",
  DB_PASSWORD: "honestly-this-is-the-pasword-for-the-production-database",
  AWS_S3_BUCKET: "some-bucket",
  AWS_SQS_QUEUE: "queue",
]
interlokVerifySystemProperties = [
  "org.jruby.embed.localcontext.scope": "threadsafe",
  "org.jboss.logging.provider": "slf4j"
]
```

### Detecting changes to the parent build gradle.

Since we're using `apply from`; this is effectively treated as a script plugin, which means that it is added to your local module cache (`~/.gradle/caches/modules-2` or similar). This cache is managed by the gradle daemon and any changes will be detected automatically (probably daily). However, in some situations you may be after the latest and greatest version of the parent gradle file. If that's the case then you can use the commandline switch _--refresh-dependencies_ to force gradle to refresh the state of all your dependencies : `./gradlew --refresh-dependences clean check assemble`. This should force a re-download of the parent gradle.

### Interlok License

If you're using licensed components check will fail, the the parent supports reading license keys from environment variables.

```shell
export INTERLOK_LICENSE_KEY="a.."
gradle clean check
```
