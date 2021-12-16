Instructions for building a Language Tool fat JAR
=================================================

This repository contains instructions on how to build a self-contained, stand-alone JAR file ("fat JAR") for [Language Tool](https://github.com/languagetool-org/languagetool).

This JAR is required by [TeXtidote](https://github.com/sylvainhalle/textidote); as a result, the repository hosts pre-compiled versions of the fat JAR that are referenced in TeXtidote build scripts.

## Build LanguageTool

The first step is to follow the instructions for [building LanguageTool](https://github.com/languagetool-org/languagetool) from the sources.

Clone the repository

    git clone --depth 5 https://github.com/languagetool-org/languagetool.git

In the root project folder, run:

    mvn clean test
    ./build.sh languagetool-standalone package -DskipTests

## Collate JARs

The files needed to create the fat JAR are dispersed in many folders. Put together in the same folder the JAR files found in the following locations (where `Y.Y` is the current Language Tool version, e.g. "5.6"):

- `languagetool-standalone/target/LanguageTool-Y.Y-SNAPSHOT/LanguageTool-Y.Y-SNAPSHOT/languagetool.jar`
- `languagetool-standalone/target/LanguageTool-Y.Y-SNAPSHOT/LanguageTool-Y.Y-SNAPSHOT/libs/*.jar`,
  **except** `hamcrest-core.jar` and `junit.jar` (very important, otherwise these files interfere with the libraries used by TeXtidote's unit tests)
- `languagetool-language-modules/XX/target/language-XX-Y.Y-SNAPSHOT.jar`, where `XX` is a two-letter language code

## Generate the fat JAR

Once all these files have been collated into the folder, this Ant build file can be used to extract and bundle them all (save it as `build.xml`).

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project name="Fat JAR" default="jar" basedir=".">
    <target name="jar">
      <jar destfile="../languagetool-bundle-5.6.jar" filesetmanifest="skip">
	    <manifest>
	      <attribute name="Class-Path" value="."/>
	    </manifest>
	    <zipgroupfileset dir=".">
	      <include name="*.jar" />
	    </zipgroupfileset>
      </jar>
    </target>
</project>
```

The JAR can then be built by simply running:

    ant

## Fix the JAR

The bundled JAR is unusable in that state. Two issues must be fixed manually.

First, one of the included libraries is signed and merging it into one fat JAR destroys the signature. This causes Java to ignore all unsigned classes (basically everything else). The fix (thanks to this [StackOverflow post](https://stackoverflow.com/a/51456080)) is to remove these files:

- `META-INF/*.DSA`
- `META-INF/*.RSA`
- `META-INF/*.SF`

Second, each of the `language-XX-Y.Y-SNAPSHOT.jar` has its own file called `META-INF/org/languagetool/language-module.properties`, which contains text lines like this:

    languageClasses=org.languagetool.language.Arabic

Since each of these files corresponds to the same location in the fat JAR's directory structure, they overwrite each other and only a single of them subsists --causing all other languages to become unknown to Language Tool. The workaround is to put into `language-module.properties` the *merged* contents of all these files:

```
languageClasses=org.languagetool.language.Arabic
languageClasses=org.languagetool.language.Asturian
languageClasses=org.languagetool.language.Belarusian
languageClasses=org.languagetool.language.Breton
languageClasses=org.languagetool.language.Catalan,org.languagetool.language.ValencianCatalan
languageClasses=org.languagetool.language.Chinese
languageClasses=org.languagetool.language.Danish
languageClasses=org.languagetool.language.Dutch
languageClasses=org.languagetool.language.English,org.languagetool.language.AmericanEnglish,org.languagetool.language.AustralianEnglish,org.languagetool.language.BritishEnglish,org.languagetool.language.CanadianEnglish,org.languagetool.language.NewZealandEnglish,org.languagetool.language.SouthAfricanEnglish
languageClasses=org.languagetool.language.Esperanto
languageClasses=org.languagetool.language.French
languageClasses=org.languagetool.language.Galician
languageClasses=org.languagetool.language.German,org.languagetool.language.AustrianGerman,org.languagetool.language.GermanyGerman,org.languagetool.language.SimpleGerman,org.languagetool.language.SwissGerman
languageClasses=org.languagetool.language.Greek
languageClasses=org.languagetool.language.Italian
languageClasses=org.languagetool.language.Japanese
languageClasses=org.languagetool.language.Persian
languageClasses=org.languagetool.language.Polish
languageClasses=org.languagetool.language.Portuguese,org.languagetool.language.BrazilianPortuguese,org.languagetool.language.PortugalPortuguese
languageClasses=org.languagetool.language.Romanian
languageClasses=org.languagetool.language.Russian
languageClasses=org.languagetool.language.Slovak
languageClasses=org.languagetool.language.Slovenian
languageClasses=org.languagetool.language.Spanish
languageClasses=org.languagetool.language.Swedish
languageClasses=org.languagetool.language.Ukrainian
```

Save these contents as `language-module.properties`, and add this file to the archive (using an archive manager) into the folder `META-INF/org/languagetool` (deleting existing files with the same name beforehand).

<!-- :wrap=soft: -->