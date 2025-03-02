---
title: Github Actions
section: Extend:Development:Tools
---

[GitHub](/develop/github) supports workflow automation in the cloud through [Github Actions](https://github.com/features/actions). This feature is very useful for automating builds, deployment and other aspects of [continuous integration](/develop/project-management#continuous-integration).  

# Services

[SciJava](/libs/scijava) projects use Github Actions in a variety of ways:

-   Perform builds of SciJava projects. Github Actions deploy `SNAPSHOT` builds to the [SciJava Maven repository](https://maven.scijava.org/) in response to pushes to each code repository's `master` branch. So any downstream projects depending on a version of `LATEST` for a given component will match the last successful Github build—i.e., the latest code on `master`.
-   Run each project's associated {% include wikipedia title='Unit testing' text='unit tests'%}. Github Actions is instrumental in early detection of new bugs introduced to the codebase.
-   Perform [releases](/develop/releasing) of [SciJava](/libs/scijava) projects. Github Actions deploys release builds to the appropriate Maven repository—typically either the SciJava Maven repository or [OSS Sonatype](https://oss.sonatype.org/).
-   Keep the [javadoc](/develop/source#javadocs) site updated.
-   Keep other web resources updated.

# Automatic Deployment of Maven Artifacts

Deploying your library to a [Maven](/develop/maven) repository makes it available for other developers. It is also a [contribution requirement for the Fiji project](/contribute/fiji).

## Requirements

-   Host your [open-source](/licensing/open-source) project on [GitHub](/develop/github).
-   Contact an ImageJ admin in [Gitter](/discuss/chat#gitter) or [the Image.sc Forum](http://forum.image.sc/) and request that they add the authentication secrets for deployment to your organization.

## Instructions

-   In order to add Github CI support to a repository, the secrets are needed: A) for deploying to Maven repositories; and B) in the case of deploying to OSS Sonatype, for GPG signing of artifacts. 
-   If the secrets have been added to your organization, and you have push access to the relevant repository on GitHub, you can use the [github-actionify.sh script](https://github.com/scijava/scijava-scripts/blob/master/github-actionify.sh) with the -f flag to perform the needed operations. 
-   The github-actionify script will return '[ERROR] Dirty working copy' if you have uncommited changes. If you get this error, check the status of the repository with `git status` and then run `github-actionify -f` again.
-   If you need help, please ask [on the Image.sc Forum](https://forum.image.sc/) in the Development category, or in the [scijava-common channel](https://gitter.im/scijava/scijava-common) on Gitter.

## Configuration of JavaFX builds

-   The workflows setup by the [github-actionify.sh script](https://github.com/scijava/scijava-scripts/blob/master/github-actionify.sh) do not include JavaFX. 
-   To add support for JavaFX edit the files contained in the folder `.github/workflows/` to match those found in other SciJava projects that depend on JavaFX, e.g. [FilamentDetector](https://github.com/fiji/FilamentDetector).

## Testing things which cannot run headless

If your tests require a display (i.e.: they do not pass when run [headless](/learn/headless)), you can use [Xvfb](/learn/headless#xvfb) as follows:

    before_script:
      - "export DISPLAY=:99.0"
      - "sh -e /etc/init.d/xvfb start"
      - sleep 3 # give xvfb some time to start

Of course, you should do this only as a last resort, since the best unit tests should not require a display in the first place.

