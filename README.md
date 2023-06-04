# How we do releases of NuoDB Control Plane

There are two deliverables that are created out of the code in this repository.

1. The NuoDB Control Plane Docker image, which is published to the GitHub Container Registry using a GitHub Actions workflow.
2. The NuoDB Control Plane Helm charts within the [`charts`](/charts) directory.

The Docker image is versioned independently of the Helm charts, but all of the Helm charts are versioned together.
For example, any release that is publishing a new version of [`charts/nuodb-cp-crd`](/charts/nuodb-cp-crd) will also publish new versions of [`charts/nuodb-cp-operator`](/charts/nuodb-cp-operator), [`charts/nuodb-cp-rest`](/charts/nuodb-cp-rest), and [`charts/nuodb-cp-3ds`](/charts/nuodb-cp-3ds).

## Branching and tagging conventions

Branching and tagging conventions are used to specify which commits the deliverable artifacts for a release will be created from.
Semantic versioning is used with the form `<major>.<minor>.<patch>`.
To summarize semver conventions:

- Patch version releases can introduce bug fixes.
- Minor version releases can introduce backwards-compatible enhancements.
- Major version releases can introduce backwards-incompatible enhancements.

The tag format `v<major>.<minor>.<patch>` is used to denote a release of the Docker image.
The tag format `charts/v<major>.<minor>.<patch>` is used to denote a release of the Helm charts.

Major and minor releases are created from the `main` branch, while patch releases are created from release branches.
Branches of the form `v<major>.<minor>-dev` are used to create patch releases for the Docker image for a particular major and minor version.
Branches of the form `charts/v<major>.<minor>-dev` are used to create patch releases for the Helm charts for a particular major and minor version.

## The `relman.py` tool

The [`relman.py`](relman.py) tool can be used to prepare a release.
Currently it is capable of checking that the current branch follows the tagging conventions using `--check-current` or `--check-tags`, and it can also generate a changelog by scraping commit messages since the last commit using `--create-changelog`.
The changelog files for all releases are stored in [`changelog`](/changelog) for the Docker image and [`charts/changelog`](/charts/changelog) for the Helm charts.
These files should be checked into version control, perhaps after some manual editing, and can be used to describe the release in GitHub.

## Example: Creating a minor release

To create a minor release of the Docker image, run the following sequence of steps:

1. Switch to the `main` branch:
```sh
git checkout main
```
2. Use the `relman.py` tool to check that the current development version is larger than the last release tag and to generate the changelog:
```sh
./release/relman.py --check-tags --create-changelog
```
3. Check in the generated changelog after revising its content:
```sh
git add changelog/v<major>.<minor>.0.md
git commit -m 'Releasing version v<major>.<minor>.0'
```
4. Tag the commit and create release branch for patch releases:
```sh
git tag v<major>.<minor>.0
git branch v<major>.<minor>-dev
```
5. Once you've verified that everything is correct, push the commit and tag:
```sh
git push --tags --atomic origin main v<major>.<minor>-dev
```

To further streamline and standardize the release process, `relman.py` can be used to create the commit, tag, and release branch as follows:

1. Switch to the `main` branch:
```sh
git checkout main
```
2. Use the `relman.py` tool to check that the current development version is larger than the last release tag, commit the changelog, tag the commit, and create release branch:
```sh
./release/relman.py --check-tags --create-changelog --commit --tag --branch
```
3. Once you've verified that everything is correct, push the commit and tag:
```sh
git push --tags --atomic origin main v<major>.<minor>-dev
```

Creating a major release is identical to creating a minor release.
Creating a patch release is similar, but there is no step to create another branch, and it is only the patch version that is updated.

## Automatic publishing

The Docker image is built and published to the GitHub Container Registry using a GitHub Actions workflow which is triggered by the creation of a tag of the form `v<major>.<minor>.<patch>`.
To perform ad-hoc publishing of image, the [`docker/build.sh`](/docker/build.sh) script can be invoked with the `PUSH_REPO` environment variable, but this should be avoided since the amount of data can be retained and transferred to the GitHub Container Registry is very limited.

There is currently no automatic process for publishing the NuoDB Control Plane Helm charts.
