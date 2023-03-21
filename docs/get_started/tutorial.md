# Stacker Tutorial

Stacker is a tool that allows you to build OCI images in a reproducible manner, completely unprivileged. In this tutorial, we'll introduce the stacker configuration file and perform a few simple builds.

> :pencil2: For this tutorial, we assume you have followed the [installation instructions](../developer_guide/building_stacker.md) and your environment satisfies all [runtime dependecies](../get_started/get_stacker.md).

## Your first `stacker.yaml` file

The basic input to stacker is the stacker file, such as `stacker.yaml`. This YAML-format file uses `key:value` pairs to describe what the base for your OCI image should be, and what steps must be performed to construct the image. 

Let's start with one of the smallest stacker files:

```yaml
    first:
        from:
            type: docker
            url: docker://centos:latest
```

In this example, the key named `first` represents the name of the layer. 

> :bulb: This key can have any name except `config`, which has a special usage in the stacker file. See the full documentation for [stacker yaml](../reference/stacker_file.md).

Using the small stacker file, named `first.yaml`, we can perform a basic stacker build:

<pre>
    $ <b>stacker build -f first.yaml</b>
    building image first...
    importing files...
    Getting image source signatures
    Copying blob sha256:5e35d10a3ebadf9d6ab606ce72e1e77f8646b2e2ff8dd3a60d4401c3e3a76f31
     69.60 MB / 69.60 MB [=====================================================] 16s
    Copying config sha256:44a17ce607dadfb71de41d82c75d756c2bca4db677bba99969f28de726e4411e
     862 B / 862 B [============================================================] 0s
    Writing manifest to image destination
    Storing signatures
    unpacking to /home/ubuntu/tutorial/roots/_working
    running commands...
    generating layer...
    filesystem first built successfully
</pre>

During the build, stacker downloaded the `centos:latest` tag from the docker hub and generated it as an OCI image with the additional `first` tag. We can verify the result by using the [`umoci ls`](https://man.archlinux.org/man/umoci-ls.1.en) command, which lists the tags in an image:

<pre>
    $ <b>umoci ls --layout oci</b>
    centos-latest
    first
</pre>

The `centos-latest` tag is the OCI tag for the base image, and `first` is the name of the image we generated.

If we execute a rebuild at this point, fewer steps are needed:

<pre>
    $ <b>stacker build -f first.yaml</b>
    building image first...
	importing files...
	found cached layer first
</pre>

The rebuild is shorter because stacker caches all of the inputs to a build, and only rebuilds the parts that change. The cache (and all of stacker's metadata) reside in the `/stacker` directory, from which you run stacker. 

> :bulb: Stacker's metadata can be cleaned with `stacker clean`, which also removes its entire cache.

## Adding `import` and `run` directives

At this point in our example, the only input is a base image, but what if we want to import a script to run or a config file? Consider the next stacker file example:

```yaml
    first:
        from:
            type: docker
            url: docker://centos:latest
        import:
            - config.json
            - install.sh
        run: |
            mkdir -p /etc/myapp
            cp /stacker/config.json /etc/myapp/
            /stacker/install.sh
```

If the content of `install.sh` is "echo hello world," the stacker build output will look like this:

<pre>
    $ <b>stacker build -f first.yaml</b>
	building image first...
	importing files...
	copying config.json
	copying install.sh
	Getting image source signatures
	Skipping fetch of repeat blob sha256:5e35d10a3ebadf9d6ab606ce72e1e77f8646b2e2ff8dd3a60d4401c3e3a76f31
	Copying config sha256:44a17ce607dadfb71de41d82c75d756c2bca4db677bba99969f28de726e4411e
	 862 B / 862 B [============================================================] 0s
	Writing manifest to image destination
	Storing signatures
	unpacking to /home/ubuntu/tutorial/roots/_working
	running commands...
	running commands for first
	+ mkdir -p /etc/myapp
	+ cp /stacker/config.json /etc/myapp
	+ /stacker/install.sh
	hello world
	generating layer...
	filesystem first built successfully
</pre>

In this latest stacker file, we've added an `import` section, with two new directives :

```yaml
    import:
        - config.json
        - install.sh
```

This new section imports two files into the `/stacker` directory inside the image. This directory will not be present in the final image, so you must copy any needed files from this directory into their final place in the image. Also, importing files from the web (via URLs like  http://example.com/foo.tar.gz) is supported, and these files will also be cached on disk. 

> :pencil2: If a file is already cached, stacker will not access the URL again. If the file at the URL changes, you must run `stacker build` with the `--no-cache` argument or you can delete the file from its cached location (in this case, `/stacker/imports/$target_name/foo.tar.gz`).

The other new addition in our latest stacker file example is a `run` section:

```yaml
    run: |
        mkdir -p /etc/myapp
        cp /stacker/config.json /etc/myapp/
        /stacker/install.sh
```

This section lists the commands that will be run in order to install and configure the image.

Note that the build in the latest example used a cached version of the base layer again, but then rebuilt the part where you asked for commands to be run, since that is new.

## dev/build containers

Finally, stacker offers "build only" containers, which are built but not emitted in the final OCI image. Consider this stacker file example:

```yaml
    build:
        from:
            type: docker
            url: docker://ubuntu:latest
        run: |
            apt update
            apt install -y software-properties-common git
            apt-add-repository -y ppa:gophers/archive
            apt update
            apt install -y golang-1.9
            export PATH=$PATH:/usr/lib/go-1.9/bin
            export GOPATH=~/go
            mkdir -p $GOPATH/src/github.com/openSUSE
            cd $GOPATH/src/github.com/openSUSE
            git clone https://github.com/opencontainers/umoci
            cd umoci
            make umoci.static
            cp umoci.static /
        build_only: true
    umoci:
        from:
            type: docker
            url: docker://centos:latest
        import: stacker://build/umoci.static
        run: cp /stacker/umoci.static /usr/bin/umoci
```

This file builds a static version of umoci in an ubuntu container, but the final image will contain only an `umoci` tag with a statically linked version of `umoci` at `/usr/bin/umoci`. There are a few new directives to support this result:

```yaml
    build_only: true
```

This line indicates that the container shouldn't be emitted in the final image, because we're only going to import something from it and we don't need the rest of it.

```yaml
    import: stacker://build/umoci.static
```

This line performs the actual import. The line calls for this action: "From a previously built stacker image called `build`, import `/umoci.static`."
