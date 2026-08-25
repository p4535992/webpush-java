# JFrog Artifactory publishing

This repository can publish the Maven release artifacts to a JFrog Artifactory local Maven repository when a GitHub Release is published.

The release workflow builds and tests the project, generates the main JAR, sources JAR and Javadoc JAR, publishes the Maven publication to Artifactory, stores the three JARs as GitHub Actions artifacts, and attaches them to the GitHub Release.

## 1. Create the Artifactory repository

A ready-to-use local Maven repository definition is stored in:

```text
.jfrog/repositories/webpush-java-local.json
```

The repository key is `webpush-java-local`. It accepts releases and rejects snapshots.

You can create it from the JFrog UI as a **Local Maven** repository, or provision it once with the Artifactory Repository REST API. Repository creation requires JFrog Admin or Project Admin permissions.

Example using an admin-scoped token:

```bash
export JFROG_URL="https://your-company.jfrog.io"
export JFROG_ADMIN_TOKEN="..."

curl --fail-with-body \
  --request PUT \
  --header "Authorization: Bearer ${JFROG_ADMIN_TOKEN}" \
  --header "Content-Type: application/json" \
  --data-binary @.jfrog/repositories/webpush-java-local.json \
  "${JFROG_URL%/}/artifactory/api/repositories/webpush-java-local"
```

Do not use the admin-scoped token for normal release publishing. Create a narrower token/user that only has deploy permissions on the release repository.

## 2. Configure GitHub

In **Settings -> Secrets and variables -> Actions**, create these repository variables:

- `JFROG_URL`: JFrog platform URL, for example `https://your-company.jfrog.io`. A URL ending in `/artifactory` is also accepted.
- `JFROG_REPOSITORY`: repository key, normally `webpush-java-local`.

Create these repository secrets:

- `JFROG_USERNAME`: JFrog user associated with the publishing token.
- `JFROG_ACCESS_TOKEN`: access/identity token with deploy permissions on `JFROG_REPOSITORY`.

## 3. Publish a release

Create a GitHub Release for a tag such as `5.1.3` or `v5.1.3` and publish it. The workflow `.github/workflows/release-jfrog.yml` starts on the `release.published` event.

The tag is the source of the published Maven version. For example, both `5.1.3` and `v5.1.3` produce:

```text
nl.martijndwars:web-push:5.1.3
web-push-5.1.3.jar
web-push-5.1.3-sources.jar
web-push-5.1.3-javadoc.jar
```

The workflow can also be started manually with **Run workflow** and an existing tag. Manual runs publish to JFrog and store workflow artifacts, but do not modify a GitHub Release.

## Local publishing test

With the same environment variables configured locally:

```bash
./gradlew clean test jar sourcesJar javadocJar -PreleaseVersion=5.1.3
./gradlew -Pjfrog -PreleaseVersion=5.1.3 publishMavenJavaPublicationToJfrogRepository
```

The existing Sonatype release flow remains available through `./gradlew -Prelease ...`.
