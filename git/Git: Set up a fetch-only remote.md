https://stackoverflow.com/questions/7556155/git-set-up-a-fetch-only-remote

I don't think you can *remove* the push URL, you can only *override* it to be something other than the pull URL. So I think the closest you'll get is something like this:

    $ git remote set-url --push origin no-pushing
    $ git push
    fatal: 'no-pushing' does not appear to be a git repository
    fatal: The remote end hung up unexpectedly

You are setting the push URL to `no-pushing`, which, as long as you don't have a folder of the same name in your working directory, git will not be able to locate. You are essentially forcing git to use a location that does not exist.