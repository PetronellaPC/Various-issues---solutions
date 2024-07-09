Podman sock setup to replace Docker sock for testcontainers - python and/or Java on MacOS

Some use Docker with Colima, and might work, for me it did not. 
I use Podman on MacOS, it has a few advantages. It is a Red Hat product and for thiose using Red Hat Linux distros and Cloud products it's a natural thing to use Podman.
I encountered several problems with Docker with Lima/Colima on my machine, I eventually ended up removing all the instances of the multiple VMs I configured and tried with lima. The main problem for me was not being able to run testcontainers with docker - lima, regardless of VM configurations, docker socket mapping, env setting ( DOCKER_HOST ,
TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE etc.). So I switched to Podman. And It works.

There are 2 options:
  1. Forwarding the port
  2. Using the Podman sock configured on the localhost.

The 1st method I'm putting because I saw it in a few solutions proposed on various sources. I used it for a while but at some point some network configs prevented it from working. Still, here it is:
  i. get the sock on the podman VM : 
      `podman system info | grep sock`
it should be something of the sort: `/run/user/${UID}/podman/podman.sock`. Alternatively, ssh into the podman machine - if only one machine installed use
podman machine ssh and manually get the UID with `echo ${UID}`

  ii. To get the podman machine port use :
      `podman system connection list --format=json | jq '.[0].URI' | sed -E 's|.+://.+@.+:([[:digit:]]+)/.+|\1|' ` 
`jq`is required, or run 
      `podman system connection list` 
to see the podman machine port in the URI. 
  iii. For convenience set an alias for the forwarding command. In this command the ssh key to the podman machine is required `-i ~/.ssh/machine` it might have a different name on your machine. It can be found in `~/.local/share/containers/podman/machine/applehv/`
      `alias podman-sock="rm -f /tmp/podman.sock && ssh -i ~/.ssh/machine -p $(podman system connection list --format=json | jq '.[0].URI' | sed -E 's|.+://.+@.+:([[:digit:]]+)/.+|\1|') -L'/tmp/podman.sock:/run/user/${UID}/podman/podman.sock' -N -v -v -v core@localhost"`
The multiple `-v` are for the verbosity level, more `v`s, more details. Feel free to remove them.
  iv. Set the env vars that testcontainers use: 
      `export DOCKER_HOST=unix:///tmp/podman.sock`
      `export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/tmp/podman.sock`
      `export TESTCONTAINERS_RYUK_DISABLED=true`
The TESTCONTAINERS_RYUK_DISABLED tels testecontainer not to use the `testcontainers/ryuk:version` which usually created problems. 

This should work for both Java and Python testcontainers. 

The other way that I set up by combining different sources an info is simpler, I find:
Basically, there is one sock on localhost connected to the podman sock on the podman VM. The location can be seen runnig: 
  `podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}'`
To this sock there are 2 soft links set, namelly the `/var/run/docker.sock` that is the soft link to `~/.local/share/containers/podman/machine/podman.sock` which is the soft link to the localhost sock connected to the Podman vm sock
So, just set: 
      `export DOCKER_HOST=unix://$(podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}')`
      `export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock`
      `export TESTCONTAINERS_RYUK_DISABLED=true`
      `export TESTCONTAINERS_RYUK_PRIVILEGED=false`
and you should be all set. 
