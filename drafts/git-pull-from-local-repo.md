# Git: Pull from local repositories

This is a trick I use when I have multiple copies of the same repo locally.

You can add a local tracked repository in git by running `git remote add local file:///path/to/proj` (note there are triple `/` after `file:`).


And then you can just pull from your local repo by running `git pull local` which is especially helpful if you need to pull down over a large amount of data (say over 100MB or over a GB) while another local repo has the latest changes.

One downside is that, sometimes I get confused about which one of the local repos has the latest changes. I might add a script to help me find out one day.
