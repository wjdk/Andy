### raft机制概述

每个服务器可能处在三种状态中，分别是Follower、Candidate、Leader。当Follower在一定时间内没有收到来自Leader的hearbeat时，Follower会变成Candidate，并向其他服务器发送RequestVote Rpc来申请选票，当Candidate得到超过半数的选票时，会转变为Leader。Leader会定期向每个服务器发送hearbeat。当客户端需要更新日志时，Leader会先在本地更新但不提交吗，并向所有服务器发送更新请求。当有半数服务器在本地更新了日志时，Leader会提交日志标记事务完成。

### RPC交互

RPC的其中一方任期号若小于另一方，更新较小一方的任期号，并将其状态转变为Follower。

### 领导者选举机制

每个服务器rf初始化完成后执行选举检查器协程```go rf.ticker()```。选举检查器每过固定时间（10ms）检查选举是否超时。若当前时间与上一次心跳时间之差超过选举时间设置，发起一次选举。通过随机设置选举超时时限避免活锁。

发起选举时将状态从Follower转变为Candidate，并将当前任期号+1，投自己一票，然后向其他服务器发送RequesetVote Rpc。得到半数选票时转变为Leader。

RequestVote Rpc: 如果发送Rpc的任期号<投票者的任期号，拒绝投票。否则更新投票者当前任期号后，若投票者未投票且发送者的日志不旧于投票者，同意投票。

### 日志复制机制

Leader被选举出来后会启动三个协程，分别是leaderTicker,appendChecker和commitChecker。
leaderTicker会定期给follower发送心跳信息，使用AppendEntries Rpc发送空日志实现。
appendChecker会定期尝试向follower同步leader的日志：
leader会记录每个服务器的nextIndex表示该服务器下一个需要同步的日志，每一次同步Follower时，会发送对应的nextIndex到当前最新日志位置的所有日志。若更新失败，leader会将对应的nextIndex减一，这样如果日志不一致总有一次nextIndex合法可以更新。
commitChecker会定期检查还未提交的日志是否可以提交：
通过leader的nextIndex数组可以得到当前已更新日志的服务器数量，若过半则可提交，更新leader的commitIndex。

`Start(command)`时客户端会向所有服务器发送执行命令请求，但只由认为自己是Leader的服务器会在本地执行相应的日志。在appendChecker中每隔一段时间尝试将Follower与leader同步。

AppendEntries RPC: 若Leader的任期号小于当前Follower任期号，说明是旧的leader更新失败。否则判断RPC的PrevLogTerm和当前日志是否吻合，不吻合则更新失败，否则从PrevLogTerm开始更新日志。
所有AppendEntries RPC（包括心跳）都会试图根据Leader的commitIndex更新当前Follower的commitIndex（若min(leaderCommit,len(rf.log)-1)更大则更新）。

每个服务器初始化后都会启动一个apply()协程，定期检查自己的commitIndex是否更新，若更新则实际上提交commitIndex更新区间的日志。

### QA

Q：为什么不在Start(command)时直接尝试更新Follower的日志？
A：由于网络或者服务器崩溃原因Follower可能没有办法立即更新日志，而我们需要在服务器恢复的第一时间恢复正常工作，所以采用这种协程轮询的方式。

Q：什么情况下会出现两个Leader？
A: 有A,B,C,D,E五台服务器，A初始为Leader。A、B发生网络分区，和C、D、E断开连接后，C成为新的Leader，但A依旧为Leader，此时客户端执行新的命令，C执行的命令能得到过半认可成功提交，A只能得到AB的认可无法提交。此时网络恢复连接后，由于C是经过新的选举后的Leader任期号更高，所以A会变为旧的leader在RPC的过程中转变为Follower。正确性：任期号更高的Leader总是得到了超过半数认可，因此执行日志时实际会提交日志的只有新leader而不是旧leader。

