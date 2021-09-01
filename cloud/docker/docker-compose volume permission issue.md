https://stackoverflow.com/questions/53170709/docker-compose-could-not-open-directory-permisson-denied

I solved by adding ":z" to end of volume defintion

```
version: '3'
services:
  db:
    image: postgres
    container_name: dummy_project_postgres
    volumes:
      - ./data/db:/var/lib/postgresql/data:z

  event_planner:
    build: ./dummy_project
    container_name: dummy_project
    volumes:
      - .:/web
    ports:
      - "8000:8000"
    depends_on:
      - db
    links:
      - db:postgres
```

What ":z" means

> Labeling systems like SELinux require that proper labels are placed on volume content mounted into a container. Without a label, the security system might prevent the processes running inside the container from using the content. By default, Docker does not change the labels set by the OS.<br/><br/>
To change the label in the container context, you can add either of two suffixes :z or :Z to the volume mount. These suffixes tell Docker to relabel file objects on the shared volumes. The z option tells Docker that two containers share the volume content. As a result, Docker labels the content with a shared content label. Shared volume labels allow all containers to read/write content. The Z option tells Docker to label the content with a private unshared label. Only the current container can use a private volume.

https://docs.docker.com/engine/reference/commandline/run/#mount-volumes-from-container---volumes-from

[what is 'z' flag in docker container's volumes-from option?](https://stackoverflow.com/questions/35218194/what-is-z-flag-in-docker-containers-volumes-from-option)