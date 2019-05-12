# Building Tungsten Fabric for Windows

## Dependencies

Machine with Windows 10 and installed updates is required.

1. Enable .Net Framework 3.5 Windows Feature:

       Enable-WindowsOptionalFeature -Online -FeatureName NetFX3 -All

1. Install VS2015 (any version).
1. Install Windows [SKD](https://go.microsoft.com/fwlink/p/?LinkID=845298) and [WDK](https://go.microsoft.com/fwlink/p/?LinkID=845980) version 15063 (for Windows 10 version 1703).
1. Create environment variable `MSBUILD` pointing to `MSBuild.exe`, eg. `C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe`
1. Install Chocolatey:

       Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

1. Install Pythons and required libraries:

       choco install python3
       py -3 -m pip install wheel
       py -3 -m pip install scons
       choco install python2 --version=2.7.13
       py -2 -m pip install lxml

1. Install Git:

       choco install git

1. Install GNU utilities:

       choco install gnuwin32-coreutils.install
       choco install patch
       choco install make
       choco install vim
       choco install winflexbison3
       Copy-Item C:\ProgramData\chocolatey\bin\win_flex.exe C:\ProgramData\chocolatey\bin\flex.exe
       Copy-Item C:\ProgramData\chocolatey\bin\win_bison.exe C:\ProgramData\chocolatey\bin\bison.exe

1. Add GNU utils directory to `PATH` env var, eg. `C:\Program Files (x86)\GnuWin32\bin`
1. Install CMake

       choco install CMake --install-arguments='ADD_CMAKE_TO_PATH=System'

1. Install WiX

       choco install wixtoolset

1. Add WiX install directory to `PATH` env var, eg. `C:\Program Files (x86)\WiX Toolset v3.11\bin\`
1. Install Boost

       choco install boost-msvc-14 --source="'https://www.myget.org/F/contrail-windows-builder/'" --version=1.62.0

1. Add Boost directory to `INCLUDE` env var (create this variable if it doesn't exist), eg. `C:\local\boost_1_62`
1. Create env var `BOOST_ROOT` pointing to Boost directory, eg. `C:\local\boost_1_62`

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

    Solution:

        Remove-Item -Recurse -Force "C:\Program Files (x86)\Windows Kits\10\Include\wdf"

    Read more [here](https://community.osr.com/discussion/270106).

2. Issue with `Microsoft.Build.Tasks.v12.0.dll`

    Solution:

        cd $EWDKPath\Program Files\MSBuild\14.0\Bin\amd64
        cp Microsoft.Build.Tasks.Core.dll Microsoft.Build.Tasks.v12.0.dll
