TON (The Open Network)

###  smart contract pay gas fee.
太坊模式：bank，用户对contract发起transfer，用户付gas fee。
TON模式：每个contract自带balance，用户对contract发起transfer，由contract支付gas fee。当contract的balance为0的时候，contract自动被从链上移除。充值balance后，自动添加到链。
这样做的好处是：当需要全网调整Gas Fee的时候，不会影响到用户和Miner的权益。

### Calls between smart contracts are asynchronous and not atomic
1. 异步
2. 非原子

操作可以串联，并且在其中一个任务失败时，对整个事件进行回滚。
>  The whole process is even atomic - if any of these steps fails, even the last, the whole transaction rolls back like it never happened.

