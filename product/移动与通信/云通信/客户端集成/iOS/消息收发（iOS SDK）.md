
## 1. 消息发送

### 1.1 通用消息发送

**a. 会话获取 **

会话是指面向一个人或者一个群组的对话，通过与单个人或群组之间会话收发消息，发消息时首先需要先获取会话，获取会话需要指定会话类型（群组&单聊），以及会话对方标志（对方帐号或者群号）。获取会话由 getConversation 实现：
 
**原型：** 

```
@interface TIMManager : NSObject

/**
 *  获取会话
 *
 *  @param type 会话类型，TIM_C2C 表示单聊 TIM_GROUP 表示群聊
 *  @param receiver C2C 为对方帐号identifier， GROUP 为群组Id
 *
 *  @return 会话对象
 */
-(TIMConversation*) getConversation: (TIMConversationType)type receiver:(NSString *)receiver;

@end
```
**参数说明： **

参数 | 说明
---|---
type | 会话类型，如果是单聊，填写 TIM_C2C，如果是群聊，填写TIM_GROUP 
receiver | 会话标识，单聊情况下，receiver为对方帐号identifier，群聊情况下，receiver为群组Id 

**示例：**
 
通过指定类型，获取相应会话。 
以下示例获取对方identifier 为"iOS-001"的单聊会话： 

```
TIMConversation * c2c_conversation = [[TIMManager sharedInstance] getConversation:TIM_C2C receiver:@"iOS-001"];
```

以下示例获取群组Id为"TGID1JYSZEAEQ"的群聊会话： 

```
TIMConversation * grp_conversation = [[TIMManager sharedInstance] getConversation:TIM_GROUP receiver:@"TGID1JYSZEAEQ"];
```

**b. 消息发送 **

通过 TIMManager 获取会话 TIMConversation后，可发送消息和获取会话缓存消息； 
ImSDK中消息的解释可参阅（1.2.1. ImSDK对象简介)。 
ImSDK中的消息由TIMMessage表达， 一个TIMMessage 由多个 TIMElem 组成，每个TIMElem可以是文本和图片，也就是说每一条消息可包含多个文本和多张图片。
![](//mccdn.qcloud.com/static/img/7226ab79d4294cc53980c888892f5c6d/image.png)
发消息通过 TIMConversation 的成员 sendMessage 实现，有两种方式实现，一种使用闭包，另一种调用方实现protocol回调： 

**原型：**
```
@interface TIMConversation : NSObject

-(int) sendMessage: (TIMMessage*)msg succ:(TIMSucc)succ fail:(TIMFail)fail;

-(int) sendMessage: (TIMMessage*)msg cb:(id)cb;

@end
```
**参数说明：**

参数 | 说明
---|---
msg | 消息 
succ | 成功回调 
fail | 失败回调 
cb | TIMCallback protocol 回调 

以下分别解释发送文本、图片等不同类型的消息。

### 1.2 文本消息发送

文本消息由 TIMTextElem 定义： 

```
@interface TIMTextElem : TIMElem {
    NSString * text;
}
```

text 传递需要发送的文本消息：

**示例：**

```
TIMTextElem * text_elem = [[TIMTextElem alloc] init];

[text_elem setText:@"this is a text message"];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:text_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```

其中失败回调中，code表示错误码，具体可参阅错误码，err表示错误描述。

### 1.3 图片消息发送

图片消息由TIMImageElem定义。它是TIMElem的一个子类，也就是说图片也是消息的一种内容。 发送图片的过程，就是将TIMImageElem加入到TIMMessage中，然后随消息一起发送出去。详细如下：
 
**TIMImageElem原型：**

```
/**
 *  存储要发送的图片路径，必须是本地路径，可参考下面示例
 */
@interface TIMImageElem : TIMElem

/**
 *  要发送的图片路径
 */
@property(nonatomic,retain) NSString * path;

/**
 *  发送时不用关注，接收时保存生成的图片所有规格，用法详见4.5.2节
 */
@property(nonatomic,retain) NSArray * imageList;

/**
 *  图片压缩等级，详见 TIM_IMAGE_COMPRESS_TYPE
 */
@property(nonatomic,assign) TIM_IMAGE_COMPRESS_TYPE level;

/**
 *  查询上传进度
 */
- (uint32_t) getUploadingProgress;

@end
```

**参数说明：**

参数 | 说明
---|---
path | 存储要发送的图片路径，必须是本地路径，可参考下面示例 
imageList | 发送时不用关注，接收时保存生成的图片所有规格，可以参阅图片消息接收部分
level | 发送图片前对图片进行压缩，level表示压缩等级，详见 TIM_IMAGE_COMPRESS_TYPE 定义

发送图片时，只需要设置图片路径path。发送成功后可通过 imageList 获取所有图片类型。另外通过 getUploadingProgress 方法可查询当前上传进度。

**图片发送示例： **

```
/**
*  获取聊天会话, 以同用户iOS-001的单聊为例，群聊可参见4.1节a部分
*/
TIMConversation * c2c_conversation = [[TIMManager sharedInstance] getConversation:TIM_C2C receiver:@"iOS-001"];

/**
*  构造一条消息
*/
TIMMessage * msg = [[TIMMessage alloc] init];

/**
*  构造图片内容
*/
TIMImageElem * image_elem = [[TIMImageElem alloc] init];
image_elem.path = @"/xxx/imgPath.jpg";

/**
*  将图片内容添加到消息容器中
*/
[msg addElem:image_elem];

/**
*  发送消息
*/
[conversation sendMessage:msg succ:^(){  //成功
       NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {  //失败
       NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```

示例中发送了一张绝对路径是@"/xxx/imgPath.jpg"的图片。 

### 1.4 表情消息发送

表情消息由 TIMFaceElem 定义，SDK并不提供表情包，如果开发者有表情包，可使用 index 存储表情在表情包中的索引，由用户自定义，或者直接使用data存储表情二进制信息以及字符串key，都由用户自定义，SDK内部只做透传：

```
@interface TIMFaceElem : TIMElem

/**
 *  表情索引，用户自定义
 */
@property(nonatomic, assign) int index;
/**
 *  额外数据，用户自定义
 */
@property(nonatomic,retain) NSData * data;
@end
```

**参数说明：**

参数 | 说明
---|---
index|表情索引标号，由开发者定义
data|表情二进制数据，由开发者定义

index 和 data 只需要传入一个即可，ImSDK只是透传这两个数据。

**示例：**

```
TIMFaceElem * face_elem = [[TIMFaceElem alloc] init];

[image_elem setIndex:10];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:face_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```

示例中发送了索引为10的表情，具体10标识哪种表情，需要开发者在两端都持有一份表情包，索引到编号为10的表情，也可以通过data通过二进制数据来标识。

### 1.5 语音消息发送 

语音消息由 TIMSoundElem 定义，其中data存储语音数据，语音数据需要提供时长信息，以秒为单位，**注意，一条消息只能有一个语音Elem，添加多条语音Elem时，AddElem函数返回错误1，添加不生效，另外，语音和文件elem不一定会按照添加时的顺序获取，建议逐个判断elem类型展示，而且语音和文件elem也不保证按照发送的elem顺序排序**。 

```
/**
 *  语音消息Elem
 */
@interface TIMSoundElem : TIMElem
/**
 *  上传时任务Id，可用来查询上传进度
 */
@property(nonatomic,assign) uint32_t taskId;
/**
 *  上传时，语音文件的路径（设置path时，优先上传语音文件）
 */
@property(nonatomic,retain) NSString * path;
/**
 *  存储语音数据
 */
@property(nonatomic,retain) NSData * data;
/**
 *  语音消息内部ID
 */
@property(nonatomic,retain) NSString * uuid;
/**
 *  语音数据大小
 */
@property(nonatomic,assign) int dataSize;
/**
 *  语音长度（秒），发送消息时设置
 */
@property(nonatomic,assign) int second;

/**
 *  查询上传进度
 */
- (uint32_t) getUploadingProgress;

@end
```

**参数说明：**

参数|说明
---|---
path|上传语音的文件路径
data|上传的语音数据，如果传入path，此字段留空即可，path和data二者只需要传入一个，建议使用path
uuid|上传成功以后会生成唯一的标识，用户可以根据此标识保存文件，ImSDK内部不会保存资源数据
dataSize|语音数据大小
second|语音长度

另外，通过 getUploadingProgress 方法可查询上传进度。

**示例： **

```
TIMSoundElem * sound_elem = [[TIMSoundElem alloc] init];

[sound_elem setPath:@"./xxx.mp3"];
[sound_elem setSecond:10];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:sound_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```


###  1.6 地理位置消息发送 

地理位置消息由 TIMLocationElem定义，其中desc存储位置的描述信息，longitude、latitude分别表示位置的经度和纬度：

```
@interface TIMLocationElem : TIMElem
/**
 *  地理位置描述信息，发送消息时设置
 */
@property(nonatomic,retain) NSString * desc;
/**
 *  纬度，发送消息时设置
 */
@property(nonatomic,assign) double latitude;
/**
 *  经度，发送消息时设置
 */
@property(nonatomic,assign) double longitude;
@end
```
**示例： **

```
NSString *desc= @"腾讯大厦";

TIMLocationElem * location_elem = [[TIMLocationElem alloc] init];

[location_elem setDesc:desc];
[location_elem setLatitude:113.93];
[location_elem setLongitude:22.54];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:location_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}]; 
```

### 1.7 小文件消息发送 

文件消息由 TIMFileElem 定义，另外还可以提供额外的显示文件名信息。**注意：一条消息只能添加一个文件Elem，添加多个文件时，AddElem函数返回错误1，另外，语音和文件elem不一定会按照添加时的顺序获取，建议逐个判断elem类型展示。**

```
/**
 *  文件消息Elem
 */
@interface TIMFileElem : TIMElem
/**
 *  上传时，文件的路径（设置path时，优先上传文件）
 */
@property(nonatomic,retain) NSString * path;
/**
 *  文件数据，发消息时设置，收到消息时不能读取，通过 getFileData 获取数据
 */
@property(nonatomic,retain) NSData * data;
/**
 *  文件内部ID
 */
@property(nonatomic,retain) NSString * uuid;
/**
 *  文件大小
 */
@property(nonatomic,assign) int fileSize;
/**
 *  文件显示名，发消息时设置
 */
@property(nonatomic,retain) NSString * filename;

@end
```
**参数说明：**

参数|说明
---|---
path | 文件路径
data | 要发送的文件二进制数据。如设置path，可不用设置data，二者只需要设置一个字段即可，推荐使用path
filename | 文件名，SDK不校验是否正确，只透传。 

**示例：**

```

TIMFileElem * file_elem = [[TIMFileElem alloc] init];

[file_elem setPath:./xxx/a.txt];
[file_elem setFilename:@"a.txt"];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:file_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```

### 1.7 自定义消息发送 

自定义消息是指当内置的消息类型无法满足特殊需求，开发者可以自定义消息格式，内容全部由开发者定义，ImSDK只负责透传。另外如果需要iOS APNs推送，还需要提供一段推送文本描述，方便展示。
自定义消息由 TIMCustomElem定义，其中data 存储消息的二进制数据，其数据格式由开发者定义，desc存储描述文本。一条消息内可以有多个自定义Elem，并且可以跟其他Elem混合排列，离线Push时叠加每个Elem的desc描述信息进行下发。

```
/**
 *  自定义消息类型
 */
@interface TIMCustomElem : TIMElem

/**
 *  自定义消息二进制数据
 */
@property(nonatomic,retain) NSData * data;
/**
 *  自定义消息描述信息，做离线Push时文本展示
 */
@property(nonatomic,retain) NSString * desc;

@end
```

**参数说明：**

参数|说明
---|---
data | 自定义消息二进制数据 
desc | 自定义消息描述信息 

**示例：**

```
// xml协议的自定义消息
NSString * xml = @"testTitlethis is custom msgtest msg body";

// 转换为NSData
NSData *data = [xml dataUsingEncoding:NSUTF8StringEncoding];

TIMCustomElem * custom_elem = [[TIMCustomElem alloc] init];

[custom_elem setData:data];
[custom_elem setDesc:@"this is one custom message"];

TIMMessage * msg = [[TIMMessage alloc] init];
[msg addElem:custom_elem];

[conversation sendMessage:msg succ:^(){
	NSLog(@"SendMsg Succ");
}fail:^(int code, NSString * err) {
	NSLog(@"SendMsg Failed:%d->%@", code, err);
}];
```

示例中拼接一段xml消息，具体展示由开发者决定。 

### 1.9 Elem顺序 

目前文件和语音Elem不一定会按照添加顺序传输，其他Elem按照顺序，不过建议不要过于依赖Elem顺序进行处理，应该逐个按照Elem类型处理，防止异常情况下进程Crash。

### 1.10 在线消息

对于某些场景，需要发送在线消息，即用户在线时收到消息，如果用户不在线，下次登录也不会看到消息，可用于通知类消息，这种消息不会进行存储，也不会计入未读计数。发送接口与sendMessage类似，**注意：只针对单聊消息有效。**

```
@interface TIMConversation : NSObject

/**
 *  发送在线消息（服务器不保存消息）
 *
 *  @param msg  消息体
 *  @param succ 成功回调
 *  @param fail 失败回调
 *
 *  @return 0 成功
 */
-(int) sendOnlineMessage: (TIMMessage*)msg succ:(TIMSucc)succ fail:(TIMFail)fail;

@end
```

### 1.11 消息转发

在2.4.0及以上版本，在TIMMessage中提供了copyFrom接口，可以方便地拷贝其他消息的内容到当前消息，然后将消息重新发送给其他人。

**原型：**

```
/**
 *  消息
 */
@interface TIMMessage : NSObject
/**
 *  拷贝消息中的属性（ELem、priority、online、offlinePushInfo）
 *
 *  @param srcMsg 源消息
 *
 *  @return 0 成功
 */
- (int)copyFrom:(TIMMessage*)srcMsg;

@end
```

## 2. 接收消息

在多数情况下，用户需要感知新消息的通知，这时只需注册新消息通知回调 TIMMessageListener，如果用户是登录状态，ImSDK收到新消息会通过此方法抛出，另外需要注意，通过onNewMessage抛出的消息不一定是未读的消息，只是本地曾经没有过的消息（例如在另外一个终端已读，拉取最近联系人消息时可以获取会话最后一条消息，如果本地没有，会通过此方法抛出）。在用户登录之后，ImSDK会拉取离线消息，为了不漏掉消息通知，需要在登录之前注册新消息通知。

**原型：**
```
@protocol TIMMessageListener
@optional
/**
 *  新消息通知
 *
 *  @param msgs 新消息列表，TIMMessage 类型数组
 */
- (void)onNewMessage:(NSArray*) msgs;
@end
 
@interface TIMManager : NSObject
 
-(int) setMessageListener: (id)listener;
 
@end
```

**参数说明：**

参数 | 说明
---|---
msgs | 新消息列表，注意这里可能同时会有多条消息抛出，相同会话的消息由老到新排序

回调消息内容通过参数TIMMessage传递，通过TIMMessage可以获取消息和相关会话的详细信息，如消息文本，语音数据，图片等等，可参阅（4.5消息解析）部分。

**示例：**

```
@interface TIMMessageListenerImpl : NSObject
- (void)onNewMessage:(TIMMessage*) msg;
@end
 
@implementation TIMMessageListenerImpl
@synthesize onMessage;
- (void)onNewMessage:(NSArray*) msgs {
    NSLog(@"NewMessages: %@", msgs);
}
@end
 
TIMMessageListenerImpl * impl = [[TIMMessageListenerImpl alloc] init];
[[TIMManager sharedInstance] setMessageListener:impl];
```

示例中设置消息回调通知，并且在有新消息时直接打印消息，更详细的消息解析，可参阅以下章节部分。

### 2.1 消息解析 

收到消息后，可用过 getElem 从 TIMMessage中获取所有的Elem节点： 

**遍历Elem原型：**

```
@interface TIMMessage : NSObject

-(int) elemCount;
-(TIMElem*) getElem:(int)index;

@end
```

**示例： **

```
TIMMessage * message = /* 消息 */

int cnt = [message elemCount];

for (int i = 0; i < cnt; i++) {
 TIMElem * elem = [message getElem:i];
 
 if ([elem isKindOfClass:[TIMTextElem class]]) {
     TIMTextElem * text_elem = (TIMTextElem * )elem;
 }
 else if ([elem isKindOfClass:[TIMImageElem class]]) {
     TIMImageElem * image_elem = (TIMImageElem * )elem;
 }
}
```

### 2.2 接收图片消息

接收方收到消息后，可通过 getElem 从 TIMMessage中获取所有的Elem节点，其中类型为TIMImageElem的是图片消息节点。然后通过imageList获取该图片的所有规格用来展示。详细如下：

** TIMImageElem类原型：**

```
**
 *  图片消息Elem
 */
@interface TIMImageElem : TIMElem

/**
 *  要发送的图片路径
 */
@property(nonatomic,retain) NSString * path;

/**
 *  保存本图片的所有规格，目前最多包含三种规格: 缩略图、大图、原图， 每种规格保存在一个TIMImage对象中
 */
@property(nonatomic,retain) NSArray * imageList;
@end
```

**参数说明：**

参数|说明
---|---
path | 收消息时不用关注，为nil 
imageList | 保存本图片的所有规格，目前最多包含三种规格: 缩略图、大图、原图， 每种规格保存在一个TIMImage对象中 

**TIMImage说明： **

获取到消息时通过imageList得到所有的图片规格，为TIMImage数据，得到TIMImage后可通过图片大小进行占位，通过接口 getImage下载不同规格的图片进行展示。**下载的数据需要由开发者缓存，ImSDK每次调用getImage都会从服务端重新下载数据。建议通过图片的uuid作为key进行图片文件的存储。**
 
**原型：** 

```
@interface TIMImage : NSObject
/**
 *  图片ID，全局唯一，图片标识，相同uuid的图片可以不再重复下载
 */
@property(nonatomic,retain) NSString * uuid;
/**
 *  图片规格，有三种Thumb、Large、Original，分别代表缩略图、大图、原图
 */
@property(nonatomic,assign) TIM_IMAGE_TYPE type;
/**
 *  图片大小
 */
@property(nonatomic,assign) int size;
/**
 *  图片宽度，发送图片消息时设置
 */
@property(nonatomic,assign) int width;
/**
 *  图片高度，发送图片消息时设置
 */
@property(nonatomic,assign) int height;
/**
 *  下载URL
 */
@property(nonatomic, retain) NSString * url;

/**
 *  获取图片
 *
 *  @param path 图片保存路径
 *  @param succ 成功回调，返回图片数据
 *  @param fail 失败回调，返回错误码和错误描述
 */
- (void) getImage:(NSString*) path succ:(TIMSucc)succ fail:(TIMFail)fail;

/**
 *  获取图片
 *
 *  @param path 图片保存路径，同前一个区别就是图片保存在path指向的文件里
 *  @param cb 图片获取回调
 */
- (void) getImage:(NSString*) path cb:(id)cb;

@end
```

**参数说明：**

参数 | 说明
---|---
succ | 为成功回调，返回图片二进制信息 
fail   | 为失败回调，返回错误码和描述信息 
cb | protocol回调 

**图片规格说明：**
 
每幅图片有三种规格，分别是Thumb(缩略图)、Large(大图)、Original(原图)。 
原图是指用户发送的原始图片，尺寸和大小都保持不变。 
缩略图是将原图等比压缩，压缩后宽、高中较小的一个等于198像素。 
大图也是将原图等比压缩，压缩后宽、高中较小的一个等于720像素。 
如果原图尺寸就小于198像素，则三种规格都保持原始尺寸，不需压缩。 
如果原图尺寸在198~720之间，则大图和原图一样，不需压缩。 
在手机上展示图片时，建议优先展示缩略图，用户点击缩略图时再下载大图，点击大图时再下载原图。当然开发者也可以选择跳过大图，点击缩略图时直接下原图。 
在pad或PC上展示图片时，由于分辨率较大，且基本都是wifi或有线网络，建议直接显示大图，用户点击大图时再下载原图。 

**图片解析示例：**

```
//以收到新消息回调为例，介绍下图片消息的解析过程

//接收到的图片保存的路径
NSString * pic_path = @"/xxx/imgPath.jpg";

[conversation getMessage:10 last:nil succ:^(NSArray * msgList) {  //获取消息成功
	//遍历所有的消息
	for (TIMMessage * msg in msgList) {
		//遍历一条消息的所有元素
		for (TIMImageElem * elem in TIMImageElem) {
		   //图片元素
			if ([elem isKindOfClass:[TIMImageElem class]]) {
				TIMImageElem * image_elem = (TIMImageElem * )elem;

				//遍历所有图片规格(缩略图、大图、原图)
				NSArray * imgList = [image_elem imageList];
				for (TIMImage * image in imgList) {
					[image getImage:pic_path succ:^(){  //接收成功
						NSLog(@"SUCC: pic store to %@", pic_path);
					}fail:^(int code, NSString * err) {  //接收失败
						NSLog(@"ERR: code=%d, err=%@", code, err);
					}];
				}
			}
		}
	}
} fail:^(int code, NSString * err) {  //获取消息失败
	NSLog(@"Get Message Failed:%d->%@", code, err);
}];
```
该示例从会话中取出10条消息，获取图片消息并下载相应数据。 

###  2.3 接收语音消息

收到消息后，可用过 getElem 从 TIMMessage中获取所有的Elem节点，其中TIMSoundElem为语音消息节点，原型如下： 

```
@interface TIMSoundElem : TIMElem

/**
 *  上传时，语音文件的路径（设置path时，优先上传语音文件）
 */
@property(nonatomic,retain) NSString * path;

/**
 *  发送时设置为语音数据，接收时使用getSoundToFile获得数据
 */
@property(nonatomic,retain) NSData * data;

/**
 *  语音消息内部ID
 */
@property(nonatomic,retain) NSString * uuid;

/**
 *  语音数据大小
 */
@property(nonatomic,assign) int dataSize;

/**
 *  语音长度（秒），发送消息时设置
 */
@property(nonatomic,assign) int second;

/**
 *  获取语音数据到指定路径的文件中
 *
 *  @param path 语音保存路径
 *  @param succ 成功回调，返回语音数据
 *  @param fail 失败回调，返回错误码和错误描述
 */
- (void) getSoundToFile:(NSString*)path succ:(TIMSucc)succ fail:(TIMFail)fail;

@end
```

其中 data 和 path 为创建消息时填写的语音信息，接收消息时为空。 

**其他参数说明：**

参数|说明
---|---
uuid | 唯一标识，方便用户缓存 
dataSize | 语音文件大小 
second | 语音时长，以秒为单位

获取到消息时可通过时长占位，通过接口 getSoundToFile 下载语音资源，getSoundToFile 接口每次都会从服务端下载，如需缓存或者存储，开发者可根据uuid作为key进行外部存储，ImSDK并不会存储资源文件。 

**语音消息已读状态：**

语音是否已经播放，可使用 [消息自定义字段](/doc/product/269/消息收发（iOS%20SDK）#3.8-.E6.B6.88.E6.81.AF.E8.87.AA.E5.AE.9A.E4.B9.89.E5.AD.97.E6.AE.B5) 实现，如 customInt 的值 0 表示未播放，1表示播放，当用户点击播放后可设置 customInt 的值为1。

```
@interface TIMMessage : NSObject

/**
 *  设置自定义整数，默认为0
 *
 *  @param param 设置参数
 *
 *  @return TRUE 设置成功
 */
- (BOOL) setCustomInt:(int32_t) param;

@end
```

### 2.4 接收小文件消息

收到消息后，可用过 getElem 从 TIMMessage中获取所有的Elem节点，其中TIMFileElem为文件消息节点，原型如下： 

```
@interface TIMFileElem : TIMElem
/**
 *  上传时，文件的路径（设置path时，优先上传文件）
 */
@property(nonatomic,retain) NSString * path;

/**
 *  文件数据，发消息时设置，收到消息时不能读取，通过 getFileData 获取数据
 */
@property(nonatomic,retain) NSData * data;
/**
 *  文件内部ID
 */
@property(nonatomic,retain) NSString * uuid;
/**
 *  文件大小
 */
@property(nonatomic,assign) int fileSize;
/**
 *  文件显示名，发消息时设置
 */
@property(nonatomic,retain) NSString * filename;

/**
 *  获取文件数据到指定路径的文件中
 *
 *  @param path 文件保存路径
 *  @param succ 成功回调，返回数据
 *  @param fail 失败回调，返回错误码和错误描述
 */
- (void) getToFile:(NSString*)path succ:(TIMSucc)succ fail:(TIMFail)fail;
@end
```

其中 path 和 data 为创建消息时填写的文件二进制信息，Get消息时为空。 

**参数说明：**

参数|说明
---|---
uuid | 唯一ID，方便用户进行缓存 
fileSize | 文件大小 
filename |文件显示名 

获取到消息时可只展示文件大小和显示名，通过接口 getToFile 下载文件资源。getToFile 接口每次都会从服务端下载，如需缓存或者存储，开发者可根据uuid作为key进行外部存储，ImSDK并不会存储资源文件。

## 3. 消息属性 

### 3.1 消息是否已读

通过消息属性 isReaded 是否消息已读。这里已读与否取决于APP测进行的[已读上报](/doc/product/269/未读消息计数（iOS%20SDK）#3.-.E5.B7.B2.E8.AF.BB.E4.B8.8A.E6.8A.A5)。

```
@interface TIMMessage : NSObject

/**
 *  是否已读
 *
 *  @return TRUE 已读  FALSE 未读
 */
-(BOOL) isReaded;

@end
```

### 3.2 消息状态

通过消息属性 status 可以获取消息的当前状态，如，发送中、发送成功、发送失败和删除，对于删除的消息，需要UI判断状态并隐藏。

```
/**
 *  消息状态
 */
typedef NS_ENUM(NSInteger, TIMMessageStatus){
    /**
     *  消息发送中
     */
    TIM_MSG_STATUS_SENDING              = 1,
    /**
     *  消息发送成功
     */
    TIM_MSG_STATUS_SEND_SUCC            = 2,
    /**
     *  消息发送失败
     */
    TIM_MSG_STATUS_SEND_FAIL            = 3,
    /**
     *  消息被删除
     */
    TIM_MSG_STATUS_HAS_DELETED          = 4,
};

@interface TIMMessage : NSObject

/**
 *  消息状态
 *
 *  @return TIMMessageStatus 消息状态
 */
-(TIMMessageStatus) status;

@end
```

### 3.3 是否是自己发出的消息

通过消息属性 isSelf 可以判断消息是否是自己发出的消息，界面显示时可用。

```
@interface TIMMessage : NSObject

/**
 *  是否发送方
 *
 *  @return TRUE 表示是发送消息    FALSE 表示是接收消息
 */
-(BOOL) isSelf;

@end
```

### 3.4 消息发送者以及相关资料

对于群消息，可以通过 TIMMessage 的方法 sender 得到发送用户，另外也可以通过方法 GetSenderProfile 和 GetSenderGroupMemberProfile 获取用户自己的资料和所在群的资料。**(注意，此字段是消息发送时获取用户资料写入消息体，如后续用户资料更新，此字段不会相应变更，只有产生的新消息中才会带最新的昵称）。**

```
@interface TIMMessage : NSObject

/**
 *  获取发送方
 *
 *  @return 发送方标识
 */
-(NSString *) sender;

/**
 *  获取发送者资料（发送者为本人时可能为空）
 *
 *  @return 发送者资料，nil 表示没有获取资料，目前只有字段：identifier、nickname、faceURL、customInfo
 */
-(TIMUserProfile *) GetSenderProfile;

/**
 *  获取发送者群内资料（发送者为本人时可能为空）
 *
 *  @return 发送者群内资料，nil 表示没有获取资料或者不是群消息，目前只有字段：member、nameCard、role、customInfo
 */
-(TIMGroupMemberInfo *) GetSenderGroupMemberProfile;

@end
```

1.9 版本之前，只有在线消息 onNewMessage 抛出的消息可以获取到用户资料，1.9版本以后，通过 getMessage 得到的消息也可以拿到资料（更新版本之前已经收到本地的消息无法获取到）。

对于单聊消息，通过通过 TIMMessage 的 getConversation 获取到对应会话，会话的 getReceiver 可以得到正在聊天的用户。

### 3.5 消息时间

通过消息属性 timestamp 可以得到消息时间，该时间是server时间，而非本地时间。在创建消息时，此时间为根据server时间校准过的时间，发送成功后会改为准确的server时间。

```
@interface TIMMessage : NSObject

/**
 *  当前消息的时间戳
 *
 *  @return 时间戳
 */
-(NSDate*) timestamp;

@end
```

### 3.6 消息删除

目前暂不支持Server消息删除，只能在本地删除，有两种删除方法，一种是remove，通过这种方法删除的消息，仅是打上删除的标记，并未真正删除。另外一种是 delFromStorage，从本地数据库彻底删除，但是如果使用 getMessage，可能从server 漫游消息获取到本地，此消息可能重新出现。所以如果使用了 getMessage，建议使用 remove 方法进行删除和界面过滤。

```
@interface TIMMessage : NSObject

/**
 *  删除消息：注意这里仅修改状态
 *
 *  @return TRUE 成功
 */
-(BOOL) remove;

/**
 *  从本地数据库删除消息：注意群组消息通过getMessage接口会从svr同步到本地
 *
 *  @return TRUE 成功
 */
-(BOOL) delFromStorage;

@end
```

### 3.7 消息Id

消息Id也有两种，一种是当消息生成时，就已经固定（msgId），这种方式可能跟其他用户产生的消息冲突，需要再加一个时间未读，可以认为10分钟以内的消息可以使用msgId区分。另外一种，当消息发送成功以后才能固定下来（uniqueId），这种方式能保证全局唯一。这两种方式都需要在同一个会话内判断。

```
@interface TIMMessage : NSObject

/**
 *  消息Id
 */
-(NSString *) msgId;

/**
 *  获取消息uniqueId
 *
 *  @return uniqueId
 */
- (uint64_t) uniqueId;

@end
```

### 3.8 消息自定义字段

开发者可以对消息增加自定义字段，如自定义整数、自定义二进制数据，可以根据这两个字段做出各种不通效果，比如语音消息是否已经播放等等。另外需要注意，此自定义字段仅存储于本地，不会同步到server，更换终端获取不到。

```
@interface TIMMessage : NSObject

/**
 *  设置自定义整数，默认为0
 *
 *  @param param 设置参数
 *
 *  @return TRUE 设置成功
 */
- (BOOL) setCustomInt:(int32_t) param;

/**
 *  设置自定义数据，默认为""
 *
 *  @param data 设置参数
 *
 *  @return TRUE 设置成功
 */
- (BOOL) setCustomData:(NSData*) data;

/**
 *  获取CustomInt
 *
 *  @return CustomInt
 */
- (int32_t) customInt;

/**
 *  获取CustomData
 *
 *  @return CustomData
 */
- (NSData*) customData;

@end
```

### 3.9 消息优先级

对于直播场景，会有点赞和发红包功能，点赞相对优先级较低，红包消息优先级较高，具体消息内容可以使用TIMCustomElem进行定义，发送消息时，可设置消息优先级。具体消息优先级的策略，可参阅 [互动直播集成多人聊天方案](/doc/product/269/互动直播集成多人聊天方案)。**注意：只针对群聊消息有效。**

```
@interface TIMMessage : NSObject

/**
 *  设置消息的优先级
 *
 *  @param priority 优先级
 *
 *  @return TRUE 设置成功
 */
- (BOOL) setPriority:(TIMMessagePriority)priority;

/**
 *  获取消息的优先级
 *
 *  @return 优先级
 */
- (TIMMessagePriority) getPriority;

@end
```

### 3.10 群组消息会话的接收消息选项

对于群组会话消息，可以通过消息属性判断本群组设置的接收消息选项，可参阅 [群组管理](/doc/product/269/群组管理（iOS%20SDK）#5.11-.E4.BF.AE.E6.94.B9.E6.8E.A5.E6.94.B6.E7.BE.A4.E6.B6.88.E6.81.AF.E9.80.89.E9.A1.B9)。**注意：只针对群聊消息有效。**

```
@interface TIMMessage : NSObject

/**
 *  获取消息所属会话的接收消息选项（仅对群组消息有效）
 *
 *  @return 接收消息选项
 */
- (TIMGroupReceiveMessageOpt) getRecvOpt;

@end
```

### 3.11 已读回执

对于单聊消息，用户开启已读回执功能后，对方调用setReadMessage时会同步已读信息到本客户端。

**开启已读回执功能：**

```
@interface TIMManager : NSObject

/**
 * 启用已读回执，启用后在已读上报时会给对方发送回执，只对单聊回话有效
 */
-(void) enableReadReceipt;

/**
 *  设置消息回执回调
 *
 *  @param listener 回调
 *
 *  @return 0 成功
 */
-(int) setMessageReceiptListener: (id<TIMMessageReceiptListener>)listener;

@end
```

**原型：**

```
@interface TIMMessage : NSObject

/**
 *  对方是否已读（仅C2C消息有效）
 *
 *  @return TRUE 已读  FALSE 未读
 */
-(BOOL) isPeerReaded;

@end
```


## 4. 会话操作

### 4.1 获取所有会话 

可以通过 ConversationCount 获取当前会话数量，从而得到所有本地会话：
 
**原型： **

```
@interface TIMManager : NSObject

-(int) ConversationCount;
-(TIMConversation*) getConversationByIndex:(int)index;

@end
```

**示例：**

```
int cnt = [[TIMManager sharedInstance] ConversationCount];

for (int i = 0; i < cnt; i++) {
	TIMConversation * conversation = [[TIMManager sharedInstance] getConversationByIndex:i];
}
```

2.0以上版本，提供 getConversationList 获取当前会话列表：

**原型： **

```
@interface TIMManager : NSObject

/**
 *  获取会话（TIMConversation*）列表
 *
 *  @return 会话列表
 */
-(NSArray*) getConversationList;

@end
```

**示例：**

```
NSArray * conversations = [[TIMManager sharedInstance] getConversationList];
NSLog(@"current session list : %@", [conversations description])
```

### 4.2 最近联系人漫游

ImSDK登录以后默认会获取最近联系人漫游，同时每个会话会获取到最近的一条消息。如果不需要此功能，可以调用方法禁用：

```
@interface TIMManager : NSObject

/**
 *  登录时禁止拉取最近联系人列表
 */
-(void) disableRecentContact;

@end
```

### 4.3 获取会话本地消息

ImSDK 会在本地进行消息存储，可通过 TIMConversation 方法的 getLocalMessage 获取，此方法为异步方法，需要通过设置回调得到消息数据，对于单聊，登录后可以获取离线消息，对于群聊，开启最近联系人漫游的情况下，登录后只能获取最近一条消息，可通过getMessage获取漫游消息。
 
**原型： **

```
@interface TIMConversation : NSObject

/**
 *  获取本地会话消息
 *
 *  @param count 获取数量
 *  @param last  上次最后一条消息
 *  @param succ  成功时回调
 *  @param fail  失败时回调
 *
 *  @return 0 本次操作成功
 */
-(int) getLocalMessage: (int)count last:(TIMMessage*)last succ:(TIMGetMsgSucc)succ fail:(TIMFail)fail;

/**
 *  获取本地会话消息
 *
 *  @param count 获取数量
 *  @param last  上次最后一条消息
 *  @param cb    回调
 *
 *  @return 0 成功
 */
-(int) getLocalMessage: (int)count last:(TIMMessage*)last cb:(id<TIMGetMessageCallback>)cb;

@end
```

**参数说明：**

参数|说明
---|---
count | 指定获取消息的数量 
last | 指定上次获取的最后一条消息，如果last传nil，从最新的消息开始读取 
succ | 成功回调 
fail | 失败回调 

**示例：**

```
[conversation getLocalMessage:10 last:nil succ:^(NSArray * msgList) {
	for (TIMMessage * msg in msgList) {
		if ([msg isKindOfClass:[TIMMessage class]]) {
			NSLog(@"GetOneMessage:%@", msg);
		}
	}
}fail:^(int code, NSString * err) {
	NSLog(@"Get Message Failed:%d->%@", code, err);
}];
```

对于图片、语音等资源类消息，消息体只会包含描述信息，需要通过额外的接口下载数据，可参与消息解析部分，下载后的真实数据不会缓存，需要调用方进行缓存。 

### 4.4 获取会话漫游消息

对于群组，登录后可以获取漫游消息，对于C2C，开通漫游服务后可以获取漫游消息，通过 ImSDK 的 getMessage 接口可以获取漫游消息，如果本地消息全部都是连续的，则不会通过网络获取，如果本地消息不连续，会通过网络获取断层消息。
 
**原型： **

```
@interface TIMConversation : NSObject

/**
 *  获取会话消息
 *
 *  @param count 获取数量
 *  @param last  上次最后一条消息
 *  @param succ  成功时回调
 *  @param fail  失败时回调
 *
 *  @return 0 本次操作成功
 */
-(int) getMessage: (int)count last:(TIMMessage*)last succ:(TIMGetMsgSucc)succ fail:(TIMFail)fail;

/**
 *  获取会话消息
 *
 *  @param count 获取数量
 *  @param last  上次最后一条消息
 *  @param cb    回调
 *
 *  @return 0 成功
 */
-(int) getMessage: (int)count last:(TIMMessage*)last cb:(id<TIMGetMessageCallback>)cb;

@end
```

**参数说明：**

参数|说明
---|---
count | 指定获取消息的数量 
last | 指定上次获取的最后一条消息，如果last传nil，从最新的消息开始读取 
succ | 成功回调 
fail | 失败回调 

**示例：**

```
[conversation getMessage:10 last:nil succ:^(NSArray * msgList) {
	for (TIMMessage * msg in msgList) {
		if ([msg isKindOfClass:[TIMMessage class]]) {
			NSLog(@"GetOneMessage:%@", msg);
		}
	}
}fail:^(int code, NSString * err) {
	NSLog(@"Get Message Failed:%d->%@", code, err);
}];
```

对于图片、语音等资源类消息，消息体只会包含描述信息，需要通过额外的接口下载数据，可参与消息解析部分，下载后的真实数据不会缓存，需要调用方进行缓存。 

### 4.5 删除会话

删除会话有两种方式，一种只删除会话，但保留了所有消息，另一种在删除会话的同时，也删除掉会话相关的消息。可以根据不同应用场景选择合适的方式。另外需要注意的是，如果删除本地消息，对于群组，通过getMessage会拉取到漫游消息，所以存在删除消息成功，但是拉取到消息的情况，取决于是否重新从漫游拉回到本地。如果不需要拉取漫游，可以通过 getLocalMessage 获取消息，或者只通过 getMessage 拉取指定条数（如未读条数数量）的消息。

**原型：**

```
@protocol TIMManager : NSObject

/**
 *  删除会话
 *
 *  @param type 会话类型，TIM_C2C 表示单聊 TIM_GROUP 表示群聊
 *  @param receiver    用户identifier 或者 群组Id
 *
 *  @return TRUE:删除成功  FALSE:删除失败
 */
-(BOOL) deleteConversation:(TIMConversationType)type receiver:(NSString*)receiver;

/**
 *  删除会话和消息
 *
 *  @param type 会话类型，TIM_C2C 表示单聊 TIM_GROUP 表示群聊
 *  @param receiver    用户identifier 或者 群组Id
 *
 *  @return TRUE:删除成功  FALSE:删除失败
 */
-(BOOL) deleteConversationAndMessages:(TIMConversationType)type receiver:(NSString*)receiver;

@end
```
**参数解释：**

参数|说明
---|---
type|会话类型，如果是单聊，填写 TIM_C2C，如果是群聊，填写TIM_GROUP 
receiver|会话标识，单聊情况下，receiver为对方用户identifier，群聊情况下，receiver为群组Id 

其中 deleteConversation 仅删除会话，deleteConversationAndMessages 删除会话以及消息。

**示例：**

```
TIMConversation * conversation = [[TIMManager sharedInstance] deleteConversation:TIM_C2C receiver:@"iOS_002"];
```
使用中删除好友@"iOS_002"的C2C会话。

### 4.6 同步获取会话最后的消息

UI展示最近联系人列表时，时常会展示用户的最后一条消息，在1.9以后版本增加了同步获取接口，用户可以通过此接口方便获取最后一条消息进行展示。**目前没有网络无法获取，另外如果禁用了最近联系人，登陆后在有新消息过来之前无法获取**。此接口获取并不会过滤删除状态消息，需要APP层进行屏蔽。
 
**原型： **

```
@interface TIMConversation : NSObject

/**
 *  从cache中获取最后几条消息
 *
 *  @param count 需要获取的消息数，最多为20
 *
 *  @return 消息（TIMMessage*）列表，第一条为最新消息
 */
-(NSArray*) getLastMsgs:(uint32_t)count;

@end
```

**参数说明：**

参数|说明
---|---
count | 需要获取的消息数，注意这里最多为20 

### 4.7 禁用会话本地存储

直播场景下，群组类型会话的消息量很大，时常需要禁用直播群的本地消息存储功能。在2.2以后版本增加了针对单个会话禁用本地存储的功能，可以实现不存储直播群消息，同时存储C2C私聊消息。
 
**原型： **

```
@interface TIMConversation : NSObject

/**
 * 禁用本会话的存储，只对当前初始化有效，重启后需要重新设置
 * 需要 initSdk 之后调用
 */
-(void) disableStorage;

@end
```

### 4.8 设置会话草稿

UI展示最近联系人列表时，时常会展示用户的草稿内容，在2.2以后版本增加了设置和获取草稿的接口，用户可以通过此接口设置会话的草稿信息。**草稿信息会存本地数据库，重新登录后依然可以获取**。
 
**原型： **

```
@interface TIMMessageDraft : NSObject

/**
 *  设置自定义数据
 *
 *  @param userData 自定义数据
 *
 *  @return 0 成功
 */
-(int) setUserData:(NSData*)userData;

/**
 *  获取自定义数据
 *
 *  @return 自定义数据
 */
-(NSData*) getUserData;

/**
 *  增加Elem
 *
 *  @param elem elem结构
 *
 *  @return 0       表示成功
 *          1       禁止添加Elem（文件或语音多于两个Elem）
 *          2       未知Elem
 */
-(int) addElem:(TIMElem*)elem;

/**
 *  获取对应索引的Elem
 *
 *  @param index 对应索引
 *
 *  @return 返回对应Elem
 */
-(TIMElem*) getElem:(int)index;

/**
 *  获取Elem数量
 *
 *  @return elem数量
 */
-(int) elemCount;

/**
 *  草稿生成对应的消息
 *
 *  @return 消息
 */
-(TIMMessage*) transformToMessage;

/**
 *  当前消息的时间戳
 *
 *  @return 时间戳
 */
-(NSDate*) timestamp;

@end


@interface TIMConversation : NSObject

/**
 *  设置会话草稿
 *
 *  @param draft 草稿内容
 *
 *  @return 0 成功
 */
-(int) setDraft:(TIMMessageDraft*)draft;

/**
 *  获取会话草稿
 *
 *  @return 草稿内容，没有草稿返回nil
 */
-(TIMMessageDraft*) getDraft;

@end
```

**参数说明：**

参数|说明
---|---
draft | 需要设置的草稿 ，需要清空会话草稿时传入nil

### 4.9 删除本地会话消息

ImSDK支持保留会话同时删除本地的会话消息。**再次拉取消息时群组类型会话会从服务器重新拉取到消息**。

**原型： **

```
@interface TIMConversation : NSObject

/**
 *  删除本地会话消息
 *
 *  @param succ  成功时回调
 *  @param fail  失败时回调
 *
 *  @return 0 本次操作成功
 */
-(int) deleteLocalMessage:(TIMSucc)succ fail:(TIMFail)fail;

@end
```

**参数说明：**

参数|说明
---|---
succ | 成功回调
fail | 失败回调


## 5. 系统消息

会话类型（TIMConversationType）除了C2C单聊和Group群聊以外，还有一种系统消息，系统消息不能由用户主动发送，是系统后台在相应的事件发生时产生的通知消息。系统消息目前分为两种，一种是关系链系统消息，一种是群系统消息。

关系链变更系统消息，当有用户加自己为好友，或者有用户删除自己好友的情况下，系统会发出变更通知，开发者可更新好友列表。相关细节可参阅 [关系链变更系统通知](/doc/product/269/用户资料与关系链（iOS%20SDK）#8.-.E5.85.B3.E7.B3.BB.E9.93.BE.E5.8F.98.E6.9B.B4.E7.B3.BB.E7.BB.9F.E9.80.9A.E7.9F.A5) 部分。

当群资料变更，如群名变更或者群内成员变更，在群里会有系统发出一条群事件消息，开发者可在收到消息时可选择是否展示给用户，同时可刷新群资料或者群成员。详细内容可参阅：[群组管理-群事件消息](/doc/product/269/群组管理（iOS%20SDK）#8-.E7.BE.A4.E4.BA.8B.E4.BB.B6.E6.B6.88.E6.81.AF)。

当被管理员踢出群组，被邀请加入群组等事件发生时，系统会给用户发出群系统消息，相关细节可参阅：[群组管理-群系统消息](/doc/product/269/群组管理（iOS%20SDK）#9-.E7.BE.A4.E7.B3.BB.E7.BB.9F.E6.B6.88.E6.81.AF)。 

