---
title: ImageJA
section: Explore:Libraries
artifact: net.imagej:ij
icon: /media/icons/imagej.png
project: /software/imagej
---

ImageJA is a project that provides a clean [Git](/develop/git) history of the [ImageJ](/software/imagej) project, with a proper 'pom.xml' file so that it can be used with [Maven](/develop/maven) without hassles.

{% include aside title="Old versions of `ij.jar`" content="
* Versions up to 1.48q are [in the SciJava Maven repository](http://maven.scijava.org/content/repositories/releases/net/imagej/ij/).
* Versions starting with 1.48r are [on Maven Central](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22net.imagej%22%20AND%20a%3A%22ij%22).
" %}

## Why ImageJA?

The [ImageJ](/software/imagej) project, developed by {% include person id='rasband' %}, lives in the {% include github org='imagej' repo='imagej1' label='imagej/imagej1 repository' %} on [GitHub](/develop/github). The `imagej1` repository uses the Ant build system. Changes are pushed (at most) once per day, with a corresponding datestamp. This scheme has some drawbacks:

-   ImageJ cannot be published easily to public repositories for use as a dependency downstream.
-   The `imagej1` repository's source code does not precisely correspond to ImageJ's actual releases. Hence, that repository does not have any [Git](/develop/git) release tags.
-   Developing ImageJ in an [IDE](/develop/ides) would be more convenient if it were structured as a Maven project.

The ImageJA project is an adjusted version of ImageJ which addresses the above limitations.

## How it works

The [ij1-builds job on Travis CI](https://travis-ci.com/imagej/ij1-builds) polls the [ImageJ release notes page](https://wsr.imagej.net/notes.html) for updates. When something has changed, the job performs the following actions:

1.  Downloads the latest ImageJ source archive from the ImageJ website.
2.  Extracts the archive.
3.  Restructures the source code into a Maven project.
    -   Sources are placed in `src/main/java`.
    -   A `pom.xml` is added.
4.  Commits and pushes the result to the `master` branch of the {% include github org='imagej' repo='ImageJA' label='imagej/ImageJA repository' %} on [GitHub](/develop/github).

The push triggers the followup [job](https://travis-ci.org/imagej/ImageJA), which builds and deploys the ImageJA project to the Maven Central repository (via OSS Sonatype).

## Historical note

ImageJA was originally [launched in 2005](https://list.nih.gov/cgi-bin/wa.exe?A2=IMAGEJ;cd841de0.0510) as a *fork* of [ImageJ](/software/imagej); i.e., it was synchronized closely with ImageJ with a few changes on top:

-   When run as an applet, ImageJA is embedded.
-   The internal structure of ImageJA's recorder allows command listeners to get much more fine-grained information.
-   When launching a text editor, in many cases ImageJA will now choose Fiji's Script Editor, if available, instead of the old AWT based ImageJ editor.
-   ImageJA has an easy Plugin installer via {% include bc path='Plugins | Install PlugIn...'%} (ImageJ only has that drag-n-drop thingie).
-   The instance listener is RMI-based with ImageJA, so there is no security issue with it.
-   ImageJA's Command Launcher has fuzzy matching, too.
-   A couple of bug fixes:
    -   JavaScript in ImageJA can find plugin classes, too.
    -   ImageJA also put back some not-yet-deprecated methods as deprecated.
    -   A simple bug fix in PolygonRoi drawing (it moves to the first point, but then draws a line to the same first point rather than the second).
    -   A little bug fix in StackWindow: if you have a 2D time lapse, ImageJ will still use the zSelector (rather than the tSelector).
    -   ImageJA can handle https:// URLs, too.

However, these days, needed changes to ImageJ are instead patched at runtime; see the [ImageJ Legacy](/libs/imagej-legacy) page for details.
