# Fuchsia Manifest

Contains the jiri manifests for Fuchsia.

## Prerequisites

On Ubuntu:

 * `sudo apt-get install golang git-all build-essential lzip curl`

On Mac:

 * `TODO(abarth): Figure out what needs to be installed.`

## Creating a new checkout

Fuchsia uses the `jiri` tool to manage repositories
[https://github.com/vanadium/go.jiri](https://github.com/vanadium/go.jiri).
This tool manages a set of repositories specified by a manifest.  The bootstrap
procedure requires that you have Go 1.4 or newer and Git installed and on your
PATH.  To create a new Fuchsia checkout in a directory called `fuchsia` run the
following commands. The `fuchsia` directory should not exist before running
these steps.

```
curl -s https://raw.githubusercontent.com/vanadium/go.jiri/master/scripts/bootstrap_jiri | bash -s fuchsia
cd fuchsia
export PATH=`pwd`/.jiri_root/scripts:$PATH
jiri import fuchsia https://fuchsia.googlesource.com/manifest
jiri update
```

### Working without altering your PATH

If you don't like having to mangle your environment variables, and you want `jiri`
to "just work" depending on your current working directory, there's a shim for that.

```
sudo cp .jiri_root/scripts/jiri /usr/local/bin
sudo chmod 755 /usr/local/bin/jiri
```

That script is just a wrapper around the real `jiri`.  It crawls your parent directory
structure until it finds a `.jiri_root` directory, then executes the `jiri` it finds there.

## Submitting changes

To submit a patch to Fuchsia, you may first need to generate a cookie to
authenticate you to GoogleSource (you know this is necessary if ```jiri cl mail```
prompts you for a username/password).  To generate a cookie, log into
GoogleSource and click the "Generate Password" link at the top of
https://fuchsia.googlesource.com. Then, copy the generated text and execute it
in a terminal.

Once authenticated, follow these steps to submit a patch to Fuchsia:

```
# create a new change (makes a new local branch, checks it out)
jiri cl new branch_name

# write some awesome stuff, commit to branch_name
vim some_file ...
git commit ...

# upload the patch to gerrit
jiri cl mail

# once the change is landed, clean up the branch
jiri cl cleanup branch_name
```

## Cross-repo changes

Changes in two separate repos will be automatically tracked for you by `jiri`
if you use the same branch name.

```
# make and commit the first change
cd fuchsia/bin/fortune
jiri cl new add_feature_foo
vim foo_related_files ...
git commit ...

# make and commit the second change, with the same branch name ('add_feature_foo')
cd fuchsia/build
jiri cl new add_feature_foo
vim more_foo_related_files ...
git commit ...

# upload both changes to gerrit
jiri cl mail

# after the changes are reviewed, approved and submitted, cleanup the local branch
jiri cl cleanup add_feature_foo
```

Multipart changes are tracked in gerrit via topics, are tested together, and
can be landed in Gerrit at the same time with `submit whole topic`.
