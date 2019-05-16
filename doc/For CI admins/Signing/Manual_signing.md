# Manual procedure for signing vRouter driver

## Prerequisites

1.  One machine for HLK Server (preferably Windows Server 2016 with GUI)
1.  One machine for HLK Client (Windows Server 2016, GUI doesn't matter)

## Steps

1.  Build vRouter driver. Save `vrouter.sys`, `vrouter.inf`, `vrouter.cat` and `vrouter.pdb`.
1.  Sign `vrouter.cat` and `vrouter.sys` with EV code signing certificate.
1.  Add both machines to a workgroup.
1.  Install vRouter on the client.
1.  Follow all the steps from [this](https://docs.microsoft.com/en-us/windows-hardware/test/hlk/getstarted/step-1-install-controller-and-studio-on-the-test-server) documentation.

    - in step 5. search for `vrouter.sys` in `software device` tab
    - in step 6. there will probably be only 1 or 2 tests - that's ok
    - in step 8. create an unsigned `.hlkx` file

1.  Copy the unsigned `.hlkx` file to the signing machine, open it in HLK Studio and sign it.
1.  Go to [Microsoft Partner Center](https://partner.microsoft.com/en-us/dashboard/hardware/Search), upload the signed `.hlkx` file and wait for it to be signed by Microsoft (takes about 15 minutes).
1.  Download signed files.

