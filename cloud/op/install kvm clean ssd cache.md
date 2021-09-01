follow this instruction to clear the sssd cache:
https://www.reddit.com/r/Fedora/comments/adkdhu/usermod_adding_user_to_group_fails_because_sssd/
https://www.rootusers.com/how-to-clear-the-sssd-cache-in-linux/

### The sss_cache Tool
The cache can be cleared with the sss_cache utility which is used for performing cache cleanup by invalidating records in the SSSD cache. Invalidated records must be reloaded fresh from the identity provider server where the information actually resides, such as FreeIPA or Active Directory for example.

The -E flag can be used to invalidate all cached entries, with the exception of sudo rules.

```sss_cache -E```
Alternatively we can also simply invalidate a specific user only from the cache with the -u flag, followed by the account username.

```sss_cache -u user1```
For further information, see the [sss_cache](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sssd-cache.html) manual page.

### Deleting Cache Files
SSSD stores its cache files in the /var/lib/sss/db/ directory.

While using the sss_cache command is preferable, it is also possible to clear the cache by simply deleting the corresponding cache files.

Before doing this it is suggested that the SSSD service be stopped.

```systemctl stop sssd```
After this we want to delete all files within the /var/lib/sss/db/ directory.

```rm -rf /var/lib/sss/db/*```
Once complete we can start SSSD back up again.

```systemctl restart sssd```
SSSD should now start up correctly with an empty cache, any user login will now first go directly to the defined identity provider for authentication, and then be cached locally afterwards.

It’s recommend to only clear the cache if the identity provider servers performing the authentication within the domain are available, otherwise users will not be able to log in once the cache has been flushed.