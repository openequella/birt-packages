# BIRT OSGI runtime library and Report Framework for openEquella

openEquella requires the BIRT OSGI runtime library and BIRT Report Framework to support the integration with BIRT.
However, the full packages of these two libraries are not available on Maven central, and they can only be downloaded
from the [Eclipse BIRT download page](https://download.eclipse.org/birt/downloads/drops/). Historically, this issue was
solved by downloading the full packages, adding a variety of tweaks to make the packages work with oEQ, and publishing
to remote repository. Unfortunately, the knowledge of what tweaks were needed and how to add them was lost. 

As a result, above manual work has been done again with version 4.9.0 of the two libraries. And all the steps required to
make the libraries work with oEQ are captured in this project, which uses two sub-projects to build the BIRT OSGI runtime 
library and the BIRT Report Framework respectively. However, there is no guarantee that the steps described below will work
perfectly with any future version of the BIRT OSGI runtime library and BIRT Report Framework.

## Relationship between the two sub-projects

The Report Framework requires some Jars that are provided by the OSGI runtime library. Therefore, the OSGI runtime library
must always be built before the Report Framework.

## Setup

Before building you need to first setup a gradle.properties. This can be done simply with:

```
cp gradle.properties.sample gradle.properties
```

The values in here will need to be setup properly if you plan to publish the artefacts to OSSRH. But
even for local building you at least need them to have a value - so just copy the sample file to
start with.

(Details on publishing can be found in the 'Deploy' section.)

## Creating the BIRT OSGI runtime library

The gradle build file in 'birt-osgi' includes two tasks which will create a ZIP file for the OSGI runtime library.
* Download version 4.9.0 of the BIRT OSGI original ZIP file from the official download page.
* Unzip the archive file to folder `/osgi`.
* Download the BIRT compatibility Jar from Maven central to `/osgi/ReportEngine/platform/plugins`. The version is pinned to 1.2.500.
* Update file 'bundles.info' in `osgi/ReportEngine/platform/configuration/org.eclipse.equinox.simpleconfigurator`
by appending below line to include the info of the compatibility Jar.
  * org.eclipse.osgi.compatibility.state,1.2.500.v20210730-0750,plugins/org.eclipse.osgi.compatibility.state-1.2.500.jar,4,false
* Compress `/osgi` to a ZIP file named `birt-osgi-${project.version}`.

The ZIP file will be produced by executing the task `buildBirtOsgi`.

```
./gradlew buildBirtOsgi
```

## Creating the BIRT Report Framework

The gradle build file in 'birt-report-framework' includes two tasks which will create a ZIP file as the OSGI Report Framework.
* Download version 4.9.0 of the BIRT Report Framework original ZIP file from the official download page.
* Unzip the archive file to folder `/framework`.
* Copy Jars listed below from the BIRT OSGI sub-project to `/framework`.
  * 'org.eclipse.datatools.connectivity.oda_3.6.101.201811012051.jar'
  * 'org.eclipse.datatools.connectivity.oda.consumer_3.4.101.201811012051.jar'
  * 'org.eclipse.equinox.common_3.16.0.v20220211-2322.jar'
  * 'org.eclipse.equinox.registry_3.11.100.v20211021-1418.jar'
  * 'org.eclipse.core.runtime_3.24.100.v20220211-2001.jar'
  * 'org.apache.batik.i18n_1.14.0.v20210324-0332.jar'
  * 'org.eclipse.osgi_3.17.200.v20220215-2237.jar'
* Extract `Tidy.jar` from `org.eclipse.birt.report.engine_4.9.0.v202203150031.jar` which is in the BIRT OSGI sub-project, and copy to `/framework`.
* Compress `/framework` to a ZIP file named `birt-report-framework-${project.version}`.

The ZIP file will be produced by executing the task `buildBirtReportFramework`.

```
./gradlew buildBirtReportFramework
```

## Generate the artifacts

By running below command, the BIRT OSGI runtime library and Report Framework will be created under `birt-osgi` and `birt-report-framework`, respectively.

```
./gradlew build
```

## Clean the project

By running below command, all files except `build.gradle` in the sub-projects will be deleted.

```
./gradlew clean
```

## Deploying

The artifacts are uploaded to [Sonatype OSSRH](https://oss.sonatype.org).
Firstly, credentials must be supplied in `gradle.properties`.

```
ossrhUsername=username
ossrhPassword=password
```

Secondly, update the developer details in `gradle.properties`. The details will be included in the generated POM files.

```
developerId=yourID
developerName=yourName
developerEmail=yourEmail
```

Thirdly, update the signing details in `gradle.properties`. Please follow this [guide](https://central.sonatype.org/publish/requirements/gpg/) to generate the signing key.

```
signing.keyId=key
signing.password=password
signing.secretKeyRingFile=file
```

Then by running below command, the artifacts will be uploaded.

```
./gradlew publish
```

### Local

If you wish to do you own build and use locally, you can publish to your local machine's maven
repository (typically located at `~/.m2/repository`) with the command:

```
./gradlew publishToMavenLocal
```
