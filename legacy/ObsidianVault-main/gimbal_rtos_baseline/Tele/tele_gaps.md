# tele gaps

目前已经解决：

- RX DMA 数据先复制到稳定 pending buffer。
- pending 未消费时，新命令明确计数丢弃。
- TX busy 时丢新帧，不污染正在发送的 DMA buffer。
- snapshot 与 frame mapping 分层，诊断页可切换。

期望逻辑：

- 命令不仅“收到”，还应有统一 `accepted / rejected / applied` 返回。
- 为关键动作分配 command sequence，主机可判断回复对应哪次请求。
- 通信超时是否触发停机，需要按比赛控制链定义，不能默认“一断线就停”。
- 参数持久化与版本校验尚未加入。

