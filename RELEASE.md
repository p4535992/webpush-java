# Release process

0. Update `CHANGELOG.md` with source-code changes and compile-dependency changes.

1. Choose the new release version. JitPack uses the Git tag as the Maven version.

2. Build and validate locally if desired:

```bash
GROUP=com.github.p4535992 \
ARTIFACT=webpush-java \
VERSION=5.1.4 \
./gradlew clean build publishToMavenLocal
```

This is the same Maven publication model JitPack uses. The expected coordinates are:

```text
com.github.p4535992:webpush-java:5.1.4
```

and the publication includes the main JAR, sources JAR and Javadoc JAR.

3. Create and publish a GitHub Release for the version tag, for example `5.1.4`.

Publishing a GitHub Release automatically runs `.github/workflows/release.yml`. The workflow:

- runs the tests with JDK 11;
- builds the main JAR, sources JAR and Javadoc JAR;
- runs `publishToMavenLocal` with JitPack-compatible coordinates;
- verifies the generated Maven POM/JARs;
- stores the JARs as GitHub Actions artifacts;
- attaches the three JARs to the GitHub Release.

No repository credentials or publishing secrets are needed for JitPack. JitPack builds the public GitHub tag on demand and exposes it as:

```text
https://jitpack.io
com.github.p4535992:webpush-java:<tag>
```

For convenience, changing `.github/release-version` on `master` also runs the release workflow. If that release/tag does not exist, the workflow creates it at the triggering commit and attaches the generated artifacts.

4. Consumers add JitPack and the dependency:

```groovy
repositories {
    mavenCentral()
    maven { url 'https://jitpack.io' }
}

dependencies {
    implementation 'com.github.p4535992:webpush-java:5.1.4'
}
```

The first dependency request causes JitPack to build the tag if it has not already been built.

5. Maven Central remains optional and uses the historical Sonatype flow:

```bash
./gradlew -Prelease clean publish
./gradlew -Prelease closeAndReleaseRepository
```
