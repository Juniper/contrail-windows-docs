# Capturing network traffic without Wireshark

## Preparing development machine

1. Install [Microsoft Network Monitor](https://blogs.technet.microsoft.com/netmon/p/downloads/).
2. Go to `Options > Parser Profiles`, select `Windows` and click `Set As Active`.

## Collecting traces

1. Open an elevated command prompt and run:

        netsh trace start persistent=yes capture=yes tracefile=.\trace.etl


2. Reproduce the issue.

3. Open an elevated command prompt and run:

        netsh trace stop

4. Copy trace.etl file to development machine and open with Network Monitor.

## Filtering

For filtering refer to [this](https://blogs.technet.microsoft.com/netmon/2006/10/17/intro-to-filtering-with-network-monitor-3-0/) and [this](https://blogs.technet.microsoft.com/rmilne/2016/08/11/network-monitor-filter-examples/).
