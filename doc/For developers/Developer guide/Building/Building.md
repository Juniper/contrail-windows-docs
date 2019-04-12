# Building Tungsten Fabric for Windows

## Setting up repositories

    git clone https://review.opencontrail.org/Juniper/contrail-api-client src\contrail-api-client
    git clone https://review.opencontrail.org/Juniper/contrail-build tools\build
    git clone https://review.opencontrail.org/Juniper/contrail-common src\contrail-common
    git clone https://review.opencontrail.org/Juniper/contrail-controller controller
    git clone https://review.opencontrail.org/Juniper/contrail-third-party third_party
    git clone https://review.opencontrail.org/Juniper/contrail-vrouter vrouter
    git clone https://review.opencontrail.org/Juniper/contrail-windows windows
    cp tools/build/SConstruct ./
     
    cd third_party
    py fetch_packages.py

## Compiling

For vRouter:
    scons vrouter

For vRouter Agent:
    scons -j <nr of threads> contrail-vrouter-agent.msi

For Node Manager:
    $packages = @(
        "database:node_mgr",
        "build/debug/sandesh/common/dist",
        "sandesh/library/python:pysandesh",
        "vrouter:node_mgr",
        "contrail-nodemgr"
    )
    scons -j <nr of threads> @packages

## Known issues

1. Issue with `wdf`
Solution: delete folder `C:\Program Files (x86)\Windows Kits\10\Include\wdf`
Read more at https://community.osr.com/discussion/270106

2. Issue with `Microsoft.Build.Tasks.v12.0.dll`
Solution:
    cd $EWDKPath\Program Files\MSBuild\14.0\Bin\amd64
    cp Microsoft.Build.Tasks.Core.dll Microsoft.Build.Tasks.v12.0.dll
