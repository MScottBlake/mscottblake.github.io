---
title: Improve your AutoPkg repo-add Commands
date: 2025-07-21T20:30:25-04:00
categories:
  - blog
  - AutoPkg
---

Over the years, I have seen a lot of instances of one of these two snippets used to configure AutoPkg to add a list of recipe repositories.

```bash
for repo in $(cat repo_list.txt); do
  autopkg repo-add $repo
done
```

```bash
while read -r line ; do
  autopkg repo-add $line
done < repo_list.txt
```

These both work, but they are inefficient. In both cases, you are looping over the list and running `autopkg repo-add` on each repository. The more repositories you add to your list, the slower it becomes.

I'm here to tell you that there is a better way. The `repo-add` verb can take a list of repositories, so you only need to call it once.

```bash
xargs autopkg repo-add < repo_list.txt
```

This snippet is both easier to read and more performant. In essense, it reads the contents of the file and runs a single command to add all of the repositories at once.

If you are running AutoPkg in an environment that charges by the minute, little performance boosts like this can really add up. Small change, big payoff.

Happy automating!
