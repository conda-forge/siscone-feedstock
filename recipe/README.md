# SISCone packaging on conda-forge

This feedstock supplies two related packages:

- **`siscone`** – the runtime shared libraries `libsiscone` and `libsiscone_spherical`
  (`siscone.dll` and `siscone_spherical.dll` on Windows). Nothing else: no headers, no CMake files.
- **`siscone-devel`** – everything needed to compile and link against SISCone: the headers under
  `include/siscone/`, the CMake package configuration (`find_package(siscone)`, targets
  `siscone::siscone` and `siscone::siscone_spherical`), and the import libraries on Windows. It
  depends on the exactly matching `siscone` build.

Both carry `run_exports` on `siscone` (`x.x`), so anything built against them gets the runtime
package as a run dependency automatically.

## How recipes use these packages

### A. The recipe compiles or links against SISCone

Put `siscone-devel` in `host`. The runtime dependency on `siscone` comes from its `run_exports`;
do not list it in `run`.

```yaml
# recipe.yaml (excerpt)
requirements:
  host:
    - siscone-devel
```

### B. The recipe's installed headers or CMake configuration reference SISCone

If consumers of your package need SISCone's headers or CMake configuration, put `siscone-devel` in
the `run` requirements of the output that ships those files, usually your own `*-devel` output, in
addition to `host`. A typical case is an installed `fooConfig.cmake` that calls
`find_package(siscone)` or links `siscone::siscone`. FastJet is such a case: its
`fastjetConfig.cmake` calls `find_package(siscone REQUIRED)`.

```yaml
# recipe.yaml (excerpt)
outputs:
  - package:
      name: libfoo-devel
    requirements:
      host:
        - siscone-devel
      run:
        - siscone-devel
```

### C. The package only runs software that was built against SISCone

Nothing to do: the `run_exports` above already pull in `siscone`.

### D. Interactive development

To compile your own code against SISCone in an environment, install `siscone-devel` (plus a
compiler, e.g. `cxx-compiler`).

## Further details

- Before the split (`siscone` 3.1.3 build 0 and earlier), `siscone` held the libraries, the
  headers and the CMake files together. From build 1 on, the headers and CMake files are only in
  `siscone-devel`. A recipe that kept `siscone` in `host` now fails to find SISCone's headers or
  `find_package(siscone)`: switch it to `siscone-devel` (case A).
- The packages are built from the same source with the same options. The two outputs only divide
  the installed files between them.
- See conda-forge/siscone-feedstock#26 for the split, and conda-forge/fastjet-cxx-feedstock#27 for
  the same split of FastJet.
