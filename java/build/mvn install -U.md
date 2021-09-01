https://stackoverflow.com/questions/29360564/what-is-the-difference-between-mvn-install-u-and-mvn-install

Maven is checking for update of SNAPSHOT artifacts base on an interval. By default it is checked daily. Which means, if in the morning you got an update in SNAPSHOT, and another version is available in the afternoon in the remote repository, you will not be able to get it until tomorrow.

`-U`  options force checking for SNAPSHOT updates even the update interval is not reached.

One note to add, although the description for  `-U`  in  `mvn -h`  is

> Forces a check for updated  **releases**  and snapshots on remote repositories

base on my previous experience, releases are never checked for updates. i.e. We will always rely on whatever we previous retrieved for releases.