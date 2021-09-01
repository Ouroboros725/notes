https://stackoverflow.com/questions/35123108/cherry-pick-and-squash-a-range-of-commits-into-a-subdirectory-or-subtree

Pass `-n` to `git cherry-pick`. This will apply all the commits, but not commit them. Then simply do `git commit` to commit all the changes in a single commit.