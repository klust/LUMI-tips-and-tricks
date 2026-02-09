# Meeting with Kenneth

### 20251010

- upcoming Python 3.9 requirement is problematic
        - uv as escape hatch, not fully working as hoped

- extensions installed in EasyBuild itself (Rich, ...)
        - sanity checks caused trouble
        - embed $PYTHONPATH in eb script
        - control Python search path via sys.path in Python scripts
        - required because of not using system Python, and because of modules
coming from Cray PE not installed by EB
        - taking it a step further with EasyBuild 5.x, maybe running into
trouble because of that

- problem with installing wheels, can't unpack whl
        - PythonPackage doesn't unpack wheel file by default since EasyBuild v4.4.0
        - should work for both extensions & bundle components

- problems with Bundle easyblock = > fixed in EasyBuild v5.1.2

- `PythonPackage` too strict, causes trouble when used in LUMI context
        - being able to use `filter_env_vars` in easyconfig file could help
        - sanity checks often go wrong
        - => provide options to disable some checks
                - don't hard inject `--no-build-isolation` by adding support for
`disable_no_build_isolation = True`

- selectively skipping commands in sanity check
        - some easyblocks run shell commands directly into sanity_check_step
rather than letting them being overruled by sanity_check_commands
        - skipping entire sanity check in easyconfig: `skipsteps = ['sanity']`

- often using Python/SciPy-bundle from Cray PE (in which numpy is linked
to libsci)

- toolchains
        - need to be re-written for EaysBuild 5.x
          ```
         from easybuild.version import VERSION
          if VERSION > LooseVersion('4'):
                OPENMP_FLAG = 'fopenmp'
          else:
             OPENMP_FLAG = '-fopenmp'
         ```

- keep docs for older EasyBuild versions

- LUMI labels in easybuild-* repositories + grant Kurt triage rights

- patches that can be contributed => please do
        - leave it up to us to test in "standard" environment

- patch to accept other exceptions when trying to import keyring

- keeping easyblocks compatible with EasyBuild 4.x and 5.x
        - custom easyblocks
           ```
          from easybuild.version import VERSION
           if VERSION > LooseVersion('4'):
                    from easybuild.tools.run import run_cmd
          ```

- try to use hooks to "port" easyconfigs/easyblocks to EasyBuild 5.x
        - see EESSI hooks:
https://eur01.safelinks.protection.outlook.com/?url=https%3A%2F%2Fgithub.com%2FEESSI%2Fsoftware-layer-scripts%2Fblob%2Fmain%2Feb_hooks.py&data=05%7C02%7CKurt.Lust%40uantwerpen.be%7C945138b11b16491f837408de07d74b6f%7C792e08fb2d544a8eaf72202548136ef6%7C0%7C0%7C638956818446859963%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=vW4HSa12lzKs6igldRykXPXPoizglQNjK1UwtPPksTE%3D&reserved=0

- EasyBuild hackathon for LUMI in Antwerp/Brussels (early 2026)

- user-focused EasyBuild training (for LUMI)
        - maybe in combo with EESSI/EFP?



