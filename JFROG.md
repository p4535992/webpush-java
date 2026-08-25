# JFrog Artifactory publishing

This repository can build a GitHub Release and publish the Maven release artifacts to a JFrog Artifactory local Maven repository.

The release workflow builds and tests the project, generates the main JAR, sources JAR and Javadoc JAR, stores them as GitHub Actions artifacts, attaches them to the GitHub Release, ensures the target Artifactory Maven repository exists, and then publishes the Maven publication to JFrog.

## 1. JFrog repository

A local Maven repository definition is stored in:

```text
.jfrog/repositories/webpush-java-local.json
```

The default repository key is `webpush-java-local`. It accepts releases and rejects snapshots.

During a release, the GitHub Action first checks whether the target repository exists. If it does not exist, the workflow attempts to create it using the Artifactory Repository REST API.

Automatic creation requires JFrog Admin or Project Admin permissions. If the token only has deploy permission, create the repository once from the JFrog UI or with an admin-scoped token, then keep the narrower deploy token in GitHub Actions.

Example manual provisioning:

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

## 2. Configure GitHub

In **Settings -> Secrets and variables -> Actions**, create this repository variable:

- `JFROG_URL`: JFrog platform URL, for example `https://your-company.jfrog.io`. A URL ending in `/artifactory` is also accepted.

Optional repository variable:

- `JFROG_REPOSITORY`: repository key. If omitted, `webpush-java-local` is used.

Create these repository secrets:

- `JFROG_USERNAME`: JFrog user associated with the publishing token.
- `JFROG_ACCESS_TOKEN`: access/identity token with deploy permission on the target repository. To let the workflow create the repository automatically, this identity also needs repository-creation privileges.

## 3. Publish a release

There are three supported entry points.

### Release request from master

Set `.github/release-version` to the version to publish, for example:

```text
5.1.3
```

A push of that file to `master` runs the complete pipeline. If the GitHub Release does not exist, the workflow creates it; if it already exists, it reuses it and replaces the attached JAR assets.

### Existing GitHub Release

Publishing a GitHub Release for a tag such as `5.1.3` or `v5.1.3` also starts the workflow through `release.published`.

### Manual run

The workflow can be started with **Run workflow** and an existing tag.

The release/tag value controls the Maven version. For example `5.1.3` produces:

```text
nl.martijndwars:web-push:5.1.3
web-push-5.1.3.jar
web-push-5.1.3-sources.jar
web-push-5.1.3-javadoc.jar
```

## Local publishing test

With the JFrog environment variables configured locally:

```bash
./gradlew clean test jar sourcesJar javadocJar -PreleaseVersion=5.1.3
./gradlew -Pjfrog -PreleaseVersion=5.1.3 publishMavenJavaPublicationToJfrogRepository
```

`JFROG_REPOSITORY` is optional locally too; it defaults to `webpush-java-local`.

The existing Sonatype release flow remains available through `./gradlew -Prelease ...`.
