Omarchy Silicon downstream
==========================

This is the Omarchy Silicon downstream Mesa repository for Apple Silicon
graphics work. It targets all present and future Apple M-series chips and every
materially distinct board or hardware profile. Development is active: no Apple
Silicon board is currently supported or qualified, and no supported release is
shipped.

The upstream Mesa build, support, bug-report, and GitLab contribution
instructions below are inherited context. They apply to upstream Mesa and its
GitLab or mirror workflow, not to Omarchy Silicon support, installation, or
release claims.

For Omarchy Silicon project routes:

* `Issues <https://github.com/omarchy-silicon/omarchy-apple-platform/issues>`_
* `Discussions <https://github.com/omarchy-silicon/omarchy-apple-platform/discussions>`_
* `Private security advisory <https://github.com/omarchy-silicon/omarchy-apple-platform/security/advisories/new>`_

Upstream Mesa attribution and licensing remain in place below and in the
repository's existing license notices.

`Mesa <https://mesa3d.org>`_ - The 3D Graphics Library
======================================================


Source
------

The upstream Mesa source lives at https://gitlab.freedesktop.org/mesa/mesa.
This checkout is an Omarchy Silicon downstream; the upstream source and its
mirrors are referenced for Mesa context and do not establish Omarchy Silicon
support.


Upstream Mesa build & install
-----------------------------

You can find more information in our documentation (`docs/install.rst
<https://docs.mesa3d.org/install.html>`_), but the recommended way is to use
Meson (`docs/meson.rst <https://docs.mesa3d.org/meson.html>`_):

.. code-block:: sh

  $ meson setup build
  $ ninja -C build/
  $ sudo ninja -C build/ install

Upstream Mesa support
---------------------

Many Mesa devs hang on IRC; if you're not sure which channel is
appropriate, you should ask your question on `OFTC's #dri-devel
<irc://irc.oftc.net/dri-devel>`_, someone will redirect you if
necessary.
Remember that not everyone is in the same timezone as you, so it might
take a while before someone qualified sees your question.
To figure out who you're talking to, or which nick to ping for your
question, check out `Who's Who on IRC
<https://dri.freedesktop.org/wiki/WhosWho/>`_.

The next best option is to ask your question in an email to the
mailing lists: `mesa-dev\@lists.freedesktop.org
<https://lists.freedesktop.org/mailman/listinfo/mesa-dev>`_


Upstream Mesa bug reports
-------------------------

If you think something isn't working properly, please file a bug report
(`docs/bugs.rst <https://docs.mesa3d.org/bugs.html>`_).


Upstream Mesa contributing
--------------------------

Contributions are welcome, and step-by-step instructions can be found in our
documentation (`docs/submittingpatches.rst
<https://docs.mesa3d.org/submittingpatches.html>`_).

Note that Mesa uses gitlab for patches submission, review and discussions.
