https://unix.stackexchange.com/questions/6435/how-to-check-if-pwd-is-a-subdirectory-of-a-given-path

To test if a string is a prefix of another, in any Bourne-style shell:

    case $PWD/ in
      /home/*) echo "home sweet home";;
      *) echo "away from home";;
    esac

The same principle works for a suffix or substring test. Note that in `case` constructs, unlike in file names, `*` matches any character, including a `/` or an initial `.`.

In shells that implement the `[[ … ]]` syntax (i.e. bash, ksh and zsh), it can be used to match a string against a pattern. (Note that the `[` command can only test strings for equality.)

    if [[ $PWD/ = /home/* ]]; then …

----

If you're specifically testing whether the current directory is underneath `/home`, a simple substring test is not enough, because of symbolic links.

If `/home` is a filesystem of its own, test whether the current directory (`.`) is on that filesystem.

    if [ "$(df -P . | awk 'NR==2 {print $6}')" = "/home" ]; then
      echo 'The current directory is on the /home filesystem'
    fi

If you have the NetBSD, OpenBSD or GNU (i.e. Linux) `readlink`, you can use `readlink -f` to strip symbolic links from a path.

    case $(readlink -f .)/ in $(readlink -f /home)/*) …

Otherwise, you can use `pwd` to show the current directory. But you must take care not to use a shell built-in if your shell tracks `cd` commands and keeps the name you used to reach the directory rather than its “actual” location.

    case $(pwd -P 2>/dev/null || env PWD= pwd)/ in
      "$(cd /home && { pwd -P 2>/dev/null || env PWD= pwd; })"/*) …
  
---  

If you want to reliably test whether a directory is a subdirectory of another, you'll need more than just a string prefix check.  Gilles' answer describes in detail how to do this test properly.

But if you do want a simple string prefix check (maybe you've already normalized your paths?), this is a good one:

    test "${PWD##/home/}" != "${PWD}"

If `$PWD` starts with "/home/", it gets stripped off in the left side, which means it won't match the right side, so "!=" returns true.