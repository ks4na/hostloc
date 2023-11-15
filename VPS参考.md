# VPS 选购参考

- racknerd 1c1.5g30g 1Gbps [NY] [Ryzen]
  - 【￥ 140/year】
- raksmart 1c1g 40g 100m [SV]
  - 【￥ 73/year】
- 【不再续费】 megalayer 1c1g 50g 20m 全向
  - 【￥ 99/year】
- 【不再续费】 cloudcone 1c1g 30g（SSD cached）全向 500Mbps [LA]
  - 【￥ 117/year】

> 考虑移除 cloudcone ,转到 contabo hetzner liteserver netcup  注意是否支持ipv6
> 移除 megalayer 服务器  
> 黑五看看 racknerd 的便宜机器测试一下网速，看看 contabo 场地费是否免费

主力：contabo 4c8g 50g 200m 或者 liteserver 4c4g 80g
再在 raksmart 找一台反代机器，带宽要大，实际下载速度要快(或其他平台 CN2GIA 不限流量 最低 10M)

## 测评

### 测评指标

- 性能
  - CPU 性能： `sudo sysbench cpu --cpu-max-prime=20000 run`
  - FIO 磁盘性能： `curl -sL yabs.sh | bash -s -- -r`
- 网速和延迟
  - 延迟测试： `ping.pe/raksmart.yumecoder.top`
  - 多地上下行速率： `curl -Lso- bench.sh | bash`
  - 本机 curl 下载速度测试： `curl -o /dev/null http://<domain>/500mb.bin`

1. 网速测试：
   curl -o /dev/null http://racknerd.yumecoder.top/500mb.bin 【19: 106k 0：2M 】【位置为 NY，开启bbr速度提升很大】
   curl -o /dev/null http://raksmart.yumecoder.top/500mb.bin 【11: 3.64M 12: 3.52M 14: 4.3M 20: 3.86M,4.25M 】
   curl -o /dev/null http://megalayer.yumecoder.top/500mb.bin 【11: 2.42M 12: 80k 14: 20k 20: 50k】
   curl -o /dev/null https://cloudcone.yumecoder.top/500mb.bin 【11: 80k 12: 30k】（进国内变慢了，ucloud 下载没问题）
   curl -o /dev/null <contabo>/500mb.bin

2. 延迟测试
   ping.pe/racknerd.yumecoder.top 【19：250ms】
   ping.pe/raksmart.yumecoder.top 【11：170ms 12: 180ms 14：180ms 20：180ms】
   ping.pe/megalayer.yumecoder.top 【11: 170ms 丢包偶尔 12: 170ms 丢包偶尔 14：170ms 丢包频繁 20：190ms 丢包频繁】
   ping.pe/cloudcone.yumecoder.top 【11: 190ms 丢包频繁 12: 200ms 丢包频繁】
   ping.pe/<contabo>

3. 性能测试

   1. cpu `sudo sysbench cpu --cpu-max-prime=20000 run`

      - racknerd 【2482】
      - cloudcone 【226.45】
      - megalayer 【469.07】
      - raksmart 【255.47】
      - ucloud 【620.88】

   2. bench 脚本 `curl -Lso- bench.sh | bash`

      - racknerd 【cpu: AMD Ryzen 9 7950X 4.5Ghz, IO: 2287M/s】
      - cloudcone 【cpu: Intel Xeon E312xx 2.1Ghz, IO: 290M/s】
      - megalayer 【cpu: 未知 2.3Ghz, IO: 211M/s】
      - raksmart 【cpu: 未知 2.8Ghz, IO: 162M/s】
      - ucloud 【cpu: AMD EPYC 2.9Ghz, IO: 78.7M/s】

   3. FIO 磁盘性能测试 `curl -sL yabs.sh | bash -s -- -r`

      - racknerd 【936 / 5630 / 7060 / 7530】
      - cloudcone 【7.7 / 113 / 161 / 215】
      - megalayer 【3.2 / 52 / 403 / 401】
      - raksmart 【120 / 253 / 245 / 279】
      - ucloud 【15 / 77 / 77 / 77】
