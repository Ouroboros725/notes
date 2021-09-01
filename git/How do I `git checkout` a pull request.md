https://github.com/genome/docs/wiki/How-do-I-%60git-checkout%60-a-pull-request%3F

You can checkout a single pull request reference by doing,
```
# replace $PR with the pull request number
git fetch origin +refs/pull/$PR/merge
git checkout FETCH_HEAD
```
If you want to fetch them all,
```
git fetch origin +refs/pull/*/merge:refs/remotes/origin/pr/*
```
For which you can then checkout the pull request by doing,
```
# replace $PR with the pull request number
git checkout origin/pr/$PR
```
If you want to automatically fetch pull requests you can add it to your repo's config,
```
git config --add remote.origin.fetch +refs/pull/*/merge:refs/remotes/origin/pr/*/merge
git fetch
```