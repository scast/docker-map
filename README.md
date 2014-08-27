docker-map
==========

Utilities for programmatically building and managing Docker images and containers.
----------------------------------------------------------------------------------

Project:


Overview
--------
Docker provides an elegant way of running various applications and services inside
of containers. This package provides additional tools for building images for these
containers, connect dependent resources, and run them in development as well as production
environments.

The library can be seen as an extension to the Docker Remote API client for Python,
`docker-py`. Based on it, available deployment tools can be enhanced
(see dockerfabric) or custom orchestration can be implemented.


Building images
---------------
Writing Dockerfiles is not hard. However, it only allows for using variable context to a
limited extent. For example, you may want to re-define directory paths in your project,
without having to adjust it in multiple places; or you keep frequently reoccurring tasks
(e.g. creating system user accounts) in your Dockerfile, and would like to use templates
rather than copy & paste.

`DockerFile`
============
Generates a Dockerfile, that can either be saved locally or sent off to Docker
through the remote API. Supports common commands such as `addfile` (`ADD`) or `run`, but
also formats `CMD` and `ENTRYPOINT` appropriately for running a shell or exec command.

`DockerContext`
===============
Generates a Docker context tarball, that can be sent to the remote API.
Its main purpose is to add files from `DockerFile` automatically, so that the Dockerfile
and the context tarball are consistent.


Creating, connecting, and running containers
--------------------------------------------
Containers can be created easily on the command line or using the Remote API, but managing
dependencies can be tedious. Whereas the path and links may be quite individual to the
local configuration, directory paths are typically constant within the containers.
This package therefore intends to reduce repetitions of names and paths in API commands,
by introducing the following main features:

* Automatically create and assign shared volumes, where the only purpose is to share data
between containers.
* Use alias names instead of paths to bind host volumes to container shares.
* Automatically create and start containers when their dependent containers are started.

`ContainerAssignment`
=====================
Keeps the elements of a configured container. Its main elements are:
* `image`: Docker image to base the container on (default is identical to container name).
* `instances`: Can generate multiple instances of a container with varying host mappings;
by default there is one main instance of each container.
* `shares`: Volumes that are simply shared by the container, only for the purpose of
keeping data separate from the container instance, or for linking the entire container
to another.
* `binds`: Host volume mappings. Uses alias names instead of directory paths.
* `uses`: Can be names of other containers, or volumes shared by another volume through
`attaches`. Has the same effect as the `volumes_from` argument in the API, but using alias
names.
* `links_to`: For container linking.
* `attaches`: Generates a separate container for the purpose of sharing data with another
one, assigns file system permissions as set in `permissions` and `user`. This makes
configuration of sockets very easy.

`ContainerMap`
==============
Contains three sets of elements:
# Container names, associated with a `ContainerAssignment`.
# Volumes, mapping shared directory paths to alias names.
# Host shares, mapping host directory paths to alias names.

`ContainerAssignment` instances and their elements can be created and used in a
dictionary-like or attribute syntax, e.g.
`container_map.container_name.uses` or
`container_map['container_name']['uses']`.
Volume aliases are stored in `container_map.volumes` and host binds in
`container_map.host`.

`MappingDockerClient`
=====================
Applies a `ContainerMap` to a Docker client. A container on the map can easily be created
with all its dependencies by running
`client.create('container_name').

Running the container can be as easy as
`client.start('container_name')
or can be enhanced with custom parameters such as
`client.start('container_name', expose={80: 80})`.

Todo
----
* More detailed introduction with examples.
* Possibly add more Docker container configuration elements, e.g. command.