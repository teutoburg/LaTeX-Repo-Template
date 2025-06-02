# LaTeX-Repo-Template
Template repository for hosting a LaTeX document with diff and build automation.

## What this is
Basically a collection of GitHub Action workflows to host a LaTeX document (assumed to be named `main.tex` and stored in root) and build LaTeX-Diff PDFs for pull requests, as well as build the full PDF for every release.

## How it works
The actions for PRs are controlled via labels.
There are three different functions based on a PR's labels.

### LaTeX-Diff for PRs
For any PR with the `content` label, a diff PDF is built and stored as an action artifact.
The diff is based on the PR's HEAD and BASE.
A comment is added to the PR linking to the workflow run which has the artifact link (see also Limitations below).
For every new push to the PR, the diff is re-run, as well as for any potential base change.

### Before-After PDFs for PRs
For any PR with the `format` label, a "before" (BASE) and "after" (HEAD) version of the document is built and stored as an action artifact.
Similar to the previous function, but instead of the diff, two full versions of the document are built.
This is useful for changes to the formatting, layout, or otherwise, where no diff-able changes in content occur (hence the label name).
It is generally recommended to do content and format/layout changes in separate PRs, but it is also possible to apply both labels and get three PDFs.

### Release from PR
For any PR with the `release` label, a release will be created once the PR is merged.
A new tag will be created based on the current date (CalVer scheme) in the YYYY-MM-DD format, and a release named "Version YYYY-MM-DD" based on it.
The repo also includes a `release.yml` file, which specifies the headers and groups for the auto-generated release notes.
By default, this groups all PRs labelled `content` into a section "Content changes" and everything else into a section "Other changes", _except_ PRs labelled `no release notes` (spaces not dashes), which are ignored.
Feel free to modify this list to your own needs.

The full LaTeX document will be built from the merge commit, named "${REPO_NAME}_version_${TAG_NAME}.pdf", and uploaded to the release as an attachment.
The content of the commonly used `\date{}` field is replaced by the tag name for the build (see also Limitations below), enabling the version date to be shown in the PDF.

### Build for manual release
When a manual release is created, the same workflow as above is triggered with the exception that the PDF is uploaded to the existing (just created) release instead of creating a new one.
This can be useful when making multiple changes through multiple PRs and only releasing after all are done.
For a single change (using one or more commits) in a single PR, the labelled solution is recommended.

## Known limitations
This is currently a one-man project tailored to some specific needs, thus a few limitations were accepted.

### Usage of `pull_request_target`
This workflow action trigger is known to have some security limitations.
Please inform yourself on the exact nature of those limitations and if they are relevant in your case, before using this action.
I **do not** take _any_ responsibility whatsoever for what you do with the actions from this repository.
Using it in security-critical projects is discouraged.

Also, by its nature, the workflows need `write` permission, keep that in mind for any security concerns.

### File name and location
The LaTeX document to be built is assumed to be named `main.tex` and stored in the root of the repository.
If that doesn't work for you, feel free to find the relevant lines and change them in your copy.

The same applied for the names of the three labels, `content`, `format` and `release`.

### When to label
The `content` and `format` labels are best applied to the PR _before it's created_, i.e. in the editor window you get when creating a new PR.
This ensures that they are triggered immediately when the PR is created.
The `release` label is only taken into account once the PR is _merged_.

### Versioning
Because this uses a YYYY-MM-DD based calver scheme to label the releases, it is not possible to create more than one release per day.
Running the "release from PR" action on a day for which a release already exists will result in overwriting the uploaded PDF, but no changes to the release notes.

### Usage of the `\date{}` field
The build workflow uses a regex to replace the content of the `\date{}` command in `main.tex`.
This is only done for the build and _not_ committed back into the repo.
If you're using that command for anything else, it will get overwritten.
If you store your date information in another way in your document, feel free to change the appropriate code in your copy.
If you're using e.g. the `datetime2` package to format the output of `\date{}` in your document and want to build it locally, it is recommended to use a placeholder date, e.g. 0000-00-00 to avoid errors.

### Linking to the PDFs in PRs
I found no easy way to link the PR-PDFs (diff or before-after) directly in the added comment (and have it robust enough for re-build on further pushes).
Thus the link that is created only points to the workflow run, where the zip file containing the PDF(s) can be found under artifacts.
The only _real_ way to solve this would be to host the PDFs externally and link there, but this is currently well outside the scope of this project.
