# Private Fork Experiment

This is an experiment in manually creating a fork of a project using git.

The idea was to have one public GitHub repository hold a generic template or boilerplate code base (say, a program or website without any branding or content), then make a private fork of that repo which contains customizations like logos, artwork, and content. However, GitHub doesn't support making forks of your own repos (currently), and I wanted to know if it was possible to fork a public project privately.

Short answer: yes.

**Note:**  
I'm not using GitHub Templates because you can't (as far as I'm aware) make forks of them. When you use a template the repo is copied as-is and the two remain wholly separate with improvements unshareable.

# My Experiment

## Git Default with Readme File

I tried to follow [this blog post's](http://creativenotice.com/2014/09/how-to-manually-fork-a-github-repo/) instructions:

> 1. Create a new empty repo on Github.
> 2. Clone the new empty repo down to my local dev machine.
> 3. Add the original repo as the upstream of the new local repo.
> 4. Now just fetch and merge the upstream master to new repo master.

And so ran the following commands:

```
$ git clone git@github.com:roboticforest/generic-public-test-repo.git
$ git remote add upstream git@github.com:roboticforest/specialized-private-test-repo.git
$ git fetch upstream main
$ git merge upstream/main main
```

When I created a new private repo called `specialized-private-test-repo` via GitHub's UI I initialized it with a dummy README.md file. That created a merge conflict needing to be resolved when trying to read in this repo's history. Instead of trying to fix that I simply deleted and remade that repo with no content of any kind.

## Merge to an Empty Repo

I ran into a problem when running those commands again on the empty repo. At the final merge step I got the following error:  
`merge: main - not something we can merge`

After a little investigating I found that the default primary work branch for the empty `specialized-private-test-repo` was named `master` instead of `main` and when I tried the merge again I got the same error.

Or, at least I *think* the default branch is named `master`. When I try `git branch` or `git branch --list` to see where I'm at nothing prints. `git status` gives the usual `On branch master` but running `git branch -m master main` to rename `master` prints out:

```
error: refname refs/heads/master not found
fatal: Branch rename failed
```

So, am I on a default branch of some kind or not? I've never tried to mess with a truly empty repo before.

## Manually Creating a Branch

I next tried `git checkout -b main` and again tried to merge the manually created `upstream/main` and got `not something we can merge` again. After a quick glance through the git docs I tried `git branch main --set-upstream-to=upstream/main` and got `fatal: branch 'main' does not exist`.

## Create a Bare Clone, Then Push to Empty Repo

This is what finally worked. Doing a `merge` to the empty repo doesn't work, but since the repo *is empty* there's no harm in asking git to just overwrite what metadata is there with a `push` command.

I used `git clone --bare` to create a copy of just the git history of the public repo and pushed that straight up to the empty private repo. With that done I was able to make a normal working clone of the private repo and added unique files to both. I next used `git remote add upstream` to point the private repo to the public repo it was made from.

This setup works nicely. I was able to `git fetch` from the public repo then merge/rebase the new content into the private repo. And, within the private repo I can work normally, doing push/pull operations without needing to specify which remote I'm using.

To retrieve improvements from the private fork repo that I would like to see in the original public repo I was able to use `git cherry-pick` to pull commits into a new branch, then push those commits to the public repo like normal. I had read that GitHub wouldn't recognize such pushes as being from a fork since the upstream connection is done on my local machine. That was false. I ran `git push upstream` from a `for-upstream` branch and when I went to GitHub I was presented with messages about opening a Pull Request.

# The Full Procedure

## 1. Create a public repo on GitHub.

This is your public template or boilerplate project. Use GitHub's web interface for this.

## 2. Create an *empty* private repo on GitHub.

This will be your private fork. Again, use GitHub's web interface. **Make sure you do not initialize it with any files (readme, .gitignore, license, and so on).**

The rest of the instructions will happen on your local machine.

## 3. Make a bare clone of the public GitHub repo.

This will only copy the git history and metadata. It won't create the usual working directory.

`git clone --bare git@github.com:username/public-repo.git`

## 4. Push the bare clone up to the private GitHub repo.

```
cd public-repo.git
git push --mirror git@github.com:username/private-repo.git
```

After this, you can delete the bare clone.

```
cd ..
rm -rf public-repo.git
```

## 5. Create a normal clone of the private repo.

```
git clone git@github.com:username/private-repo.git
cd private-repo
```

## 6. Lastly, add the public repo as an upstream remote.

```
git remote add upstream git@github.com:username/public-repo.git
git fetch upstream
```

Running `git remote show` should list:

```
origin
upstream
```

## 7. Do your work.

Now, you can freely work within the private fork as you normally would. Make changes, commit, and push to the private repo on GitHub. To pull in updates from the public repo, you can use `git fetch` and merge or rebase them as needed.

Like so:

```bash
git fetch upstream main
git merge upstream/main
```

## 8. Optional: Pushing changes back from private to public.

If you ever want to contribute changes from your private fork back to the public repo, you can push changes to a new branch and create a pull request.

```bash
git checkout -b for-upstream
git cherry-pick <commit-hash>  # Select specific commits if needed
git push upstream for-upstream
```

Then, navigate to the public repository on GitHub and handle the Pull Request.
