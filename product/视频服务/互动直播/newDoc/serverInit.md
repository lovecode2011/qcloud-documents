## 准备业务后台接口
* 如果开发者采用托管账户模式，那在集成互动直播sdk的过程中基本不需要后台接口，可以跳过此文档；
* 如果开发者采用独立账户模式，请参考下面的数据交互图。
![腾讯互动直播数据交互](https://mc.qcloudimg.com/static/img/4094feaf383cf1e3c5714bd3f9dbfc8e/hudongzhibo.png)

#### 独立账号模式下开发者后台server必须实现的功能

1. 调用腾讯TLS后台API生成sig，派发给客户端；
2. 设计一个业务逻辑。以便在旧sig过期前生成新的sig，派发给客户端；

#### 建议实现开发者后台server的功能

1. 上报创建房间信息；
2. 上报观众进入房间信息；
3. 提供正在进行的主播列表；
4. 统计观众人数；
5. 踢人和中断主播；

### 附录
* [TLS账号登录集成说明](https://www.qcloud.com/document/product/268/7653)。
* 独立模式server demo下载地址(准备中......)
