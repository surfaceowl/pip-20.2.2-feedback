# pip-20.2.2-feedback
feedback for pip resolution

**Problem:**  pip does not correctly resolve dependency versions from pypi packages in at least one case.  Specifically, when installing only moto and 
idna together - pip misinterprets the upper end of the version range.  In this case, the dependency requirements are pinned in setup.py, rather than requirements.txt.

**Observations**
-- default resolver reports two errors; while 2020-resolver reports only one (idna)
-- idna fails with both resolvers, because git reports moto requires `idna<2.9,>=2.5,`... but it does not - it requires "idna<3,>=2.5"
-- moto github repo (https://github.com/spulec/moto/blob/master/setup.py) lists "idna<3,>=2.5",
-- it is not clear if this is a problem with pip, or a problem with the way data is being fed to pip in the package metadata.

**steps to recreate**
**1- setup current environment**: ubuntu20.04; python3.8.3; running in virtualenv named `venv` created by:  python3 -m virtualenv venv


**2- pip list: results of pip list**
Package    Version
---------- -------
pip        20.2.2
setuptools 49.6.0
wheel      0.35.1


** remember convenience script to pip uninstall all pip install packages:  pip freeze | xargs pip uninstall -y

**3- create requirements.txt with only two pinned entries:**
idna==2.10
moto==1.3.14


**4-install requirements.txt with default resolver**
python3 -m pip install -r requirements.txt
```
ERROR: After October 2020 you may experience errors when installing or updating packages. This is because pip will change the way that it resolves dependency conflicts.

We recommend you use --use-feature=2020-resolver to test your packages with the new resolver before it becomes the default.

python-jose 3.2.0 requires ecdsa<0.15, but you'll have ecdsa 0.15 which is incompatible.
moto 1.3.14 requires idna<2.9,>=2.5, but you'll have idna 2.10 which is incompatible.
```

**5- install requirements.txt with --use-feature=202-resolver**
python3 -m pip install -r requirements.txt --use-feature=202-resolver
```
ERROR: Cannot install idna==2.10 and moto 1.3.14 because these package versions have conflicting dependencies.

The conflict is caused by:
    The user requested idna==2.10
    moto 1.3.14 depends on idna<2.9 and >=2.5

To fix this you could try to:
1. loosen the range of package versions you've specified
2. remove package versions to allow pip attempt to solve the dependency conflict

ERROR: ResolutionImpossible: for help visit https://pip.pypa.io/en/latest/user_guide/#fixing-conflicting-dependencies
```

**6- check setup that should work correctly**
use backup requirements.txt file with lower version of idna (below <2.9 as reported by pip errors above)
add new requirement to fix `ecdsa` dependency error with default resolver
ecdsa==0.14
idna==2.8
moto==1.3.14

```run pip cleanup script:  pip freeze | xargs pip uninstall -y```
run python3 -m pip install -r requirements2.8.txt  (works correctly with both resolvers)
