# Making a release

Releasing a new version of Dicio is not only a matter of building and publishing an APK. There are a few things to do beforehand, such as updating the translations, making sure the metadata is correct, preparing a changelog, testing out the app. [^note]

[^note]: This page is partially inspired by [NewPipe's release instructions](https://teamnewpipe.github.io/documentation/07_release_instructions/).

## Preliminary steps

### Permissions

Having the following permissions is required to perform all of the steps outlined in this page, however if you are missing either of them you can ask someone who has them to perform those steps for you.

1. Have admin rights on Weblate
    - You should be able to access [Weblate's Maintenance page](https://hosted.weblate.org/projects/dicio-android/#repository)
    - Tip: if the correct page does not show up when clicking that URL, make sure you are logged in ;-)
2. Have at least maintainer rights on the dicio-android repo

Steps requiring permission are annotated with [^weblate] or [^repo].

[^weblate]: Requires permissions on Weblate, see [Permissions](#permissions)

[^repo]: Requires permissions on the dicio-android repo, see [Permissions](#permissions)

### Repositories

- Have a cloned Dicio local repository (for the rest of the page, `origin` is assumed to be the remote at `github.com/DicioTeam/dicio-android`)
- Add the `weblate` remote to the same local repository (the URL used below can be found on the Maintenance page on Weblate)
    - `git remote add weblate https://hosted.weblate.org/git/dicio-android/strings/`
- Switch to the `main` branch and make sure it is up-to-date with the remote:
    - `git checkout main`
    - `git pull origin main`

### Version name and conventions

- Find the version code of the next release by looking for `versionCode` in [`app/build.gradle.kts`](https://github.com/DicioTeam/dicio-android/blob/main/app/build.gradle.kts): You will add 1 to that value (from now on called `NEW_VERSION_CODE`) to get the new value (but do not edit the file yet)
- Choose the version number of the next release according to [semantic versioning](https://semver.org/) (from now on called `X.X.X`)

## Merging any remaining PRs

Since releases don't happen too often, it makes sense to merge any Pull Request that is either important (e.g. it provides a fix for a crash) or small (e.g. it just adds some translations), instead of delaying it to the next release cycle. *If you don't have much time available, you can skip this step.*

If you have write access to the repository, then approve and merge the PRs yourself [^repo], otherwise approve them and ask a maintainer to merge them.

## Preparing a changelog

Two changelogs need to be prepared: the GitHub one, with all information, and the F-Droid/PlayStore one, that lists the essential information.

For both changelogs, use these style rules:
- Capitalize the first letter of each item
- Use English verbs as if you were asking someone to do something, so for example use "Fix abc" and not "Fixed abc"; this allows saving a few characters and using a consistent style
- Prepend `[SKILL_NAME]` to the items that have to do with a specific skill

### GitHub changelog

The GitHub changelog is what appears under the [releases page](https://github.com/DicioTeam/dicio-android/releases). Prepare the changelog offline and keep it around for later. It is also possible to [create a draft release](https://github.com/DicioTeam/dicio-android/releases/new) on GitHub [^repo].

- Use the following structure:
    ```md
    ### New
    - …

    ### Improved
    - …

    ### Fixed
    - …

    ### Development
    - …
    ```
- Go through [the commit history](https://github.com/DicioTeam/dicio-android/commits/main/) to find out which pull requests were merged and which other commits happened that could be relevant for a changelog
- List new features, improved features/behaviors, bug fixes, and non-user-facing changes under the corresponding sections (sections with no points can be removed)
- List any new languages that were added in the "New" section
- Append `#1234` to the item(s) that were done in pull request 1234 (there can even be multiple PRs)
- Append `(thanks @USERNAME!)` if `@USERNAME` contributed to a item

For example:
```md
- [Weather] Add new provider #456 (thanks @bestContributor!)
```
or (for a change directly committed)
```md
- Update all dependencies 
```

### F-Droid changelog

- Create a new English changelog in the [`fastlane/metadata/android/en-US/changelogs/`](https://github.com/DicioTeam/dicio-android/tree/main/fastlane/metadata/android/en-US/changelogs/) folder
- The file should be named `NEW_VERSION_CODE.txt`, using the new version code found in the [Version name and conventions](#version-name-and-conventions)
- The file should have this structure (sections with no points can be removed):
    ```txt
    New
    • ...

    Improved
    • ...

    Fixed
    • ...
    ```
- Make sure you use the `•` for points (it looks nicer than `-`)
- Summarize only the most important changes from the draft release
- Make sure the file size is **at most 500 bytes**, in order to **fit [F-Droid's changelog size limit](https://f-droid.org/en/docs/All_About_Descriptions_Graphics_and_Screenshots/#fastlane-structure) (!)**
    - Tip: removing the newline at the end of the file saves 1 byte ;-)
- Commit the file on the `main` branch (try to stick to the provided commit message template)
    - `git add fastlane/metadata/android/en-US/changelogs/NEW_VERSION_CODE.txt`
    - `git commit -m "Add changelog for vX.X.X (NEW_VERSION_CODE)"`
- If you are an admin of the dicio-android repo, just push the changes to the remote `main`
    - `git push origin main` [^repo]
    - If you are not an admin, create a pull request normally and ask a maintainer to merge it

## Ensuring metadata is up-to-date

- The app descriptions may list outdated information about supported skills and supported languages.  
- The sources of truth are [`SkillHandler.kt`](https://github.com/DicioTeam/dicio-android/blob/main/app/src/main/kotlin/org/stypox/dicio/eval/SkillHandler.kt#L46) for skills and [`language.proto`](https://github.com/DicioTeam/dicio-android/blob/main/app/src/main/proto/language.proto) for languages.  
- Check whether both skill and language lists are up-to-date in the [repository's README](https://github.com/DicioTeam/dicio-android/blob/main/README.md) and in [Fastlane's English metadata](https://github.com/DicioTeam/dicio-android/blob/main/fastlane/metadata/android/en-US/full_description.txt).
- Make any changes needed to bring them up-to-date, and again either push directly to `main` [^repo] or open a pull request

## Testing for regressions

Before releasing, you should check whether there are any regressions, i.e. new bugs or crashes that were not there in the previous version, *not* bugs that were already known or present in previous versions.

- Build and run the app on your physical phone e.g. using Android Studio or downloading the latest debug APK built by [Continuous Integration](https://github.com/DicioTeam/dicio-android/actions/workflows/ci.yml?query=branch%3Amain) on the `main` branch.
- Try to use the app in many ways and see if anything goes wrong (e.g. try to use every skill, try using STT and TTS, try to switch language, etc.).
- If you find a regression that is either impactful (e.g. app crashes on an important feature) or simple to fix (e.g. a missing null check), open a pull request for it.
- Otherwise, document the regression in a GitHub issue. If it's easy to determine who the author of the regression was, ping them and ask them to provide a fix.
- The release should be delayed if there are significant regressions that need to be fixed, otherwise they risk becoming permanently part of the codebase.

## Pulling translations from Weblate

Do this as one of the last steps before release, to incorporate as many translations as possible.

### Prepare Weblate <small>[^weblate]</small>

1. Go to [Weblate's Maintenance tab](https://hosted.weblate.org/projects/dicio-android/#repository)
2. Press the *Lock* button to prevent translators from translating while you are creating commits; remember to *Unlock* later!
3. Press the *Update* button to update Weblate with the latest changes on dicio-android's `main` branch
4. Press the *Commit* button, if needed, to make sure Weblate creates a commit for translations which have not been committed yet

### Pull changes

Now go **back to the local git repository**:

5. In case you followed these steps before, delete the `weblate-main` branch
    - `git branch -D weblate-main`
6. Fetch new changes from the `weblate` remote
    - `git fetch weblate`
7. Create a new branch starting from `weblate/main`, named `weblate-main`, and switch to it
    - `git checkout -b weblate-main weblate/main`
8. If you run `git log --oneline --graph` you should see a Weblate commit on top, and then all of the commits currently on the `main` branch:
    ```md
    * cmt12hash (HEAD -> weblate-main, weblate/main) Translated using Weblate (...)
    * cmt89hash (origin/main, main) Commit message ...
    ```
9. Switch back to the `main` branch
    - `git checkout main`
10. Merge `weblate-main` into `main`:
    - `git merge weblate-main`
11. If you are an admin of the dicio-android repo, just push the changes to the remote `main`
    - `git push origin main` [^repo]
    - If you are not an admin, create a pull request normally and ask a maintainer to merge it

Note that we had to do this process on dicio-android's `main` branch because:
- Weblate's components are connected to dicio-android's `main` branch, and will update changes from there
- Weblate's git repo is not writable, so there is no way to push commits there manually

### Update Weblate <small>[^weblate]</small>

12. Go back to [Weblate's Maintenance tab](https://hosted.weblate.org/projects/dicio-android/#repository)
13. Press the *Update* button to update Weblate with the commit you just pushed on dicio-android's `main` branch
14. **Press the *Unlock*** button to allow translators to translate the changelog and possibly other components (**do not forget this step!**)

## Finally releasing

### Update version numbers

- Edit the [`app/build.gradle.kts`](https://github.com/DicioTeam/dicio-android/blob/main/app/build.gradle.kts) file to bump the release
    - Set `versionCode` to `NEW_VERSION_CODE`, i.e. increment the value by 1 as described in [Version name and conventions](#version-name-and-conventions)
    - Set `versionName` to `"X.X.X"`
- Commit the version bump (try to stick to the provided commit message template)
    - `git add app/build.gradle.kts`
    - `git commit -m "Release vX.X.X (NEW_VERSION_CODE)"`
- If you are an admin of the dicio-android repo, just push the changes to the remote `main`
    - `git push origin main` [^repo]
    - If you are not an admin, create a pull request normally and ask a maintainer to merge it

### Creating a GitHub release <small>[^repo]</small>

- Run the ["Build and create release"](https://github.com/DicioTeam/dicio-android/actions/workflows/build_and_create_release.yml) workflow using the "Run workflow" button (make sure to select `main` as the branch). This will create a draft release, which you can find [here](https://github.com/DicioTeam/dicio-android/releases).
- Download the signed APK from the draft release, and make sure it installs properly on your device as an update.
- Insert the GitHub changelog you [prepared earlier](#github-changelog) into the draft release body.
- Make sure `vX.X.X` is set as the tag name
- Make sure `vX.X.X` is set as the release title
- Make sure `main` is set as the "Target:" branch
- Publish the release

### Creating an F-Droid release

<sup>The store listing is [here](https://f-droid.org/packages/org.stypox.dicio/).</sup>  
No manual steps are needed to push the update to F-Droid, since [the metadata](https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/org.stypox.dicio.yml) is setup to update automatically after a new `git` tag is created (which happens when pushing the "Release" button of a GitHub release).  
One thing to look out for, however, is whether the build is reproducible. See [#308](https://github.com/DicioTeam/dicio-android/issues/308).

### Creating a Play Store release <small>[^repo]</small>

<sup>The store listing is [here](https://play.google.com/store/apps/details?id=org.stypox.dicio).</sup>  
*Note: the Play Store release is lower priority, so even if something blocks it (e.g. some new Google requirement), it's fine.*

- Run the ["Publish to Play Store"](https://github.com/DicioTeam/dicio-android/actions/workflows/publish_to_play_store.yml) workflow using the "Run workflow" button (make sure to select `main` as the branch).[^repo]
- Ask @Stypox to complete the release on Google Play Console
