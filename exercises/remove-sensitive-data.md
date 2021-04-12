# Removing sensitive data

## Before you proceed

A history re-write is an irreversible and destructive procedure. It should only be explored as a last resort for your shared branches. There are numerous ways to recover from introducing mistakes in a commit. You're highly advised to go through the guides below prior to attempting any of the methods in this exercise:

- [Fixing Commit Mistakes](https://githubtraining.github.io/training-manual/#/19_fixing_commit_mistakes)
- [Rewriting History with Git Reset](https://githubtraining.github.io/training-manual/#/20_rewriting_history_git_reset)
- [Cherry Picking](https://githubtraining.github.io/training-manual/#/21_git_cherry_pick)

## 0. Setting up the repository

First create a branch, and then add a file named `patient-list-example.csv` to the branch you created. Add the following lines as the contents of the csv file.

```
FIRST_NAME,LAST_NAME,EMAIL,PHONE,ADDRESS,DOCTOR
Alexander,Leonard,aleonard@supermail.com,555-867-5309,64 Young St.,Dr. Abby Aina
Jon,Allen,jonallen@supermail.com,555-867-5309,503C N. Mayfield Dr.,Dr. Jaelyn Schoener
Kay,Mcdonald,kmac@supermail.com,555-867-5309,826 W. Old York Street,Dr. Damian Cardno
Fernando,Garza,fgarza12@supermail.com,555-867-5309,58 East Orange St.,Dr. Jaelyn Schoener
Fred,Townsend,fred1@supermail.com,555-867-5309,15 Mill Pond Circle,Dr. Adaline Ardinger
Meredith,Sims,msims05@supermail.com,555-867-5309,759 Adams St.,Dr. Frederick Chaffee
Dorothy,Blair,dorothy.blair@supermail.com,555-867-5309,132 Boston Ave.,Dr. Julia Greenstein
Rodolfo,Gross,rodolfo@supermail.com,555-867-5309,8 Oakwood St.,Dr. Lloyd Meader
Louise,Berry,lberry@supermail.com,555-867-5309,291 Summerhouse Rd.,Dr. Zion Molz
```

Commit that file and create a Pull Request to merge the branch into the default branch.
Merge the Pull Request.

## 1. Remove a file containing sensitive information from all commits

### 1.1 Identifying where sensitive information was introduced

In this scenario, a feature branch was created that contained sensitive information. That feature branch was then merged into our default branch.

First we can examine the history to see when the sensitive information was introduced. From the command prompt run:
`git log --oneline`

Identify where the sensitive information was introduced.

### 1.2 Clean up

The main objective is to remove the file `patient-list-example.csv` from all commits and branches because it contains sensitive information.

#### 1.2.1 History rewrite tools / options

1. [git-filter-branch](https://git-scm.com/docs/git-filter-branch)
2. [git-filter-repo](https://github.com/newren/git-filter-repo)
3. [BFG repo cleaner](https://rtyley.github.io/bfg-repo-cleaner/)

The author of `git-filter-repo` has drafted a great comparison on the different tools mentioned above and the caveats of each. It is advised that you [read it](https://github.com/newren/git-filter-repo#why-filter-repo-instead-of-other-alternatives) for a better understanding of these tools.

#### 1.2.2 Execution

For the purposes of this example we will be using `git-filter-repo`.

1. Make sure you have a **fresh clone** of the repository you want to work with. It's also not a bad idea to **lock** the repository in GitHub to avoid anyone  writing new changes to it while you are removing the sensitive data.

    You can do so from the Branch Protection settings by enabling the "Restrict who can push to matching branches" settings. Set either an empty value or add the team or person that is doing the history rewrite. Note that `repository admins` will still be able to push changes with this method.

    ```sh
    git clone git@github.com:<ORG_NAME>/<REPO_NAME>.git mirror_<REPO_NAME>
    ```

2. Since we know the file we would like to purge (`patient-list-example.csv`), try executing the following in your freshly cloned repository:

    ```sh-session
    # Make sure you are in the repository's root path
    $ git-filter-repo --path patient-list-example.csv --invert-paths --use-base-name
    ```


    Definitions:  
      **`--path <dir_or_file>`** - Exact paths (files or directories) to include in filtered history. Multiple --path options can be specified to get a union of paths.  
      **`--invert-paths`** - Invert the selection of files from the specified --path-{match,glob,regex} options below, i.e. only select files matching none of those options.  
      **`--use-base-name`** - Match on file base name instead of full path from the top of the repo. Incompatible with --path-rename, and incompatible with matching against directory names.  

3. Verify that the file has been removed successfully and the history was rewritten by running this command in terminal to see the file structure: `tree`.

4. Run `git log --oneline --graph --all --decorate --abbrev-commit` to examine the git history.

#### 1.2.3 Pushing the new history upstream

> When you rewrite history, the old and new histories are no longer compatible; if you push this history somewhere for others to view, it will look as though you’ve done a rebase of all branches and tags. Make sure you are familiar with the "RECOVERING FROM UPSTREAM REBASE" section of git-rebase(1) (and in particular, ["The hard case"](https://git-scm.com/docs/git-rebase#_the_hard_case)) before proceeding

The above is a snippet from the maintainers of `git-filter-repo`. Make sure you **understand** fully all the warnings mentioned and agree with the team owning the repository on a strategy forward.

You can find [the guide here](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#DISCUSSION).

**TLDR;**

The cleanest approach would be to create a fresh repository in GitHub and just push the local repository (with the re-written history) to it.

**Not recommended:** if you insist on using the same GitHub repository and accept the responsibility of that despite all the warnings above:

1. `git-filter-repo` deletes the "origin" remote to help avoid people accidentally re-pushing to the same repository, so you’ll need to remind git what origin’s url was.

    ```sh-session
    $ git remote add origin git@github.com:<ORG_NAME>/<REPO_NAME>.git

    # Example output
    $ git remote -v
    origin git@github.com:<ORG_NAME>/<REPO_NAME>.git (fetch)
    origin git@github.com:<ORG_NAME>/<REPO_NAME>.git (push)
    ```

2. You will have to force update the repository upstream via:

    ```sh-session
    $ git push --force origin --all
    ```

3. At this point, everyone and every system that has a copy of this repository **needs to delete their local copy and clone it again!**

**Note:** As pointed out [here](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/removing-sensitive-data-from-a-repository), once you push your changes to GitHub, you still need to permanently remove cached views and references to the sensitive data in pull requests by contacting GitHub Support or GitHub Premium Support.

---

## 2. Replacing sensitive data in a file instead of deleting the file

In this scenario the sensitive data is found in a file and needs to be removed without deleting the file. This could happen when sensitive data was inadvertently committed, and later, an audit reveals its presence and necessitates a cleanup.

## 2.1 Repository Set Up
Create a new branch in the repository.
Find the `authn-service/.env` file add the following secret to the end of the file
``` API_KEY=12345ABCD67890 ```
Create a Pull Request and merge the changes to the default branch.

### 2.1.2 Identifying a committed secret

Find the `.env` file. Those files are commonly used to contain configuration variables and sometimes secrets. These files should not be committed upstream, but in this example it has been committed. Obviously `API_KEY` has been exposed now and we need to scrap it.

When a credential has been exposed, remember to **"rotate and remove".** First rotate the exposed credential by generating a new credential and updating your application. Usually this is enough, but you may also want to remove the credential from the repository history.

### 2.2 Clean up

1. Make sure you have a **fresh clone** of the repository you want to work with. It's also not a bad idea to **lock** the repository in GitHub to avoid anyone writing new changes to it.

    You can do so from the Branch Protection settings; by enabling the "Restrict who can push to matching branches" settings. Set either an empty value or add the team or person that is doing the history rewrite.

    ```sh-session
          $ git clone --mirror git@github.com:<OWNER>/<REPO_NAME>.git mirror_history_rewrite

          Cloning into bare repository 'mirror_history_rewrite'...
          remote: Enumerating objects: 34, done.
          remote: Counting objects: 100% (34/34), done.
          remote: Compressing objects: 100% (18/18), done.
          remote: Total 34 (delta 7), reused 31 (delta 5), pack-reused 0
          Receiving objects: 100% (34/34), done.
          Resolving deltas: 100% (7/7), done.
    ```
    ***Note:** This is a bare repo, which means your normal files won't be visible, but it is a full copy of the Git database of your repository, and at this point you should make a backup of it to ensure you don't lose anything.*

1. Verify that the sensitive data exists in the repo

    ```sh-session
    $ git rev-list --all | xargs  git grep "12345ABCD67890"

    <COMMIT-SHA>:.env:API_KEY=12345ABCD67890
    ```

    We don't have to use a general regular expression in this case. The API key is unique enough to be used as a key

1. Create a file that contains a list of expressions to be used for finding sensitive data and their replacement. In our case, the file will look like:

    ```sh-session
    $ cat cleanup.txt

    12345ABCD67890
    ```

    The `cleanup.txt` file contains only the API key. If no replacement is provided, `git-filter-repo` will search for this value and remove it. More complex examples can be found [here](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html).

1. Run `git-filter-repo` to replace the occurrences of the string from history

    ```sh-session
    $ git-filter-repo --replace-text cleanup.txt --replace-refs delete-no-add

    Parsed 13 commits
    New history written in 0.13 seconds; now repacking/cleaning...
    Repacking your repo and cleaning out old unneeded objects
    Enumerating objects: 32, done.
    Counting objects: 100% (32/32), done.
    Delta compression using up to 16 threads
    Compressing objects: 100% (16/16), done.
    Writing objects: 100% (32/32), done.
    Building bitmaps: 100% (11/11), done.
    Total 32 (delta 7), reused 22 (delta 5), pack-reused 0
    Completely finished after 0.28 seconds.
    ```

    Replace refs are used to rewrite parents. In this case, we are deleting them so the only new commit hashes would be present.

1. At this point your repo must be clean without the sensitive information. Check that is the case by running the following command:

    ```sh
    git rev-list --all | xargs git grep "12345ABCD67890"
    ```

1. We could strip out any unwanted data by doing a `git gc`  
    Learn more about `git gc` [here](https://git-scm.com/docs/git-gc).

    ```sh-session
    $ git reflog expire --expire=now --all && git gc --prune=now --aggressive

    Enumerating objects: 32, done.
    Counting objects: 100% (32/32), done.
    Delta compression using up to 16 threads
    Compressing objects: 100% (21/21), done.
    Writing objects: 100% (32/32), done.
    Reusing bitmaps: 9, done.
    Building bitmaps: 100% (11/11), done.
    Total 32 (delta 7), reused 25 (delta 0), pack-reused 0
    ```

1. Push the changes to the remote repository

    ```sh-session
    $ git push --mirror

    Enumerating objects: 2, done.
    Counting objects: 100% (2/2), done.
    Writing objects: 100% (13/13), 1.59 KiB | 1.59 MiB/s, done.
    Total 13 (delta 2), reused 2 (delta 2), pack-reused 11
    remote: Resolving deltas: 100% (2/2), completed with 1 local object.
    To github.com:<ORG_NAME>/<REPO_NAME>.git
    + 5114bc0...117a0da trunk -> trunk (forced update)
    + 5c97416...84e4bfa 0.1.0-beta -> 0.1.0-beta (forced update)
    ```

**Note:** As pointed out [here](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/removing-sensitive-data-from-a-repository), once you push your changes to GitHub, you still need to permanently remove cached views and references to the sensitive data in pull requests by contacting GitHub Support or GitHub Premium Support.

---
💡**Now that we're familiar with removing sensitive data, let's learn about Dependabot Alerts. [Click here](dependabot.md) to move to the next section.** 💡
