https://stackoverflow.com/questions/41951455/nexus-3-file-upload-to-hosted-maven-repository

You can't use the service urls in Nexus Repository 3. To do something like you are trying to do, try this:

curl -v -u admin:admin123 --upload-file myArtifact.jar http://nexusURL:nexusPORT/repository/myRepository/com/my/group/myArtifact/1.0.0-RC1/myArtifact-1.0.0-RC1.jar

That SHOULD do the trick?

For some good reading, you can check out the following link that explains a remote repository layout (and hopefully helps explain why what I suggested to do works):

https://cwiki.apache.org/confluence/display/MAVEN/Remote+repository+layout#Remoterepositorylayout-Repositoryartifactlayout