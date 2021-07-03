# generic-template-test-repo

I'd like to take my website repo and make it generic (no branding or content), and then make it public, then finally make a private fork of it which contains the site specific logos, artwork, and content.

The important part: I want the fork to maintain a link to the generic website template it is built from instead of being an independent copy. That way, if the template changes, or if the specialized site makes an improvement, those changes can be reflected in the other repo.

I think this is possible through using project forks, however GitHub doesn't support making forks of your own repos (currently), and I need the fork of the public template to be private. Let's get this figured out.

# Progress Notes

## Git Default with Readme File

I tried to follow [this blog post's](http://creativenotice.com/2014/09/how-to-manually-fork-a-github-repo/) instructions:

> 1. Create a new empty repo on Github.
> 2. Clone the new empty repo down to my local dev machine.
> 3. Add the original repo as the upstream of the new local repo.
> 4. Now just fetch and merge the upstream master to new repo master.

And so ran the following commands:

```
$ git clone git@github.com:roboticforest/generic-template-test-repo.git
$ git remote add upstream git@github.com:roboticforest/specialized-template-test-repo.git
$ git fetch upstream main
$ git merge upstream/main main
```

When I created a new private repo called `specialized-template-test-repo` via GitHub's UI I initialized it with a dummy README.md file. That created a merge conflict needing to be resolved when trying to read in this repo's history. Instead of trying to fix that I simply deleted and remade that repo with no content of any kind.

## An Empty Repo

I ran into an unexpected problem. At the final merge step I got the following error:  
`merge: main - not something we can merge`

After a little investigating I found that the default primary work branch for the empty `specialized-template-test-repo` was named `master` instead of `main` and when I tried the merge again I got the same mysterious error.

Or, at least I *think* the default branch is named `master`. When I try `git branch` or `git branch --list` to see where I'm at nothing prints. `git status` gives the usual `On branch master` but running `git branch -m master main` to rename `master` prints out:

```
error: refname refs/heads/master not found
fatal: Branch rename failed
```

So, am I on a default branch of some kind or not? I've never tried to mess with a truly empty repo before.

## Manually Creating a Branch

I next tried `git checkout -b main` and again tried to merge the manually created `upstream/main` and got `not something we can merge` again. After a quick glance through the git docs I tried `git branch main --set-upstream-to=upstream/main` and got `fatal: branch 'main' does not exist`.



