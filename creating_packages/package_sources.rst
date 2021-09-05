Package source code
===================

There are different approaches for obtaining the source code necessary to build
a package:

- Having the ``conanfile.py`` recipe in the same repository as the source code. This approach
  works generally better when packaging your own code. Within this approach, there are also two
  alternatives:

  - Use the :ref:`exports sources attribute <exports_sources_attribute>` or ``exports_sources()`` method
    to capture the necessary source code to build the package
    into the recipe itself, together with the ``conanfile.py``. This is the approach we have used
    in the previous section.
  - Use the :ref:`scm attribute <scm_attribute>` to capture the coordinates (URL of the remote repo, and revision or
    commit) of the current repository.

- Having the ``conanfile.py`` recipe in a different repository than the source code. This approach
  could be better for packaging third party and open source libraries. For managing multiple external
  libraries, it could be simpler to put all the recipes for those libraries in a single repository. This is the approach that is
  implemented in the `conan-center-index repository <https://github.com/conan-io/conan-center-index/tree/master/recipes>`_.


.. toctree::
   :maxdepth: 2

   package_repo
   external_repo





