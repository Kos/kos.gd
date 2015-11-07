<!--
.. title: Git adventures: Loose object is corrupted
.. slug: git-adventures-loose-object-is-corrupted
.. date: 2013-02-01 11:55:20 UTC
.. tags: git
.. category: dev
.. link:
.. description:
.. type: text
-->

I couldn't read my Git history today because I ran into this problem:

	$ git log --oneline
	f1330fd Basic automation
	7234c6e Textures work
	3b18493 Fix combo defaults
	error: inflate: data stream error (incorrect data check)
	fatal: loose object 65d626bb82c8996f8fc5f659f7c207fee1d74948 (stored in .git/objects/65/d626bb82c8996f8fc5f659f7c207fee1d74948) is corrupted

The same message appeared when checking the repository with `git fsck`.

My working copy and dozens of last commits were intact; this error showed up when trying to view more history or when trying to clone a repo. Some googling revealed that I should get the original (non-corrupted) object from somewhere and replace it.

<!--more-->

I removed the corrupted object:

	mv .git/objects/65/d626bb82c8996f8fc5f659f7c207fee1d74948 someplace/away

Just this allowed `git fsck` to complete and reveal, among the usual dangling objects, this message:

    missing commit 65d626bb82c8996f8fc5f659f7c207fee1d74948

Oh, so it's a commit! That makes sense. I plugged in a pendrive with a backup (which was another bare repository) and made a fresh clone. There, running `git show` worked there - that's some good news, because the thing that got corrupted is in my backup.

	$ git show 65d626bb82c8996f8fc5f659f7c207fee1d74948
	commit 65d626bb82c8996f8fc5f659f7c207fee1d74948
	Author: Tomasz Wesolowski <kosashi@gmail.com>
	Date:   Sat Jan 12 13:17:12 2013 +0100

	    Editor improvements

	(the diff followed)

My first idea was to add the broken repository as a remote, fetch new commits from it and continue the work from there. This had failed initially (before removing the corrupted object):

	$ git remote add broken d:/werk/mgr.broken
	$ git fetch broken
	error: inflate: data stream error (incorrect data check)
	fatal: loose object 65d626bb82c8996f8fc5f659f7c207fee1d74948 (stored in ./objects/65/d626bb82c8996f8fc5f659f7c207fee1d74948) is corrupt
	fatal: The remote end hung up unexpectedly

No success; However I later noticed that simply removing the corrupted object file made the fetch work.

I decided to simply copy the uncorrupted object file from the backup's clone to my original repository. This worked just as well. (By the way: If you can't find the object in `.git/objects/` by its name, it probably has been [packed][pack] to conserve space.)

Why exactly had this happened? I have no idea, but my PC has been surprising me in many funny ways recently, so for now I'm just grateful I make it boot every day. :)

Some reading that helped me:

- [Linus Torvalds: Some tricks to reconstruct blob objects in order to fix a corrupted repository](http://git.kernel.org/?p=git/git.git;a=blob;f=Documentation/howto/recover-corrupted-blob-object.txt;h=323b513ed0e0ce8b749672f589a375073a050b97;hb=HEAD) (not exactly my problem but interesting)
- [StackOverflow: Git "corrupt loose object"](http://stackoverflow.com/q/4254389/399317)
