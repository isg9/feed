---
title: hacking v8 with guix, bis — wingolog
url: https://wingolog.org/archives/2024/03/26/hacking-v8-with-guix-bis
published: "2024-03-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/03/26/hacking-v8-with-guix-bis
---

# hacking v8 with guix, bis — wingolog

## [hacking v8 with guix, bis](/archives/2024/03/26/hacking-v8-with-guix-bis)

26 March 2024 11:51 AM

- [v8](/tags/v8)
- [guix](/tags/guix)
- [depot\_tools](/tags/depot_tools)

Good day, hackers. Today, a pragmatic note, on hacking on [V8](https://v8.dev/) from a [Guix](https://guix.gnu.org/) system.

I’m going to skip a lot of the background because, as it turns out, I [wrote about this already almost a decade
ago](https://wingolog.org/archives/2015/08/04/developing-v8-with-guix). But following that piece, I mostly gave up on doing V8 hacking from a Guix machine—it was more important to just go with the flow of the ever-evolving upstream toolchain. In fact, I ended up installing Ubuntu LTS on my main workstations for precisely this reason, which has worked fine; I still get Guix in user-space, which is better than nothing.

Since then, though, Guix has grown to the point that it’s easier to create an environment that can run a complicated upstream source management project like V8’s. This is mainly [`guix shell`](https://guix.gnu.org/manual/en/html_node/Invoking-guix-shell.html) in the `--container --emulate-fhs` mode. This article is a step-by-step for how to get started with V8 hacking using Guix.

### get the code

You would think this would be the easy part: just `git clone` the V8 source. But no, the build wants a number of other Google-hosted dependencies to be vendored into the source tree. To perform the initial fetch for those dependencies and to keep them up to date, you use helpers from the [`depot_tools`](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html) project. You also use `depot_tools` to submit patches to code review.

When you live in the Guix world, you might be tempted to look into what `depot_tools` actually does, and to replicate its functionality in a more minimal, Guix-like way. Which, sure, perhaps this is a good approach for *packaging* V8 or Chromium or something, but when you want to work *on* V8, you need to learn some humility and just go with the flow. (It’s hard for the kind of person that uses Guix. But it’s what you do.)

You can make some small adaptations, though. `depot_tools` is mostly written in Python, and it actually bundles its own `virtualenv` support for using a specific python version. This isn’t strictly needed, so we can set the funny environment variable `VPYTHON_BYPASS="manually managed python not supported by chrome operations"` to just use python from the environment.

Sometimes `depot_tools` will want to run some prebuilt binaries. Usually on Guix this is anathema—we always build from source—but there’s only so much time in the day and the build system is not our circus, not our monkeys. So we get Guix to set up the environment using a container in [`--emulate-fhs`](https://guix.gnu.org/manual/en/html_node/Invoking-guix-shell.html#index-FHS-_0028file-system-hierarchy-standard_0029) mode; this lets us run third-party pre-build binaries. Note, these binaries are indeed free software! We can run them just fine if we trust Google, which you have to when working on V8.

### no, really, get the code

Enough with the introduction. The first thing to do is to check out `depot_tools`.

```
mkdir src
cd src
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

```

I’m assuming you have `git` in your Guix environment already.

Then you need to initialize `depot_tools`. For that you run a python script, which needs to run other binaries – so we need to make a specific environment in which it can run. This starts with a *manifest* of packages, is conventionally placed in a file named `manifest.scm` in the project’s working directory, though you don’t have one yet, so you can just write it into `v8.scm` or something anywhere:

```
(use-modules (guix packages)
             (gnu packages gcc))

(concatenate-manifests
 (list
  (specifications->manifest
   '(
     "bash"
     "binutils"
     "clang-toolchain"
     "coreutils"
     "diffutils"
     "findutils"
     "git"
     "glib"
     "glibc"
     "glibc-locales"
     "grep"
     "less"
     "ld-gold-wrapper"
     "make"
     "nss-certs"
     "nss-mdns"
     "openssh"
     "patch"
     "pkg-config"
     "procps"
     "python"
     "python-google-api-client"
     "python-httplib2"
     "python-pyparsing"
     "python-requests"
     "python-tzdata"
     "sed"
     "tar"
     "wget"
     "which"
     "xz"
     ))
  (packages->manifest
   `((,gcc "lib")))))

```

Then, you `guix shell -m v8.scm`. But you actually do more than that, because we need to set up a container so that we can expose a standard `/lib`, `/bin`, and so on:

```
guix shell --container --network \
  --share=$XDG_RUNTIME_DIR --share=$HOME \
  --preserve=TERM --preserve=SSH_AUTH_SOCK \
  --emulate-fhs \
  --manifest=v8.scm

```

Let’s go through these options one by one.

- `--container`: This is what lets us run pre-built binaries, because it uses Linux namespaces to remap the composed packages to `/bin`, `/lib`, and so on.

- `--network`: Depot tools are going to want to download things, so we give them net access.

- `--share`: By default, the container shares the current working directory with the “host”. But we need not only the checkout for V8 but also the sibling checkout for depot tools (more on this in a minute); let’s just share the whole home directory. Also, we share the `/run/user/1000` directory, which is `$XDG_RUNTIME_DIR`, which lets us access the SSH agent, so we can check out over SSH.

- `--preserve`: By default, the container gets a pruned environment. This lets us pass some environment variables through.

- `--emulate-fhs`: The crucial piece that lets us bridge the gap between Guix and the world.

- `--manifest`: Here we specify the list of packages to use when composing the environment.

We can use short arguments to make this a bit less verbose:

```
guix shell -CNF --share=$XDG_RUNTIME_DIR --share=$HOME \
  -ETERM -ESSH_AUTH_SOCK -m manifest.scm

```

I would like it if all of these arguments could somehow be optional, that I could get a bare `guix shell` invocation to just apply them, when run in this directory. Perhaps some day.

Running `guix shell` like this drops you into a terminal. So let’s initialize depot tools:

```
cd $HOME/src
export VPYTHON_BYPASS="manually managed python not supported by chrome operations"
export PATH=$HOME/src/depot_tools:$PATH
export SSL_CERT_DIR=/etc/ssl/certs/
export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
gclient

```

This should download a bunch of things, I don’t know what. But at this point we’re ready to go:

```
fetch v8

```

This checks out V8, which is about 1.3 GB, and then probably about as much again in dependencies.

### build v8

You can build V8 directly:

```
# note caveat below!
cd v8
tools/dev/gm.py x64.release

```

This will build fine... and then fail to link. The precise reason is obscure to me: it would seem that by default, V8 uses a whole Debian sysroot for Some Noble Purpose, and ends up linking against it. But it compiles against system glibc, which seems to have replaced `fcntl64` with a versioned symbol, or some such nonsense. It smells like V8 built against a too-new glibc and then failed trying to link to an old glibc.

To fix this, you need to go into the `args.gn` that was generated in `out/x64.release` and then add `use_sysroot = false`, so that it links to system glibc instead of the downloaded one.

```
echo 'use_sysroot = false' >> out/x64.release/args.gn
tools/dev/gm.py x64.release

```

You probably want to put the commands needed to set up your environment into some shell scripts. For Guix you could make `guix-env`:

```
#!/bin/sh
guix shell -CNF --share=$XDG_RUNTIME_DIR --share=$HOME \
  -ETERM -ESSH_AUTH_SOCK -m manifest.scm -- "$@"

```

Then inside the container you need to set the `PATH` and such, so we could put this into the V8 checkout as `env`:

```
#!/bin/sh
# Look for depot_tools in sibling directory.
depot_tools=`cd $(dirname $0)/../depot_tools && pwd`
export PATH=$depot_tools:$PATH
export VPYTHON_BYPASS="manually managed python not supported by chrome operations"
export SSL_CERT_DIR=/etc/ssl/certs/
export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
exec "$@"

```

This way you can run `./guix-env ./env tools/dev/gm.py x64.release` and not have to “enter” the container so much.

### notes

This all works fine enough, but I do have some meta-reflections.

I would prefer it if I didn’t have to use containers, for two main reasons. One is that the resulting build artifacts have to be run in the container, because they are dynamically linked to e.g. `/lib`, at least for the ELF loader. It would be better if I could run them on the host (with the host debugger, for example). Using Guix to make the container is better than e.g. docker, though, because I can ensure that the same tools are available in the guest as I use on the host. But also, I don’t like adding “modes” to my terminals: are you in or out of this or that environment. Being in a container is not like being in a vanilla `guix shell`, and that’s annoying.

The build process uses many downloaded tools and artifacts, including `clang` itself. This is a feature, in that I am using the same compiler that colleagues at Google use, which is important. But it’s also annoying and it would be nice if I could choose. (Having the same `clang-format` though is an absolute requirement.)

There are two tests failing, in this configuration. It is somehow related to time zones. I have no idea why, but I just ignore them.

If the build system were any weirder, I would think harder about maybe using Docker or something like that. Colleagues point to [`distrobox`](https://github.com/89luca89/distrobox) as being a useful wrapper. It is annoying though, because such a docker image becomes like a little stateful thing to do sysadmin work on, and I would like to avoid that if I can.

Welp, that’s all for today. Hopefully if you are contemplating installing Guix as your operating system (rather than just in user-space), this can give you a bit more information as to what it might mean when working on third-party projects. Happy hacking and until next time!

## related articles

- [developing v8 with guix](/archives/2015/08/04/developing-v8-with-guix)
- [wastrel milestone: full hoot support, with generational gc as a treat](/archives/2026/04/09/wastrel-milestone-full-hoot-support-with-generational-gc-as-a-treat)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)

Comments are closed.
