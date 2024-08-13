# BIRT OSGI runtime library and Report Framework for openEquella

openEquella requires the BIRT OSGI runtime library and BIRT Report Framework to support the integration with BIRT.
However, the full packages of these two libraries are not available on Maven central, and they can only be downloaded
from the [Eclipse BIRT download page](https://download.eclipse.org/birt/downloads/drops/). Historically, this issue was
solved by downloading the full packages, adding a variety of tweaks to make the packages work with oEQ, and publishing
to remote repository. Unfortunately, the knowledge of what tweaks were needed and how to add them was lost. 

As a result, above manual work has been done again with version 4.16.0 of the two libraries. And all the steps required to
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
* Download version 4.16.0 of the BIRT OSGI original ZIP file from the official download page.
* Unzip the archive file to folder `/osgi`.
* Compress `/osgi` to a ZIP file named `birt-osgi-${project.version}`.

The ZIP file will be produced by executing the task `buildBirtOsgi`.

```
./gradlew buildBirtOsgi
```

## Creating the BIRT Report Framework

The gradle build file in 'birt-report-framework' includes two tasks which will create a ZIP file as the OSGI Report Framework.
* Download version 4.16.0 of the BIRT Report Framework original ZIP file from the official download page.
* Unzip the archive file to folder `/framework`.
* Copy Jars listed below from the BIRT OSGI sub-project to `/framework`.
  * 'org.apache.batik.i18n_1.14.0.v20210324-0332.jar'
  * 'org.eclipse.birt.report.engine.emitter.config.excel_4.16.0.v202406141054.jar'
  * 'org.eclipse.birt.report.engine.emitter.prototype.excel_4.16.0.v202406141054.jar'
  * 'org.eclipse.core.runtime_3.31.100.v20240524-2010.jar'
  * 'org.eclipse.datatools.connectivity_1.15.0.202311071249.jar'
  * 'org.eclipse.datatools.connectivity.oda_3.7.0.202311071249.jar'
  * 'org.eclipse.datatools.connectivity.oda.consumer_3.5.0.202311071249.jar'
  * 'org.eclipse.emf.common_2.30.0.v20240314-0928.jar'
  * 'org.eclipse.emf.ecore_2.36.0.v20240203-0859.jar'
  * 'org.eclipse.emf.ecore.xmi_2.37.0.v20231208-1346.jar'
  * 'org.eclipse.emf.ecore.change_2.16.0.v20231208-1346.jar'
  * 'org.eclipse.equinox.common_3.19.100.v20240524-2011.jar'
  * 'org.eclipse.equinox.preferences_3.11.100.v20240327-0645.jar'
  * 'org.eclipse.equinox.registry_3.12.100.v20240524-2011.jar'
  * 'org.eclipse.osgi_3.20.0.v20240509-1421.jar'
  * 'org.eclipse.birt.report.data.oda.jdbc_4.16.0.v202406141054/oda-jdbc.jar'
* Extract `Tidy.jar` from `org.eclipse.birt.report.engine_4.16.0.v202406141054.jar` which is in the BIRT OSGI sub-project, and copy to `/framework`.
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
