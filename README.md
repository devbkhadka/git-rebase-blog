## Everything you need to know about git rebase

#### How rebase works
- Lets say we have commits c1, c2 and c3 on branch b1
- When we do `git rebase <ref>` with b1 as current branch,
	- git will checkout `<ref>` where `<ref>` can be commit hash, branch name, tag anything that points to a git commit
	- Then git will replay (apply) each commits c1, c2 and c3 one at a time. This will be same as you started working by checking out `<ref>` and redoing all your changes.
	- If there are conflicts when applying the commits, it will stop and allow you to resolve the conflict and continue using `git rebase --continue`
	- Finally git will reset current b1 to points to new HEAD
- We can use interactive rebase to pick or drop commits, reorder them, combine them and more. We'll see examples below.

#### When can we use rebase
- When you have feature branch which may takes few days before you can create PR to merge. In this case you can rebase your branch against the main branch daily and get new codes and minimize conflicts.
- Rebasing before merging your branch to main branch makes history cleaner
- When working locally you can do micro commits and before creating PR you can rebase and reorganize your commits into atomic changes.
- In some cases you may need to maintain temporary branch for testing features from multiple team members, lets say UAT branch. you can use interactive rebase to bring multiple changes from another branch to UAT branch.

#### What are the risks of using rebase and how to minimize them
The major risk of using rebase is it modifies the history and we need to do **force push** to upstream. So rule of thumb is **don't use rebase in a shared branch**. Some times there could be situation we want to use rebase when 2 people working on same feature. Lets say Backend and Frontend engineers working on same feature, In such case we can minimize the risk by using `--force-with-lease` when pushing to upstream. This prevents you from replacing other peoples work but for `--force-with-lease` to work as expected you need to keep few things in mind.

**How `force-with-lease` works**
- Git tracks upstream branches with remote branches, for example if you fetch branch `branch1` from remote named `origin` there will be a remote branch named `origin/branch1`
- Now lets say we have some changes on `branch1` locally and also rebased changing history. At the same time your colleague have pushed some changes on upstream.
- If you push with `--force` you will replace the changes from your colleague.
- If you push with `--force-with-lease` it will only replace the upstream history if `origin/branch1` has the same commit hash as the HEAD on upstream. So if there is some change on upstream after you last fetched it will fail so that we can fetch the new changes and include them in you changes before pushing again.

**Following these steps will reduce risk of replacing others changes**
- Never do `git fetch`  or `git pull` with out specifying a branch name. Doing so will fetch all the remote branches without you not knowing. So when doing `--force-with-lease` git will check newly fetched `origin/branch1` with upstream head and replace the new changes.
- Instead use `git fetch origin branch1` or `git pull origin branch1 --rebase`. git pull actually does two things git fetch and then merge, you can use rebase instead by giving `--rebase`  option
- Requlary update your local branch with upstream changes fetching and rebasing.

#### Common practices to use git rebase
**Setup repo for practice**
>> **Summary**
>> *We'll clone the example repo and make two copies of the folder, one will be used as our local repo and other as upstream repo*
>>
>> *It is recommended that you open these two folders on separate window of your IDE to avoid confusion.*

- `git clone https://github.com/devbkhadka/git-rebase-blog.git` to get the sample repo
- make a copy of repo `cp -r git-rebase-blog git-rebase-blog-upstream`
- Add `git-rebase-blog-upstream` as origin url for `git-rebase-blog`
    ```
    cd git-rebase-blog
    git remote set-url origin $(readlink -f '../')/git-rebase-blog-upstream
    ```

**Get remote changes to local**
- Lets do few changes on local repo
- `git switch branch1` checkout `branch1` and switch to branch1
- add this text at the end of `playground.txt`
  ```
  - Nepali flag is the only triangular flag in the world.
  ```
- add changes and commit
  ```
  git add .
  git commit -m "Added fact about flag"
  ```
- Similarly add the following two lines, one in each commit
  ```
  - Momo: is the most popular fast food. Momo: is made of flour and filled with chicken, meat or vegetable fillings.
  - About 80% of the Nepalese population is Hindu.
  ```
- Now open a new terminal and cd into `git-rebase-blog-upstream`
- switch to branch1 `git switch branch1`
- Add following line to `playground.txt`
  ```
  - Nepal is rich in diverse culture and language. Nepal has over 80 ethnic group and over 120 languages.
  ```
- add changes and commit
  ```
  git add .
  git commit -m "Added a fact in upstream"
  ```
- go back to terminal with `git-rebase-blog`
- `git fetch origin branch1` to get upstream changes
- `git rebase origin/branch1`
- There will be one conflict, resolve that and do `git rebase --continue`

**Git force update example**
- Switch to terminal with `git-rebase-blog-upstream`
- add a line to playground.txt and commit
  ```
  this text will be overwritten with --force-with-lease
  ```
- `git switch main` switch to main so that we can push changes from downstream to branch1.
- Now go back to terminal with `git-rebase-blog`
- Do `git fetch` to fetch from upstream
- Now do `git push origin branch1 --force-with-lease`
- Switch to terminal with `git-rebase-blog-upstream` and do `git log --oneline branch1` to see the commits. You will see last commit on upstream repo to be replaced. This happened because we fetched all the branches from the upstream but didn't get the changes to branch1.
- switch to branch1 `git switch branch1` in `git-rebase-blog-upstream` and add another line to playground.txt and commit
  ```
  this text will not be overwritten with --force-with-lease
  ```
- `git switch main` switch to main so that we can push changes from downstream to branch1.
- Now go back to `git-rebase-blog` and do `git push origin branch1 --force-with-lease`, this should fail


**Interactive git rebase**
- Lets reset the upstream to main branch, go to terminal with `git-rebase-blog-upstream` and switch to  `git switch branch1` and run `git reset --hard origin/main`
- Now come back to terminal with `git-rebase-blog` and do `git fetch origin branch1`
- Now lets do interactive rebase `git rebase -i origin/branch1`, you will see list of commits in your branch1 as below
  ```
  pick 4905fc5 Added fact about ethnicity
  pick 93ffd54 Added fact about flag
  pick 0516467 Added fact about momo
  pick 8b19d6a Added fact about population
  ```
- We can now re-organize the commits like below
  ```
  pick 4905fc5 Added fact about ethnicity
  # reordered as commit
  # squash will combine this commit with previous
  # commit and let you change commit message
  squash 8b19d6a Added fact about population
  # this will stop the rebase after this commit and you can
  # do changes and commit with --amend option to edit it
  edit 93ffd54 Added fact about flag
  squash 0516467 Added fact about momo
  ```
- Save and exit the rebase edit file, there will be some conflict because of changing order resolve them and do `git rebase --continue` until it is rebased successfully
- check the final commit history using `git log --oneline`
