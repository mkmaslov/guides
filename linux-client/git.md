# Guide to git

[git](https://git-scm.com/) is the most popular distributed version control system. It is commonly used to synchronize concurrent changes to the same document, while keeping the history of these changes.

## Contents

- [Installing git, creating SSH and GPG keys](#installing-git-creating-ssh-and-gpg-keys)
- [Cloning a repository and configuring git client](#cloning-a-repository-and-configuring-git-client)
- [Committing to a repository](#committing-to-a-repository)
- [Working with branches](#working-with-branches)
- [Applying new `.gitignore` rules](#applying-new-gitignore-rules)
- [Undoing a commit](#undoing-a-commit)


## Installing git, creating SSH and GPG keys

Install the CLI version of `git` using guides for [Windows](https://git-scm.com/download/win), [macOS](https://git-scm.com/download/mac) or [Linux](https://git-scm.com/download/linux).

Local **git client** is communicating with a remote **git instance** (on a server) via `ssh`. This connection requires an SSH key that can be created using [this guide](https://github.com/mkmaslov/guides/blob/main/linux-client/ssh-client.md). Changes to files (commits) are cryptographically signed using either an SSH key (same or different) or a GPG key, which can be created using [this guide](https://github.com/mkmaslov/guides/blob/main/linux-client/gpg.md). **SSH and GPG keys should be added as authentication and signing keys in the web interface of your git instance.**

Alternatively, a git client can connect to a git instance via `https`. **This method is discouraged.** It requires creating a personal access token in the web interface of your git instance. You should **never** use `git` with you git instance password. While it sometimes works, this is **not** how the system was intended to be used. Make sure to save the generated access token somewhere, like [a password manager](https://bitwarden.com/), as **it will only be displayed once**.


## Cloning a repository and configuring git client

Clone the repository you want to work with using:
```
git clone <GIT-HOST>:<GIT-USERNAME>/<REPOSITORY-NAME>.git
```
where `<GIT-HOST>` is a label from your `~/.ssh/config` or `<GIT-HOST> = git@<GIT-INSTANCE>`, where `<GIT-INSTANCE>` is the URL of a website that is hosting your **git instance**, e.g., [github.com](https://github.com); `<GIT-USERNAME>` is your username on the git instance and `<REPOSITORY-NAME>` is the name of a repository you want to clone.

Navigate to the newly created local directory `./<REPOSITORY-NAME>`. Set up your **local** git client settings (for this repository only) using `--local` flag. Avoid using global settings, i.e., `--global` flag, to prevent mixing up signing keys and credentials for different git instances.

For [`github.com`](https://github.com), set up the following credentials:
```
git config --local user.name "Name Surname"
git config --local user.email "<EMAIL-ALIAS>@users.noreply.github.com"
```
Where you can view/configure the email alias (`<EMAIL-ALIAS>@users.noreply.github.com`) in [GitHub → Settings → Emails](https://github.com/settings/emails).

For repositories at [`git.ista.ac.at`](https://git.ista.ac.at), use the following credentials:
```
git config --local user.name "Name Surname"
git config --local user.email "name.surname@ist.ac.at"
```
Note that the email should be in the `ist.ac.at` (not the `ista.ac.at`) domain. You can find the email you should use in [ISTA GitLab → User settings → Emails](https://git.ista.ac.at/-/profile/emails). Avoid using other email addresses or nicknames, as it may cause the commit author to be displayed incorrectly in the web interface.

To cryptographically sign commits, enable signing:
```
git config --local commit.gpgsign true
```

For signing commits using an SSH key:
```
git config --local gpg.format ssh
git config --local user.signingkey ~/.ssh/keys/<SSH-KEY-NAME>.pub
```
The SSH key for signing can be the same or differ from the SSH authentication key. In any case, the SSH key(s) (contents of `~/.ssh/keys/<SSH-KEY-NAME>.pub` from [this guide](https://github.com/mkmaslov/guides/blob/main/linux/ssh.md)) should be uploaded to the web interface of your git instance.

For signing using a GPG key:
```
git config --local user.signingkey <SUBKEY-ID>
```
Then add the GPG key (contents of `public_key.asc` from [this guide](https://github.com/mkmaslov/guides/blob/main/linux/gpg.md)) as a signing key in the web interface of your git instance.

To list local/global git client settings:
```
git config --local --list
git config --global --list
``` 
To unset a setting (you can add `--local`/`--global` flags):
```
git config --unset <SETTING-NAME>
```


## Committing to a repository

**Every time** you want to change the code in your local repository, please make sure that the local repository has the latest updates that may have been introduced by your collaborators in the meantime. To update the code in local repository, use:
```
git pull
```

After introducing changes to files in the local repository, you need to communicate them to `git`, or "stage" them. If you added new files, use:
```
git add <FILENAME>          # to add a single file
git add -A                  # to add all new files
```
to add them to the git file tree of your local repository. Afterwards, **commit all changes** using:
```
git commit -am "<GIT-MESSAGE>"
```
where `-a` requests committing all changes, and `-m` specifies the commit message. `<GIT-MESSAGE>` should contain **concise, yet specific** description of changes: write `"fixed Python routine for converting audio files"` instead of a plain `"update"`.

When ready, upload changes to the git instance using:
```
git push origin main
```


## Working with branches

By default, a `git` repository has only one branch, called `main`. When collaborating with other people on the same project, consider creating your personal branch, called `<BRANCH-NAME>`, for hosting temporary changes, and only merge the final result to the `main` branch.

Create and switch to a new branch: 
```
git branch <BRANCH-NAME>
git checkout <BRANCH-NAME>
```
Perform necessary changes and **commit** them. Then, switch back to the `main` branch: `git checkout main`. In the end, merge `<BRANCH-NAME>` into `main` branch: 
```
git merge <BRANCH-NAME>
``` 


## Applying new `.gitignore` rules

After modifying the `.gitignore` file in an existing local repository, ignore rules won't be applied retroactively to files that are already committed. To apply new `.gitignore` rules to the entire local repository, clean up the cached file tree and create the new one:
```
git rm -r --cached .
git add .
```
The next commit will remove files added to `.gitignore`:
```
git commit -am "changed .gitignore rules"
```

## Undoing a commit

To find the undesired commit, one can display full commit history using: 
```console
$ git log

commit <COMMIT-ID-C> (HEAD -> main, origin/main, origin/HEAD)
Author: Name Surname <EMAIL>
Date:   <DATE>

    <COMMIT-MESSAGE-C>

commit <COMMIT-ID-B>              <-- this is the undesired commit
Author: Name Surname <EMAIL>
Date:   <DATE>

    <COMMIT-MESSAGE-B>

commit <COMMIT-ID-A>
Author: Name Surname <EMAIL>
Date:   <DATE>

    <COMMIT-MESSAGE-A>

...
```
which shows both the commit messages and the corresponding `<COMMIT-ID>`'s. 

Let's assume that the undesired commit is `<COMMIT-ID-B>`. Start `rebase` over the last `N` commits that include `<COMMIT-ID-B>`. This will open a text editor:
```console
$ git rebase -i HEAD~N

...
pick <COMMIT-ID-REBASE-A> # <COMMIT-MESSAGE-A>
pick <COMMIT-ID-REBASE-B> # <COMMIT-MESSAGE-B>
pick <COMMIT-ID-REBASE-C> # <COMMIT-MESSAGE-C>
```
Note that `<COMMIT-ID-REBASE-*>` are different from `<COMMIT-ID-*>` displayed by `git log`, but the `<COMMIT-MESSAGE-*>` is the same.

#### To remove the commit and REMOVE all of its changes to files

Replace `pick` with `drop` and save the file (`[ESC] + :wq` in `vim`). The `rebase` command will go through all commits marked with `pick`, resulting in:
- If latter commits do not incorporate changes introduced in the undesired commit (edit the same files), the undesired commit and its changes are gone.
- Otherwise, there is a merge conflict, since commits after the dropped commit include changes that are not present in any of the previous commits. Resolve the conflict:
    * Edit problematic files. Run `git status` to figure out which files cause the merge conflict. Remove the `<<<<<<<`, `=======`, `>>>>>>>` markers from problematic files.
    * Run `git add -A` to mark merge conflicts as resolved.
    * Run `git rebase --continue` to finish the `rebase` process.

#### To remove the commit, but KEEP all of its changes to files

Replace `pick` with `edit` and save the file (`[ESC] + :wq` in `vim`). The `rebase` command will go through all commits marked with `pick`, stopping at the undesired commit marked with `edit`:
- Remove the undesired commit, preserving changes to files: `git reset HEAD^`.
- (Optional) Recommit changes as a new commit.
- Run `git rebase --continue` to finish the `rebase` process.

In order to remove or edit commits in the remote repository, perform the operations locally, then force-push the changes to the remote repository:
```
git push origin +main
```