---
layout: default
title: "Remove Untagged Images From Docker"
---

I've been playing around a lot with [docker](http://docker.io). It's
awesome, and it creates a whole new world of possibilities, and I'm
constantly coming up with new ideas of where it could be useful.

After playing with docker for about a week on my development server, I
logged in to find that my disk was completely full. I guess after
dynamically spinning up dozens of containers, and building a bunch of
projects with
[Dockerfiles](http://docs.docker.io/en/latest/use/builder/) I had
accumulated quite a few stopped containers and untagged images. I
suspect the build process to be the biggest contributor to this, as each
step in your dockerfile creates a new container, which serves as the
base for the next step. This is usfeul because it can cache the
containers and speed up builds, but it does consume a bit of space.

I was not able to find any built-in commands for clearing stopped
containers and untagged images, so I was able to put together a couple
commands.

## Remove all stopped containers.


```
docker rm $(docker ps -a -q)
```

This will remove all stopped containers by getting a list of all
containers with `docker ps -a -q` and passing their ids to docker rm.
This should not remove any running containers, and it will tell you it
can't remove a running image.

## Remove all untagged images

In the process of running docker I had accumulated several images that
are not tagged. To remove these I use this command:

```
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```

This works by using rmi with a list of image ids. To get the image ids
we call `docker images` then pipe it to `grep "^<none>"`. The grep will
filter it down to only lines with the value "&lt;none>" in the repository
column. Then to extract the id out of the third column we pipe it to
`awk "{print $3}"` which will print the third column of each line passed
to it.

After running these two commands I recovered 15G of space. There may be
more I could do to recover more space, my docker graph directory still
is over 5G, but for now this works.
