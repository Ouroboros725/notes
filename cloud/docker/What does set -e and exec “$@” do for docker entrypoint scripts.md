https://stackoverflow.com/questions/39082768/what-does-set-e-and-exec-do-for-docker-entrypoint-scripts

It basically takes all the extra command line arguments and execs them as a command. The intention is basically "Do everything in this .sh script, then in the same shell run the command the user passes in on the command line".

See:

* https://stackoverflow.com/questions/5163144/what-are-the-special-dollar-sign-shell-variables
* https://stackoverflow.com/questions/9920512/need-explanations-for-linux-bash-builtin-exec-command-behavior