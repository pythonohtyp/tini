Tini - A tiny but valid `init` for containers
=============================================

[![Build Status](https://travis-ci.org/krallin/tini.svg?branch=master)](https://travis-ci.org/krallin/tini)

Tini is the simplest `init` you could think of.

All Tini does is spawn a single child (Tini is meant to be run in a container),
and wait for it to exit all the while reaping zombies and performing
signal forwarding.


Using Tini
----------

Add Tini to your container, and make it executable. Then, just invoke Tini
and pass your program and its arguments as arguments to Tini.

In Docker, you will want to use an entrypoint so you don't have to remember
to manually invoke Tini:

    # Add Tini
    ENV TINI_VERSION v0.3.3
    ADD https://github.com/krallin/tini/releases/download//tini /tini
    RUN chmod +x /tini
    ENTRYPOINT ["/tini", "--"]

    # Run your program under Tini
    CMD ["/your/program", "-and", "-its", "arguments"]
    # or docker run your-image /your/program ...

Note that you *can* skip the `--` under certain conditions, but you might
as well always include it to be safe. If you see an error message that
looks like `tini: invalid option -- 'c'`, then you *need* to add the `--`.

Arguments Tini itself are passed like so: `/tini -v -- /your/program`.
The only supported argument at this time is `-v`, for extra verbosity (you can
pass it up to 4 times, e.g. `-vvvv`).

*NOTE*: The binary linked above is a 64-bit dynamically-linked binary.


### Existing Entrypoint ###

Tini can also be used with an existing entrypoint in your container!

Assuming your entrypoint was `/docker-entrypoint.sh`, then you would use:

    ENTRYPOINT ["/tini", "--", "/docker-entrypoint.sh"]


### Size Considerations ###

Tini is a very small file (in the 10KB range), so it doesn't add much weight
to your container.


### Building Tini ###

If you'd rather not download the binary, you can build Tini by just running
`make` (i.e. there is no `./configure` script).


Understanding Tini
------------------

After spawning your process, Tini will wait for signals and forward those
to the child process, and periodically reap zombie processes that may be
created within your container.

When the "first" child process exits (`/your/program` in the examples above),
Tini exits as well, with the exit code of the child process (so you can
check your container's exit code to know whether the child exited
successfully).


Debugging
---------

If something isn't working just like you expect, consider increasing the
verbosity level (up to 4):

    tini -v    -- bash -c 'exit 1'
    tini -vv   -- true
    tini -vvv  -- pwd
    tini -vvvv -- ls
