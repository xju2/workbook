# Athena development in a Fork

Fork the Athena repository and add ATLAS Athena as a remote repository. This will allow you to pull in changes from the main repository and keep your fork up to date.

```bash
git clone ssh://git@gitlab.cern.ch:7999/xju/athena.git
cd athena
git remote add atlas ssh://git@gitlab.cern.ch:7999/atlas/athena.git
git fetch atlas
git checkout -b atlas_main atlas/main
```
