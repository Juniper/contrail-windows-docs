# Run gerrit pipeline on github PR

When to do it:

You introduce a change to github/gerrit handling code, e.g. the
trigger mechanism - code that depends on Zuul or other Jenkins trigger plugins

## 

1. Clone Jenkins job `winci-server2016-prod` to some temporary `tmp-devel-gerrit-check`
2. Configure `tmp-devel-gerrit-check`:
* Properties content: BRANCH_NAME={branch to test}
* Branches to build: {branch to test}
3. Schedule `tmp-devel-gerrit-check` job manually with parameters:
* `ZUUL_PROJECT=Juniper/contrail-controller`
* `ZUUL_BRANCH=master`
* `ZUUL_REF=None`
* `ZUUL_URL=http://10.84.12.75/merge-warrior`
* `ZUUL_UUID= (some random UUID - it will determine path on the logserver)`
* `ZUUL_CHANGE= (can be left empty)`
* `ZUUL_PATCHSET= (can be left empty)`
4. Verify that it passes
5. (optional, recommended) Paste a link to successful build logs under your PR on github for the reviewers to verify
6. Remove temporary `tmp-devel-gerrit-check` job
