# Git: Pull from local repo

Some trick I use when I have multiple copies of the same repo on my machine:

You can add a `local` remote in git by running `git remote add local file:///path/to/proj` (note that triple `/` after `file:`).
And then you can just pull from your local repo by running `git pull local` which is especially helpful if you need to pull down over 100MB of data while the `local` repo has the latest changes.
