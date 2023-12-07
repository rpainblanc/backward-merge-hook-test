This repository is a template to test preventing wrong merges.

Install the hook locally by moving `pre-merge-commit` to `.git/hooks`.

In this case:
- the `main` branch has a tag `12.4_init`.
- an older release branch `release/12.3` has a tag `12.3_init`
- on the `release/12.3` branch, when executing `git merge main`, the script will check if an "upper tag" version exists, and stop if one is found
- on hte contrary, if on `main` and merging `release/12.3`, as it will work
