.. _external_repo:

Getting the package sources from another origin
-----------------------------------------------

The recipe ``source()`` is intended to implement functionality related to
retrieving the source code necessary to build a package from source.


.. code-block:: python

    from conans import ConanFile

    class HelloConan(ConanFile):
        ...

        def source(self):
            self.run("git clone https://github.com/conan-io/hello.git src")
            ...

You can also use the :ref:`tools.Git <tools_git>` class:

.. code-block:: python

    from conans import ConanFile, tools

    class HelloConan(ConanFile):
        ...

        def source(self):
            git = tools.Git(folder="hello")
            git.clone("https://github.com/conan-io/hello.git", "master")
            ...


.. warning::

    The ``source()`` method must be independent of any setting, option and configuration.
    It is intended to execute only once, for all different build configurations, so the
    operations done in ``source()`` must be common to all different builds and configurations.
    If you have something that is specific of one configuration, do that operation in the
    ``build()`` method, not in ``source()``.


The ``source()`` method can be parameterized using the ``conandata.yml`` feature.
This file is a yaml file that is automatically exported an

TODO CONANDATA:YML