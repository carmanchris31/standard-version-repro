## Configuration

In this setup, standard-version is responsible only for managing tags. Neither `package.json` nor `CHANGELOG.md` are used in order to avoid merge conflicts.

## Example Scenario / Description of problem

1) I commit a feature and tag it as an alpha release (`v1.1.0-alpha.0`).
2) I find an issue with the alpha and fix it, tagging a new version (`v1.1.0-alpha.1`).
3) This version is more stable so I upgrade it to a beta version (`v1.1.0-beta.0`)

    I now have a git log that looks like this:
    ```
    * 21b6793 (HEAD -> master, tag: v1.1.0-beta.0, tag: v1.1.0-alpha.1) fix: Some fix
    * 3cb3d59 (tag: v1.1.0-alpha.0) feat: Some feature
    * 776f5c6 (tag: v1.0.0) chore: Initial commit
    ```


4) Now I want to commit some more changes and tag it as an alpha again, but `standard-version` gives the following error:

    ```sh
    > npx standard-version --prerelease=alpha

    ✔ tagging release v1.1.0-alpha.0
    fatal: tag 'v1.1.0-alpha.0' already exists

    Command failed: git tag -a v1.1.0-alpha.0 -m chore(release): 1.1.0-alpha.0
    fatal: tag 'v1.1.0-alpha.0' already exists
    ```

    Even though there are already two alphas with the same version in the history, they are occluded by the beta tag:
    ```
    v1.1.0-beta.0
    v1.1.0-alpha.1
    v1.1.0-alpha.0
    v1.0.0
    ```

    Thus, standard version reset to `alpha.0` again which clashes with the existing tag.

## Possible solution

When doing a prerelease, `standard-version` should continue searching beyond the first tag in the sorted list for any tags that match the highest version plus the supplied preid:

1) Check highest tag

    a) If tag does not have a preid (e.g. `v1.1.0`, use current behavior to get recommended bump, e.g. `v1.1.1-alpha.0`)

    b) If tag has a preid, e.g. `v1.1.0-beta.0`, parse latest version `v1.1.0` and combine with supplied preid `alpha` to search for tags beginning with `v1.1.0-alpha.`

2) Iterate over each tag in the history until one of two conditions is met:

    a) tag starts with `v1.1.0-alpha.`, use as latest version. For the alpha scenario, we'd stop at `v1.1.0-alpha.1` and know the next version should be `v1.1.0-alpha.2`

    b) tag version is less than the search criterion, stop. In this case, if there were no alpha tags, we'd stop at `v1.0.0` since it's less than `v1.1.0` and the next version would be `v1.1.0-alpha.0`
