---
layout: post
title:  "How to handle git when submodules changed"
date:   2022-02-09 35:45:00 +0100
categories: [development]
---

This is just a short note for a particular nuisance when working with git and submodules, at least as long as one does not know how to resolve the issue.

I am working on a rather large open-source project. The main git repository contains most of the code, but also pulls in many dependencies as submodules. From time to time I ran into issues with the submodules, when the submodules were moved around or when the remote URLs of the submodules changed. This often shows as an error where git can no longer find a certain commit:


When submodules had their remote URLs changed, but the change was not been synced locally yet, git will fail to find newer commits:

```
fatal: remote error: upload-pack: not our ref 9386b9f73522fefb2ccbe0ce1ccdc681f5f4a86f
fatal: the remote end hung up unexpectedly
Fetched in submodule path '<path-to-submodule>', but it did not contain 9386b9f73522fefb2ccbe0ce1ccdc681f5f4a86f. Direct fetching of that commit failed.
```

And when the paths to some submodules were modified or renamed, git will often make it impossible to move between commits with the following error:

```
error: The following untracked working tree files would be overwritten by checkout:
	<path-to-submodule/path-to-file1>
	<path-to-submodule/path-to-file2>
    <path-to-submodule/path-to-file3>
    ...
```

The solution is the following. First deinitialize the existing submodules so that the paths no longer cause conflicts when checking out other commits or branches:

```shell
git submodule deinit --all
```

Then perform the checkout to the new commit or branch:

```shell
git checkout main
```

Then update the submodule URLs with

```shell
git submodule sync --recursive
```

I added the `--recursive` option so that the same operation is also performed on submodules inside the submodules. Finally initialize all submodules in the proper location using the newly synced remote URLs:

```shell
git submodule update --init --recursive
```

This should fully restore the repository submodules to the same state as if the repository was newly cloned from the currently checked out commit.