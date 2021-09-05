.. _package_repo:


Package recipe in the source repo
=================================

For creating packages built from your own code, it is usually more convenient to have the ``conanfile.py`` in the same repository as
the source code. There are two different approaches:


- Using the :ref:`exports sources attribute <exports_sources_attribute>` or method of the conanfile to
   export the source code together with the recipe. This way the recipe is self-contained and will
   not need to fetch the code from external origins when building from sources. It can be considered
   a "snapshot" of the source code.
-  Using the :ref:`scm attribute <scm_attribute>` of the conanfile to capture the remote and
   commit of your repository automatically.


.. important::

    This is a **tutorial** section. You are encouraged to execute these commands.
    For this concrete example, you will need **CMake** and **git** installed  in your path.
    It is not strictly required by Conan to create packages, you can use
    other build systems (as VS, Meson, Autotools and even your own) and SCM to do that, without any dependency
    to CMake or git.
    Some of the features used in this section are **experimental**, like ``CMakeToolchain`` or ``cmake_layout()``,
    and they might change in future releases. There are other alternative tools that are stable, please check
    the :ref:`reference section<references>` for more information.


Exporting the source with the recipe
------------------------------------

This is the case that we have seen in the previous "Getting started" section.
The local files and folders matching the patterns in ``exports_sources = "pattern1", "pattern2", ...`` are copied together with the ``conanfile.py``
recipe to the Conan cache, becoming part of the recipe. The sources are captured, and whenever this package needs to be built from sources, it can
use these sources, no extra work required.

If the attribute is not enough, there is an alternative method, that is more
powerful, in the previous example, the attribute can be replaced:

.. code-block:: bash

    $ conan new hello/0.1 -m=cmake_lib

Then replace the attribute with the following equivalent method:

.. code-block:: python

    # Remove the attribute
    # exports_sources = "src/*"

    def export_sources(self):
        self.copy("*", src="src", dst="src")

Finally, create the package to check that it keeps working:

.. code-block:: bash

    $ conan create . demo/testing
    hello/0.1@demo/testing: Calling export_sources()
    hello/0.1@demo/testing export_sources() method: Copied 1 '.txt' file: CMakeLists.txt
    hello/0.1@demo/testing export_sources() method: Copied 1 '.cpp' file: hello.cpp
    hello/0.1@demo/testing export_sources() method: Copied 1 '.h' file: hello.h
    ...
    hello/0.1: Hello World Release!


.. _scm_feature:

Capturing the SCM repository remote and commit
----------------------------------------------

If you want to avoid storing a copy of the source code inside the recipe, then
the recipe needs to capture the necessary information from the current SCM
repository in order to be able to automatically fetch the necessary source code
if a package consumer wants to rebuild the package from source later.
This capture can be done with the :ref:`scm attribute <scm_attribute>` feature. The recommended approach is to enable the capture of the SCM information in a
*conandata.yml* file that will be automatically exported with the *conanfile.py*. This is going to be the behavior in Conan 2.0 and not
using it is not stable. Please activate it with the following command (you
can also define in the *conan.conf* file directly):

.. code-block:: bash

    $ conan config set general.scm_to_conandata=1


Let's see how the ``scm`` feature works. First we will create a git repository, put our recipe
there using this ``scm`` feature, clone it and create the package:

.. code-block:: bash

    $ mkdir repo && cd repo
    $ conan new hello/0.1 -m=cmake_lib

Replace the ``exports_sources`` attribute with the following:

.. code-block:: python

    # Remove the attribute
    # exports_sources = "src/*"

    scm = {"type": "git",
           "url": "auto",
           "revision": "auto"}

Now, lets initialize the git repository, with a .gitignore, in a *repo" folder
emulating a Git server:

.. code-block:: bash

    $ echo test_package/build >> .gitignore
    $ git init .
    $ git add .
    $ git commit -m "initial commit"
    $ cd ..

We can clone now the repo, and create the package from sources, you should see at the beginning something like:

.. code-block:: bash

    $ git clone repo hello
    $ cd hello
    $ conan create . demo/testing
    hello/0.1@demo/testing: WARN: Repo origin looks like a local path: <yourpath>/repo
    hello/0.1@demo/testing: Repo origin deduced by 'auto': <yourpath>/repo
    hello/0.1@demo/testing: Revision deduced by 'auto': a1b4537314b70afc82d466bc3210e075911b4745
    hello/0.1@demo/testing: SCM: Getting sources from folder: <yourpath>/hello
    ...
    hello/0.1: Hello World Release!


Now the package can be built from source even in other computer, as long as it has access to the
remote git repository.

When doing a :command:`conan create` or :command:`conan export`, Conan will capture the sources of the local scm project folder in the local cache. This allows building packages making changes to the source code without the need of committing them and pushing them to the remote
repository.

So, if you are using the ``scm`` feature, with some ``auto`` field for `url` and/or `revision` and you
have uncommitted changes in your repository (go and do a small change in the package *hello.cpp* file, for example) a warning message will be printed:

.. code-block:: bash

    $ conan create . demo/testing
    hello/0.1@demo/testing: WARN: There are uncommitted changes,
    skipping the replacement of 'scm.url' and 'scm.revision' auto fields.
    Use --ignore-dirty to force it. The 'conan upload' command will
    prevent uploading recipes with 'auto' values in these fields.

As the warning message explains, the ``auto`` fields won't be replaced unless you specify ``--ignore-dirty``,
and by default, the :command:`conan upload` will block the upload of the recipe. This prevents recipes
to be uploaded with incorrect scm values exported.
You can use :command:`conan upload --force` to force uploading the recipe with the ``auto`` values un-replaced.
