.. _mozreview_user:

====================
MozReview User Guide
====================

This article is a guide on conducting code review at Mozilla using MozReview,
a repository-based code-review system based on
`Review Board <https://www.reviewboard.org/>`_.

For the quick and impatient who just want to look at the web interface,
it lives at https://reviewboard.mozilla.org/. Log in via Bugzilla. Read
on to learn how to create new review requests and to conduct code review
using the web interface.

Before you start code review, you need some code to review. This article
assumes you have at least basic knowledge of version control and can
create commits that should be reviewed.

Please drill down into one of the following sections to continue.

.. toctree::
   :maxdepth: 2

   mozreview/install
   mozreview/commits
   mozreview/reviewboard
   mozreview/bugzilla
   mozreview/autoland

Filing Bugs
===========

Did you find a bug in MozReview? Do you have a feature request to make
it better? `File a bug <https://bugzilla.mozilla.org/enter_bug.cgi?product=MozReview&component=General>`_
in the ``MozReview :: General`` component.

.. tip::

   We like bug reports that contain command output!

   If you see an exception, stack trace, or error message, copy it into
   the bug.

   The tests for MozReview are implemented as a series of user-facing
   commands, simulating terminal interaction. If you give us the
   commands you used to cause the error, there's a good chance we can
   reproduce it and add a test case so it doesn't break.

.. _mozreview_getting_in_touch:

Getting in Touch
================

Have feedback or questions that aren't appropriate for bugs? Get in
touch with us!

If you prefer IRC, join ``#mozreview`` ``irc.mozilla.org``.

If you prefer email or newsgroups, please use ``mozilla.dev.version-control``,
which is available via
`mailing list <https://lists.mozilla.org/listinfo/dev-version-control>`_,
`Google Group <https://groups.google.com/forum/#!forum/mozilla.dev.version-control>`_,
or `NNTP <news://news.mozilla.org:119/mozilla.dev.version-control>`_.
This is also our general development mailing list.

Adding Review Repositories
==========================

Since code review is initiated by pushing to a repository, every
repository must have a corresponding code review repository configured
to receive reviews.

To request creation of a new code review repository,
`file a bug <https://bugzilla.mozilla.org/enter_bug.cgi?product=MozReview&component=General>`_
with the source repository URL and your request will likely be granted.
