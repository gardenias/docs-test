.. _hgmozilla_bookmarks:

===============
Using Bookmarks
===============

The Mercurial project recommends the use of bookmarks for doing
development.

At its core, bookmarks are a labeling mechanism. Instead of a
numeric revision ID or alphanumeric SHA-1 (fragments), bookmarks
provide human-friendly identifiers to track and find changesets
or lines of work.

.. tip::

   If you are a Git user, bookmarks are similar to Git branches.
   Although they don't behave exactly the same.

Bookmarks and Feature Development
=================================

Bookmarks are commonly used to track the development of something -
a *feature* in version control parlance. The workflow is typically:

1. Create a bookmark to track a feature
2. Commit changes for that feature against that bookmark
3. Land the changes

Bookmarks typically exist from the time you start working on a feature
to the point that feature lands, at which time you delete the bookmark,
for it is no longer necessary.

Creating and Managing Bookmarks
===============================

Numerous guides exist for using bookmarks. We will not make an attempt
at reproducing their work here.

Recommending reading for using bookmarks includes:

* `The official Mercurial wiki <https://www.mercurial-scm.org/wiki/Bookmarks>`_
* `Bookmarks Kick Start Guide <http://mercurial.aragost.com/kick-start/en/bookmarks/>`_
* `A Guide to Branching in Mercurial <http://stevelosh.com/blog/2009/08/a-guide-to-branching-in-mercurial/#branching-with-bookmarks>`_

The following sections will expand upon these guides.

Getting the Most out of Bookmarks
=================================

Use Mercurial 3.2 or Newer
--------------------------

Mercurial 3.2 adds notification messages when entering or leaving
bookmarks. These messages increase awareness for when bookmarks are
active.

Integrate the Active Bookmark into the Shell Prompt
---------------------------------------------------

If you find yourself forgetting which bookmark is active and you
want a constant reminder, consider printing the active bookmark as
part of your shell prompt. To do this, use the
`prompt extension <https://www.mercurial-scm.org/wiki/PromptExtension>`_
or the *scm-prompt.sh* script from Facebook's
`hg-experimental repository <https://bitbucket.org/facebook/hg-experimental>`_.

Sharing Changesets
==================

Once you have a changeset (or several!) that you'd like to get checked into
a Mozilla repository, you'll need to share them with others in order to get
them reviewed and landed.

When working with mozilla-central, pushing your changesets to MozReview is
the primary method of sharing. See the
:ref:`MozReview documentation <mozreview>` for more information.

Collaborating / Sharing Bookmarks
=================================

Say you have multiple machines and you wish to keep your bookmarks in
sync across all of them. Or, say you want to publish a bookmark
somewhere for others to pull from. For these use cases, you'll need a
server accessible to all parties to push and pull from.

If you have Mozilla commit access, you can
`create a user repository <https://developer.mozilla.org/en-US/docs/Creating_Mercurial_User_Repositories>`_
to hold your bookmarks.

If you don't have Mozilla commit access or don't want to use a user
repository, you can create a repository on Bitbucket.

.. warning::

   The Firefox repository may be larger than what Bitbucket allows you to
   store. If you want to share bookmarks for the Firefox repository,
   a user repository is your best bet.

If neither of these options work for you, you can run your own Mercurial server.

Pushing and Pulling Bookmarks
-----------------------------

``hg push`` by default won't transfer bookmark updates. Instead, you
need to use the ``-B`` argument to tell Mercurial to push a bookmark
update. e.g.::

   $ hg push -B my-bookmark user
   pushing to user
   searching for changes
   remote: adding changesets
   remote: adding manifests
   remote: adding file changes
   remote: added 1 changesets with 1 changes to 1 files
   exporting bookmark my-bookmark

.. tip::

   When pushing bookmarks, it is sufficient to use ``-B`` instead of
   ``-r``.

   When using ``hg push``, it is a common practice to specify ``-r
   <rev>`` to indicate which local changes you wish to push to the
   remote. When pushing bookmarks, ``-B <bookmark>`` implies
   ``-r <bookmark>``, so you don't need to specify ``-r <rev>``.

Unlike ``hg push``, ``hg pull`` will pull all bookmark updates
automatically. If a bookmark has been added or updated since the last
time you pulled, ``hg pull`` will tell you so. e.g.::

   $ hg pull user
   pulling from user
   pulling from $TESTTMP/a (glob)
   searching for changes
   adding changesets
   adding manifests
   adding file changes
   added 1 changesets with 1 changes to 1 files (+1 heads)
   updating bookmark my-bookmark

Things to Watch Out For
-----------------------

Mercurial repositories are publishing by default. If you push to a
publishing repository, your Mercurial client won't let you modify
pushed changesets.

As of February 2015, user repository on hg.mozilla.org are
non-publishing by default, so you don't have to worry about this.
However, if you use a 3rd party hosting service, this could be
a problem. Some providers have an option to mark repositories as
non-publishing. This includes Bitbucket. **If you plan on sharing
bookmarks and rewriting history, be sure you are using a non-publishing
repository.**
