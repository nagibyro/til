# Git Ignore Relative to .git directory 
When specifying a path in `.gitignore` any pattern that starts with a path
separator like `/foo.txt` is relative to the `.gitignore` file evaluated.

If you are using a global `.gitignore` file then the patterns get merged
into the current `.gitignore` file. The `/foo.txt` is not
evaluated from the root of the global `.gitignore` file. This is not made clear
in the docs.

see [https://git-scm.com/docs/gitignore](https://git-scm.com/docs/gitignore)

Also side note I didn't realize global gitignore has a default file and doesn't
have to be explicitly set in `.gitconfig`

> Its default value is $XDG_CONFIG_HOME/git/ignore. If $XDG_CONFIG_HOME is either not set or empty, $HOME/.config/git/ignore is used instead.
