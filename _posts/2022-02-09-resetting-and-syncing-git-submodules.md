---
layout: post
title:  "Resetting and Syncing Git Submodules"
date:   2022-02-09 15:45:00 +0100
categories: [development]
---

Or: "I don't care about the submodules. Just update the submodules and reset all of them to master!"

## TL;DR

```shell
git submodule deinit --all
git checkout <new-branch>
git submodule init
git submodule sync --recursive
git submodule update --recursive
```

## Background

This is just a short note for a particular nuisance when working with git and submodules. I am working on a rather large open-source project. The main git repository contains most of the code, but also pulls in many dependencies as submodules. From time to time I ran into issues with the submodules, when the submodules were moved around or when the remote URLs of the submodules changed. This often shows as an error where git can no longer find a certain commit:

Problems with submodules:

- When submodules had their remote URLs changed, but the change was not been synced locally yet, git will fail to find newer commits:
	```
	$ git submodule update --init --recursive

	fatal: remote error: upload-pack: not our ref 9386b9f73522fefb2ccbe0ce1ccdc681f5f4a86f
	fatal: the remote end hung up unexpectedly
	Fetched in submodule path '<path-to-submodule>', but it did not contain 9386b9f73522fefb2ccbe0ce1ccdc681f5f4a86f. Direct fetching of that commit failed.
	```


- Likewise when the paths to some submodules were modified or renamed, git will often make it impossible to move between commits with the following error:

	```
	$ git checkout <some-branch>
	error: The following untracked working tree files would be overwritten by checkout:
		<path-to-submodule/path-to-file1>
		<path-to-submodule/path-to-file2>
		<path-to-submodule/path-to-file3>
		...
	```

## The hammer method

The solution is the following: First deinitialize the existing submodules so that the paths no longer cause conflicts when checking out other commits or branches:

```shell
# Deinitialize all submodules so we can checkout other branches
git submodule deinit --all
```

Then perform move to the new commit or branch, which was previously not possible:

```shell
git checkout <new-branch>
```

Then initialize the modules without updating them yet
```shell
git submodule init
```

Then update the submodule remote URLs with

```shell
git submodule sync --recursive
```

I added the `--recursive` option here so that the same operation is also performed on submodules inside the submodules. Finally update all submodules in the proper location using the newly synced remote URLs:

```shell
git submodule update --recursive
```

This should fully restore the repository submodules to the same state as if the repository was newly cloned from the currently checked out commit.
