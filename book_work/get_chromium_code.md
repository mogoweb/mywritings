
### Install depot_tools
Clone the depot_tools repository:

```sh
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

Add depot_tools to the end of your PATH (you will probably want to put this in your ~/.bashrc or ~/.zshrc). Assuming you cloned depot_tools to /path/to/depot_tools:

```sh
$ export PATH="$PATH:/path/to/depot_tools"
```

When cloning depot_tools to your home directory do not use ~ on PATH, otherwise gclient runhooks will fail to run. Rather, you should use either $HOME or the absolute path:

```sh
$ export PATH="$PATH:${HOME}/depot_tools"
```

### Get the code

Create a chromium directory for the checkout and change to it (you can call this whatever you like and put it wherever you like, as long as the full path has no spaces):

```sh
$ mkdir ~/chromium && cd ~/chromium
```

Run the fetch tool from depot_tools to check out the code and its dependencies.

```sh
$ fetch --nohooks chromium
```

If you don't want the full repo history, you can save a lot of time by adding the --no-history flag to fetch.

Expect the command to take 30 minutes on even a fast connection, and many hours on slower ones.

If you've already installed the build dependencies on the machine (from another checkout, for example), you can omit the --nohooks flag and fetch will automatically execute gclient runhooks at the end.

When fetch completes, it will have created a hidden .gclient file and a directory called src in the working directory. The remaining instructions assume you have switched to the src directory:

```sh
$ cd src
```


```sh
# Make sure you are in 'src'.
# This part should only need to be done once, but it won't hurt to repeat it. The first
# time checking out branches and tags might take a while because it fetches an extra
# 1/2 GB or so of branch commits. 
gclient sync --with_branch_heads --with_tags

# You may have to explicitly 'git fetch origin' to pull branch-heads/
git fetch --tags

# Checkout the branch 'src' tree.
git checkout -b tvos_h 53.0.2785.134

# Checkout all the submodules at their branch DEPS revisions.
gclient sync --with_branch_heads --with_tags
```

gn gen out/Default

autoninja -C out/Default content_shell

sudo apt-get install gperf bison libgcrypt11-dev

cd /usr/lib/x86_64-linux-gnu

sudo ln -s /lib/x86_64-linux-gnu/libgcrypt.so.20.2.1 libgcrypt.so.11