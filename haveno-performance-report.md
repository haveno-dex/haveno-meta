# Table of Contents

- Overview
- Run Configuration
- Performance without DAO
- Performance with DAO
- Conclusion
- References

# Overview

In this report, we'll try to find out top time consumers for Haveno's java backends. Namely, we'll try to monitor cpu
usage in the seed node and 3 app instances(that is, Alice, Bob, arbitrator)with and without DAO. The main idea is
sampling cpu usage by VisualVM and take flight recording by JDK mission control at the same time for each java backend.
Then get top 3 consumer threads according to cpu consumption percentage in VisualVM's thread cpu time tab. And get top
time consumer method in JDK mission control's flame graph. But we won't try to find top time consumer methods for JavaFX
related threads since Haveno won't use JavaFX as frontend.

# Run Configuration

We test the backends performance in 2 rounds for code base before and after removing DAO. Each test will monitor cpu
usage of the backends in idle and trading. \
For idle case, after frontend apps get started at first time, we accept all confirmation popups in the 3 app instances
and keep them untouch for about 10 minutes. \
For trading case, we keep making trades between Alice and Bob for about 10 minutes(5 trades done).\
Here are common setup steps:

1. Clean app data by deleting following directories if exist:\
   ~/.local/share/haveno-XMR_STAGENET_Alice\
   ~/.local/share/haveno-XMR_STAGENET_arbitrator\
   ~/.local/share/haveno-XMR_STAGENET_Bob\
   ~/.local/share/haveno-XMR_STAGENET_Seed_2002
2. Clone the Haveno repository and checkout
   the [commit before removing DAO](https://github.com/haveno-dex/haveno/commit/f9f2cd07c3867c225c83ea7ee9ae1e87cf6d5e93)
   or [commit after removing DAO](https://github.com/haveno-dex/haveno/commit/cefba8e4b598a7eabb524491042b685397667cca).
3. Start a local Haveno test network from this link https://github.com/haveno-dex/haveno/blob/master/docs/installing.md

# Performance without DAO

## Top Time Consumers in Idle

|App Name                        |        Top 3 Consumer Thread   |Thread Time(CPU)    |Top Consumer Method   |
|------------                    | -------------                  | -------------      |----------|
| haveno-XMR_STAGENET_Alice      |JavaFX Application Thread          |30,030 ms (33%)     |   Ignore      |
|                                |RMI TCP Connection(7)-127.0.0.1 |19,607 ms (21.5%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |InvokeLaterDispatcher              |16,985 ms (18.7%)   | Unsafe.park(boolean, long) |
|  haveno-XMR_STAGENET_Bob       |JavaFX Application Thread          |30,001 ms (31%)     | Ignore   |
|                                |RMI TCP Connection(idle)          |26,105 ms (27%)     | Object.wait(long)|
|                                |InvokeLaterDispatcher              |16,822 ms (17.4%)   |  Unsafe.park(boolean, long)|
| haveno-XMR_STAGENET_arbitrator |JavaFX Application Thread          |30,602 ms (24.9%)   | Ignore|
|                                |RMI TCP Connection(idle)          |30,129 ms (24.5%)   |Object.wait(long)|
|                                |RMI TCP Connection(6)-127.0.0.1 |23,417 ms (19.1%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|haveno-XMR_STAGENET_Seed_2002   |RMI TCP Connection(5)-127.0.0.1 |26,565 ms (77.8%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |main                            |    2,288 ms (6.7%)    |Not found|
|                                |SeedNodeMain                    |    2,074 ms (6.1%)    |Unsafe.park(boolean, long)|

## Top Time Consumers in Trading

|App Name                        |        Top 3 Consumer Thread      |Thread Time(CPU)    |Top Consumer Method   |
|------------                    | -------------                     | -------------      |----------|
| haveno-XMR_STAGENET_Alice      |QuantumRenderer-0                  |    141,438 ms (57.3%) |Ignore|
|                                |JavaFX Application Thread          |    49,319 ms (20%)    |Ignore|
|                                |InvokeLaterDispatcher                 |14,260 ms (5.8%)    |Unsafe.park(boolean, long)|
|  haveno-XMR_STAGENET_Bob       |QuantumRenderer-0                     |170,519 ms (54.5%)  |Ignore|
|                                |JavaFX Application Thread             |65,869 ms (21.1%)   |Ignore|
|                                |RMI TCP Connection(22)-127.0.0.1     |14,815 ms (4.7%)    |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|haveno-XMR_STAGENET_arbitrator  |JavaFX Application Thread             |34,027 ms (26.9%)   |Ignore|
|                                |QuantumRenderer-0                     |28,410 ms (22.5%)   |Ignore|
|                                |RMI TCP Connection(13)-127.0.0.1     |23,133 ms (18.3%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|haveno-XMR_STAGENET_Seed_2002   |RMI TCP Connection(11)-127.0.0.1     |16,249 ms (50.6%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |JFR Periodic Tasks                 |2,936 ms (9.2%)      |Ignore instrument thread|
|                                |RMI TCP Connection(10)-127.0.0.1     |2,579 ms (8%)       |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|

# Performance with DAO

## Top Time Consumers in Idle

|App Name                        |        Top 3 Consumer Thread      |Thread Time(CPU)    |Top Consumer Method   |
|------------                    | -------------                     | -------------      |----------|
|haveno-XMR_STAGENET_Alice       |JavaFX Application Thread             |32,949 ms (35.8%)   |Ignore|
|                                |InvokeLaterDispatcher                 |15,834 ms (17.2%)   |Unsafe.park(boolean, long)|
|                                |QuantumRenderer-0                     |14,577 ms (15.9%)   |Ignore|
|haveno-XMR_STAGENET_Bob         |JavaFX Application Thread             |32,373 ms (33.3%)   |Ignore|
|                                |RMI TCP Connection(11)-127.0.0.1     |19,912 ms (20.5%)   |TreeMap.put(Object, Object)|
|                                |InvokeLaterDispatcher                 |15,408 ms (15.9%)   |Unsafe.park(boolean, long)|
|haveno-XMR_STAGENET_arbitrator  |JavaFX Application Thread             |33,196 ms (30.3%)   |Ignore|
|                                |QuantumRenderer-0                     |17,363 ms (15.8%)   |Ignore|
|                                |InvokeLaterDispatcher                 |15,712 ms (14.3%)   |Unsafe.park(boolean, long)|
|haveno-XMR_STAGENET_Seed_2002   |RMI TCP Connection(12)-127.0.0.1     |18,482 ms (37.6%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |RMI TCP Connection(11)-127.0.0.1     |14,943 ms (30.4%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |JFR Periodic Tasks                 |3,571 ms (7.3%)     |Ignore instrument thread|

## Top Time Consumers in Trading

|App Name | Top 3 Consumer Thread |Thread Time(CPU)    |Top Consumer Method |
|------------                    | -------------                     | -------------      |----------|
|haveno-XMR_STAGENET_Alice       |QuantumRenderer-0                     |79,089 ms (45.5%)   |Ignore|
|                                |JavaFX Application Thread             |34,250 ms (19.7%)   |Ignore|
|                                |RMI TCP Connection(idle)           |    19,084 ms (11%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|haveno-XMR_STAGENET_Bob         |QuantumRenderer-0                     |104,185 ms (53.8%)  |Ignore|
|                                |JavaFX Application Thread             |55,770 ms (28.8%)   |Ignore|
|                                |InvokeLaterDispatcher                 |9,445 ms (4.9%)     |Unsafe.park(boolean, long)|
|haveno-XMR_STAGENET_arbitrator  |JavaFX Application Thread             |22,089 ms (27.4%)   |Ignore|
|                                |RMI TCP Connection(15)-127.0.0.1     |19,837 ms (24.6%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |RMI TCP Connection(12)-127.0.0.1     |15,780 ms (19.6%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|haveno-XMR_STAGENET_Seed_2002   |RMI TCP Connection(10)-127.0.0.1     |15,583 ms (56.8%)   |SocketInputStream.socketRead0(FileDescriptor, byte, int, int, int)|
|                                |JFR Periodic Tasks                 |2,748 ms (10%)      |Ignore instrument thread|
|                                |main                                 |2,731 ms (10%)      |Not found|

# Conclusion

According to above stats, we can see Haveno's backends cpu consumption is dominated by JavaFX and network.\
Especially for JavaFX which is the frontend, freeze a lot when I'm doing trades.

# References

## Method Used to Collect CPU Time of Threads

To sample cpu time, we use VisualVM which can be obtained from https://visualvm.github.io/download.html \
The following are the steps to sample threads' cpu consumption:

1. Launch VisualVM and double click target java process in Applications panel on left.
2. Select Sample tab and click CPU button at top.
3. Switch to Thread CPU Time tab and sort "Thread Time(CPU)" column in descending order.

## Method Used to Find Top Time Consumer Method

To make flame graph of top time consumer threads, we use JDK mission control which can be obtained from
https://openjdk.java.net/projects/jmc/8/ \
In order to keep this report concise, we didn't show flame graphs for top time consumer threads. But you can make them
to get a better insight. \
And here are the steps for this purpose:

1. Launch JDK mission control and select "Start Flight Recording..." in context menu for target java process in JVM
   Browser panel.
2. Accept default settings and click Finish button in popup to start flight recording.
3. Once recording finished, select "(Legacy)Threads" view in Outline panel.
4. Filter target thread by name in the view.
5. Find top time consumer stacktrace in Flame View tab at bottom.
