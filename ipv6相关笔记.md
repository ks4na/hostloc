# ipv6 相关笔记

## 检查带宽是否支持 ipv6

在浏览器地址栏输入网址 `http://test-ipv6.com/`，在页面会给出您的 ipv6 网络测试结果。

## MEMO

- 配置 IPv4/IPv6 双栈站点，整理笔记

  - 服务器设置 ipv6
  - 然后 DNS 可以在相应的 ipv4 记录基础上配置相同但 type 为 AAAA 的 ipv6 解析地址
  - 通常使用 nginx 进行反向代理，则监听端口的 server 配置块中必须要有对 ipv6 地址的监听

- 通常手机默认优先 ipv6 访问。

- 详细文档见 [ipw.cn](https://ipw.cn/doc/)
