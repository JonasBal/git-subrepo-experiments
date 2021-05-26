# git-subrepo: Usage Scenarios

This is a list of scenarios you might try out in order to get to know [git subrepo](https://github.com/ingydotnet/git-subrepo) better. 

For the following scenarios, assume we have 2 repositories:
- "project_bare" repo (I might also refer to it as "project repo"). `/tmp/project`
- "shared" repo (`/tmp/shared` as standalone repo, and `/tmp/project/shared` as subrepo of project) 

The shared repo is contained in the project repo as described in [git subrepo tutorial](https://github.com/ingydotnet/git-subrepo/wiki/Basics). 
You may repeat the steps of [that tutorial](https://github.com/ingydotnet/git-subrepo/wiki/Basics) prior to trying out the following scenarios.
(Note: In that [tutorial](https://github.com/ingydotnet/git-subrepo/wiki/Basics), the remote of the shared repo is just a regular directory in your file system. For my tests, I have set up remote repos on GitHub for both shared and project repos, but this should not make any change to the outcomes.)

# Scenario 1 - Pull changes of shared repo (master branch) into project_bare

1. We go to the `/tmp/shared` directory. We make some changes and push them with normal git commands.
2. We go in our `/tmp/project` directory.
3. We do a `git subrepo pull --all`. The changes of shared repo have been downloaded to `/tmp/project/shared/` successfully.

Please note that the pulling of shared repository is now represented as a commit in project_bare history.

# Scenario 2 - Handling of `.gitignored` files in subrepo

I have noticed in my file manager, that the `/tmp/project/shared` directory disappears and reappears when I do a `git subrepo pull --all`. Is the directory being deleted and recloned completely? We won't investigate the magic behind `git subrepo pull`, but we are going to make a little experiment and find out what will happen to .gitignored files during that procedure.

1. Go to `/tmp/shared` and create a `.gitignore` file. Add there an ignored directory, so that all files in that directory will be ignored by git. Push the `.gitignore` file changes.
2. Pull changes of shared repo to `/tmp/project` directory, as in Scenario 1.
3. Go to `/tmp/project/shared`. Create the .gitignored directory and add some files in there. Make a `git status` to ensure that git does not track those files.

Now, what will happen if we pull some more changes from shared repository?

4. Make some changes in shared repository and push them, as in Scenario 1.
5. Pull changes of shared in project_bare with `git subrepo pull --all`, as in Scenario 1.

Is our .gitignored data in shared directory of project bare still there?
--> Yes. Go check it out.


# Scenario 3 - Make some changes in shared from project_bare repo

1. Go in the `/tmp/project/shared` directory make there some changes. Push them with usual git commands to the project repo.

If we now look at our project_bare repository, we see that our last changes of project_bare/shared are present. But they have not been pushed  yet to the shared repo itself.

3. In order to push our changes also to the shared repo, we need to `git subrepo push shared` from the `/tmp/project` directory. 

Now the changes are also present in the shared repository.

# Scenario 4 - Different changes in `/tmp/project/shared` and `/tmp/shared`

We will investigate what will happen if we make different changes in those different instances of the shared repo.

1. Go to `/tmp/shared`, create a file `scenario_4_test` with content `scenario 4 from /tmp/shared`. 
2. Go to `/tmp/project/shared`, create a file `scenario_4_test` with content `scenario 4 from /tmp/project/shared`.

Now we have 2 different versions of the same file, in the same point of history. 
How will git-subrepo handle that?

3. Git push changes from `/tmp/project`.
4. Git push changes from `/tmp/shared`.

In both repos we now have different versions of `scenario_4_test` in shared directory. 

What will happen if we `git subrepo pull --all` in `project` directory?

5. Run `git subrepo pull --all`. We get some longer output this time.

```
 Auto-merging scenario_4_test
  CONFLICT (add/add): Merge conflict in scenario_4_test
  Automatic merge failed; fix conflicts and then commit the result.
```

That's great that we get this message! Otherwise, we could not detect if we would accidentally overwrite changes. 

Option 1 for us: We could do what subrepo suggests to us in the same output (basically resolving a merge conflict, straight-forward): 

```
  1. cd .git/tmp/subrepo/shared
  2. Resolve the conflicts (see "git status").
  3. "git add" the resolved files.
  4. git commit
  5. If there are more conflicts, restart at step 2.
  6. cd /tmp/project
  7. git subrepo commit shared

```
Please note, that we do not resolve the conflict in `/tmp/project/share`, but a custom directory that subrepo has created for us: `.git/tmp/subrepo/shared`.

Option 2: Ignore "local" changes.
What if we don't care about the merge conflicts and simply want to pull the lastest version of shared repo, and discard our "local" changes in `project/share`?

6. We first abort the merge with `git subrepo clean shared` (this command was also suggested by subrepo during the pull)
        - Please note that after this command, the custom temporary merge directory has been deleted (`/tmp/project/share`)
7. We run a `git subrepo pull --all --force`. When looking at the new contents of `scenario_4_test`, we see that our old content has been overwritten with content from shared repo. 
8. We can run a `git push` to upload these changes to the project repo. 


# Scenario 5 - Creating and pushing to a new branch (changes in both project and shared repos)

In this scenario, we will create a new branch in the project repo, and make some changes to files.

1. Go to `/tmp/project`. Create a new git branch (` git checkout -b "scenario_5"`). Make some changes to files in that directory. Also make some changes to files in `/tmp/project/shared`.
2. Push those changes to the new branch of project repo with usual git commands. Also push the changes with `git subrepo push --all`.

What do you think, to which branch have the changes been pushed in the subrepo?

They have been pushed to the `master` branch of shared repo. Subrepo did not handle the creation of the new branch for us. This is important to know for the developers, because they can easily accidentally push to a wrong branch. Think of [protecting important branches](https://docs.github.com/en/github/administering-a-repository/about-protected-branches) beforehand, just in case.

How do we do branching of shared repo from project repo then?

3. We push subrepo changes: `git subrepo push shared -b scenario_5`

The new branch is nor being created for us. Now changes have been also pushed in the new branch of shared repository.

# Scenario 5.1 - Working on the new branch

1. Make some changes in project repo, still on `scenario_5` branch. Make some commits with changes in `/tmp/project/shared`, `/tmp/project`, and push them.
2. After pushing to project repo, push to the subrepo to the same branch: `git subrepo push shared -b scenario_5`


# Scenario 6 - Switching branch of project and shared at the same time

We still are on the `scenario_5 branch`, and we want to change back to master.

1. `git checkout master`

Well, that's it. Nothing more to do here. Changes in `/tmp/project/shared` are being checked out to the version in project repo automatically.

# Scenario 6.1 - Switching only the branch of shared

Consider if you really want to switch the branch of shared repo, independently from project_bare repo.

1. `git subrepo pull shared -b scenario_5.1`

Please note the latest commit made to project repo (`git log --oneline`):
- `1234692 (HEAD -> master) git subrepo pull --branch=scenario_5.1 shared`

Changing back to master:
2. `git subrepo -f pull shared -b master`

# Scenario 7 - Pushing changes in a branch of project and shared at the same time

1. Make some commits in `/tmp/shared`, do not push yet.
2. Make some commits in `/tmp/project/shared` and push.
3. Push from `/tmp/shared`

How do we get latest changes from shared in project repo?

4. `git subrepo pull shared` raises some merge conflicts (similar to scenario 4)

Now we can decide what we will do with the merge conflict. Continue as in scenario 4 (e.g. `git subrepo pull --all --force`).


# Scenario 8 - What about a merge request (pull request) related to both repos? How to handle that?

One positive aspect of using submodule is, that in our merge request branch, we still have the code of the shared repo (with a `.gitrepo` file in there). That way, it is indeed possible to indirectly review content of shared repo from project_bare repo.

However, changes to the subrepo should be of course applied in the subrepo repository. Thus, we need two merge requests. 
You could consider discussing and reviewing code in the project merge request only, though.

# Scenario 9 - Rolling back to a previous state from project repo

Imagine e.g. that we need to rollback after an unsuccessful update and want to return to some previous commit from project repo.

We can do this just as usual by `git reset --hard <Commit-SHA>`, as this will also automatically pull the correct shared repo version, which was present in that commit.


# Scenario 10 - Simplifying workflows

Can some commands be summarized, e.g. with a [git alias](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwj75fPj_ubwAhXEDewKHSKMBrMQFjAAegQIBBAD&url=https%3A%2F%2Fgit-scm.com%2Fbook%2Fen%2Fv2%2FGit-Basics-Git-Aliases&usg=AOvVaw2o_yfXItiodGnhmpOIbWVZ)?


