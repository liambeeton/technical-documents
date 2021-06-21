# Standard Release Cycle

In the beginning there was master. At the beginning of each *Release Cycle* all working- and release- branches should be created from the master branch. At the end of each *Release Cycle* all branches that were merged into master should be removed.

## Release Cycle starts

When the *Release Cycle* starts everything we do in this release should have it's origin at master, and then be taken through the various phases. There is always a definite need for a develop and a release branch. These should be created as soon as the *Release Cycle* starts:

```
$ git checkout master
$ git fetch -p
$ git pull origin master
$ git checkout -b develop
$ git push origin develop
$ git checkout -b release
$ git push origin release
```

### Preparing for the release

To make sure we are fully ready to start the sprint we need to do two things before starting our feature work: 

1. Update the version number
2. Update dependencies

#### Update the version number

To be sure that each build from this point on contains the correct version number this task should happen first. 

Check out a branch (feature/update-version) from the master branch. Edit the package.json file and add in the new version number. In most scenarios this version number should line up with the current sprint number. The version number should also adhere to [semver](https://semver.org/) standards, for example, 22.0.0. 

Create a *Merge Request* for the feature/update-version branch to be merged into develop. **Do not delete the feature branch once merged.**

#### Update dependencies

The start of the *Release Cycle* is the correct time to do dependency updates. The only time this should be done mid-release-cycle is if a serious vulnerability is discovered in one of our dependencies. 

Check out a branch (feature/update-dependencies) from the master branch. Run an npm outdated to see which dependencies are behind and how far they are behind. 

The preference is to manually bump the version numbers as opposed to just blindly doing an npm update but this is a case-by-case type of thing. After installing, re-run all tests (They should all be bundled under one npm test command) and start up the application to see that all is still good. 

Create a *Merge Request* to merge feature/update-dependencies into develop. **Do not delete the branch once merged.**

## Create features and fixes

Now we're ready to start sprint work, building those features and fixes that drive us forward. 

Create a feature/ or fix/ branch from the master branch, for example feature/back-to-top-button. Do your work on the feature branch, remembering to follow the [Commit Message Conventions](https://chris.beams.io/posts/git-commit/). 

After all work on the branch is done, create a *Merge Request* to merge the feature branch into develop. **Do not delete the feature branch once merged.**

The feature/ and fix/ branches should all trigger builds to the DEV environment automatically on merge and present an easy option to promote the deployment to QA through some pipeline script.

## Create the STAGING release

Once 

1. All our features and fixes are in the develop branch 
2. All our features are on the QA environment 
3. The QA in the team has signed off all features on the QA environment

a STAGING release should be created.

To create a STAGING release, start by creating a **release candidate** (release-v42.0.0-staging) branch from the master branch. Rebase the *release candidate* branch onto develop:

```
$ git checkout master
$ git checkout -b release-v42.0.0-staging
$ git rebase develop
$ git push origin release-v42.0.0-staging
```

Now create a *Merge Request* to merge the *release candidate* into the release branch. 

This *Merge Request* will kick off a CI pipeline. Even though we know that all code we have merged in has been unit tested and signed off by the QA, this is a good safety precaution. 

Once the *Merge Request* is merged a build to the STAGING environment should be created.

## Create the PROD release

Once 

1. A STAGING release has been created 
2. All features are fully regressed on the STAGING environment 

a PROD release should be created. 

To create a PROD release, start by creating a *release candidate* (release-v42.0.0) branch from the master branch. Rebase the *release candidate* branch on release:

```
$ git checkout master
$ git checkout -b release-v42.0.0
$ git rebase release
$ git push origin release-v42.0.0
```

Now create a *Merge Request* to merge the *release candidate* into the master branch. 

This *Merge Request* will kick off a CI pipeline. Even though we know that all code we have merged in has been unit tested and regressed by the QA, this is a good safety precaution. 

Once the *Merge Request* is merged a build to the PROD environment should be created. This should also tag the master branch with the current version number (v42.0.0)

## *Release Cycle* ends

Once a PROD release has been created the *Release Cycle* ends. 

Some cleanup is required before moving on to the next *Release Cycle*: The develop and release branches need to be deleted. All feature/ and fix/ branches that have been merged into master need to be deleted.

```
$ git checkout master
$ git fetch -p
$ git pull origin master
$ git branch -d develop
$ git push origin --delete develop
$ git branch -d release
$ git push origin --delete release
$ git branch -d feature/back-to-top-button
$ git push origin --delete feature/back-to-top-button
```

This creates a clean environment so we can start with our next *Release Cycle*.

## Anything other than standard

There are times when it's just not possible to stick to the Standard *Release Cycle*, these situations are: 

1. Hotfixes 
2. Out-of-band feature releases
3. Pulling features from a *Release Cycle*

### Hotfixes

Hotfixes take top priority. We need the ability to create a hotfix from any release that is currently on the master branch. The tags that we create on every PROD release are imperative to creating good hotfixes, and having the ability to hotfix any release. 

To start a hotfix we need to create a maintenance branch. This branch will have it's origin at the tag of the PROD release we need to fix.

#### Create the maintenance branch

First, create the maintenance branch from the tag we want to fix:

```
$ git checkout master
$ git fetch -p
$ git checkout -b maintenance v41.0.0
$ git push origin maintenance
```

#### Update the version number

Create a branch to bump the version number for the hotfix from the correct tag:

```
$ git checkout master
$ git fetch -p
$ git checkout -b feature/update-version-v41.0.1 v41.0.0
```

Bump the version number in the package.json file to the hotfix version.

Create a *Merge Request* to merge the feature/update-version-v41.0.1 branch into the maintenance branch.

#### Create the hotfix branch

Now that the maintenance branch exists, create a hotfix *release candidate* branch from the same tag:

```
$ git checkout master
$ git fetch -p
$ git checkout release-41.0.1-hotfix v41.0.0
```

#### Do the hotfix

Now that the maintenance branch exists, and we have a *release candidate* branch, do the hotfix on the *release candidate* branch. 

Create a *Merge Request* to merge the *release candidate* branch into the maintenance branch. 

Once the *release candidate* branch is merged a build to the STAGING environment should be kicked-off automatically.

#### Release the hotfix

Once the hotfix is signed off on the STAGING environment there should be a simple way to release the new changes to the PROD environment. 

The hotfix should be tagged in git with the new version number (v41.0.1).

#### Take the hotfix forward in time

Create a *Merge Request* to merge the hotfix *release candidate* branch into the develop branch. This ensures that the hotfix moves forward and makes it's way all the way back to the master branch with the next PROD release.

**Do not delete this branch until it has made it all the way to master.**

## Out-of-band feature releases

This works exactly the same as a hotfix, and we should probably start referring to it as a hot feature, as that is probably the most accurate description.

## Pulling features from a *Release Cycle*

There are situations where a client/PM/PO decides that a feature should be pulled from a *Release Cycle*, and rather be added later. This could be a problem if the given feature has already made it's way into develop. 

Luckily, because none of our branches have been deleted, we can re-create develop with only the features that we do want. To do this, we can simply delete develop, and re-create it (from master) and re-merge all our features into it.

```
$ git checkout master
$ git fetch -p
$ git pull origin master
$ git branch -D develop
$ git push origin --delete develop
$ git checkout -b develop
$ git push origin develop
```

## Carrying over the feature to a next *Release Cycle*

The feature that was pulled will probably have to be added to the next (or a subsequent) release. We must not delete this feature branch at the end of the *Release Cycle*. 

When the next *Release Cycle* starts we can simply rebase our feature branch on top of master and continue as per-normal from there.

## Summary

With the methods above we give ourselves the ability to react to hotfixes, and have automated releases, and gain the added benefit of being able to pull features from a sprint in an efficient manner.