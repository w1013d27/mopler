# 百度网盘SDK

实现百度网盘的一些接口，整合后方便快速调用

> 使用sdk前必须获取授权，内置了设备码授权实现，config中调用回调函数即可。
>
> 参考：https://pan.baidu.com/union/doc/ol0rsap9s
>
> 您也可以选择跳过授权，在初始化配置的时候，填入AccessToken（不会自动刷新）
>
> 官方接口文档地址：https://pan.baidu.com/union/doc/



## 安装

推荐使用go module进行依赖管理

```go
go get github.com/766800551/mopler
```



## 快速开始

```go
func main() {
    c := mopler.NewConfiguration(
    "xxxxxxxxxxxxxxxxxxxxxxxxx",
    "xxxxxxxxxxxxxxxxxxxxxxxxx",
    mopler.WithConfigurationAccessToken(
      "xxxxxxxxxxxxxxxxxxxxxxxxx",
      ),
    )
    //获取用户信息
    u, _ := sdk.ApiUserInfo.UserInfo()
    mopler.PrintToJSON(u)
}
```

填入正确的信息，此时就可以看到返回了一个json字符串：

```json
{
        "baidu_name": "xxxxxxxx",
        "avatar_url": "xxxxxxxxxxx.jpg",
        "vip_type": 2,
        "uk": 123456789
}
```



## 设备码授权

由于百度不允许在非百度官方网站完成账号的授权，因此提供了一个回调函数用于引导用户完成授权。

如下所示，设备码授权分两种，第一种为二维码，第二种为设备码，推荐二维码（不用登录账号）

* 设备码：您需要将设备码告知给用户，并且指导用户打开VerificationUrl链接输入设备码和账号密码完成授权
* 二维码：通过QrcodeUrl打开为一张二维码图片，您需要指导用户通过百度类产品扫描完成授权

```go
	c := mopler.NewConfiguration(
		"xxxxxxxxxxxxxxxxxxxxxxxxx",
		"xxxxxxxxxxxxxxxxxxxxxxxxx",
		mopler.WithConfigurationAuthMethod(func(codeResp *mopler.Code) {
			// codeResp.DeviceCode      //设备码
			// codeResp.VerificationUrl //通过输入设备码完成授权
			// codeResp.QrcodeUrl       //通过扫描二维码完成授权
		}),
	)
```

如果跳过授权这一步，您只需要将AccessToken传入即可。




## 配置参数说明

由上面快速开始我们可以知道，在调用NewSdkContext之前，您需要先创建一个Configuration。NewConfiguration接收5个参数，ClientId、ClientSecret，AccessToken、TokenMode、AuthMethod（access_token是调用接口必须的参数，本sdk默认实现了设备码登录验证）。

- ClientId：您应用的AppKey。
- ClientSecret：您应用的SecretKey
- AccessToken：您通过鉴权的方式获取到的access_token， 如果设置了，那么就会跳过获取token的步骤，并且需要您自己实现过期后刷新token的方法。设置此参数后，TokenMode和AuthMethod可以在配置初始化的时候不进行传递
- TokenMode：token验证方式
- AuthMethod： 根据授权模式来引导用户授权。是一个回调函数，参数传递一个response.Code，用来指导用户完成授权，response.Code的参数说明如下：

  - DeviceCode       //设备码，可用于生成单次凭证 Access Token。
  - UserCode         //用户码。 如果选择让用户输入 user code 方式，来引导用户授权，设备需要展示 user code 给用户。
  - VerificationUrl   //用户输入 user code 进行授权的 url。
  - QrcodeUrl        //二维码url，用户用手机等智能终端扫描该二维码完成授权。
  - ExpiresIn           //deviceCode 的过期时间，单位：秒。 到期后 deviceCode 不能换 Access Token。
  - Interval            //deviceCode 换 Access Token 轮询间隔时间，单位：秒。 轮询次数限制小于 expire_in/interval。



## 基础接口

部分接口字段，由于官方Api并未说明含义，但是有返回，这里只做标识，不猜测具体含义。



### 获取用户信息

```go
u, _ := sdk.ApiUserInfo.UserInfo()
```



**返回参数**

| 参数        | 说明                                      |
| ----------- | ----------------------------------------- |
| BaiduName   | 百度帐号名称                              |
| NetdiskName | 网盘帐号                                  |
| AvatarUrl   | 头像地址                                  |
| VipType     | 会员类型，0普通用户、1普通会员、2超级会员 |
| Uk          | 用户ID                                    |
| Errno       | 错误码                                    |
| Errmsg      | 错误信息                                  |
| RequestId   | 请求Id                                    |







### 获取网盘容量信息

```go
q, _ := sdk.ApiQuota.Quota(nil)
```



**请求参数**

| 参数        | 说明                                      |
| ----------- | ----------------------------------------- |
| Checkfree   | 是否检查免费信息，0为不查，1为查，默认为0 |
| Checkexpire | 是否检查过期信息，0为不查，1为查，默认为0 |

**返回参数**

| 参数   | 说明                |
| ------ | ------------------- |
| Total  | 总空间大小，单位B   |
| Expire | 7天内是否有容量到期 |
| Used   | 已使用大小，单位B   |
| Free   | 剩余大小，单位B     |
| Errno  | 错误码              |
| Errmsg      | 错误信息                                  |
| RequestId   | 请求Id                                    |




### 获取文件列表信息

```go
list, _ := sdk.ApiFileList.FileList(&types.FileListReq{Dir: "/apps/novel"})
```



**请求参数**

| 参数   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| Dir    | 需要获取文件列表的目录，例如/apps（必填）                    |
| Order  | 排序字段：默认为name；  <br/>  time表示先按文件类型排序，后按修改时间排序；  <br/>  name表示先按文件类型排序，后按文件名称排序；<br/>  size表示先按文件类型排序，后按文件大小排序。 |
| Desc   | 默认为升序，设置为1实现降序 （注：排序的对象是当前目录下所有文件，不是当前分页下的文件） |
| Start  | 起始位置，从0开始                                            |
| Limit  | 查询数目，默认为1000，建议最大不超过1000                     |
| Web    | 值为1时，返回dir_empty属性和缩略图数据                       |
| Folder | 是否只返回文件夹，0 返回所有，1 只返回文件夹，且属性只返回path字段 |



**响应参数**

| 参数      | 说明     |
| --------- | -------- |
| Errno     | 错误码   |
| GuidInfo  |          |
| Guid      |          |
| RequestId | 请求id   |
| Errmsg    | 错误信息 |
| List      | 文件信息 |

| List参数       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| FsId           | 文件在云端的唯一标识ID                                       |
| Path           | 文件的绝对路径                                               |
| ServerFilename | 文件名称                                                     |
| Size           | 文件大小                                                     |
| ServerMtime    | 文件在服务器修改时间                                         |
| ServerCtime    | 文件在服务器创建时间                                         |
| LocalMtime     | 文件在客户端修改时间                                         |
| LocalCtime     | 文件在客户端创建时间                                         |
| Isdir          | 是否为目录，0 文件、1 目录                                   |
| Category       | 文件类型，1 视频、2 音频、3 图片、4 文档、5 应用、6 其他、7 种子 |
| Md5            | 云端哈希（非文件真实MD5），只有是文件类型时，该字段才存在    |
| DirEmpty       | 该目录是否存在子目录，只有请求参数web=1且该条目为目录时，该字段才存在， 0为存在， 1为不存在 |
| Thumbs         | 只有请求参数web=1且该条目分类为图片时，该字段才存在，包含三个尺寸的缩略图URL |
| HasMore        | 是否还有下一页，0表示无，1表示有                             |
| Cursor         | 当还有下一页时，为下一次查询的起点                           |
| Privacy        |                                                              |
| Unlist         |                                                              |
| ServerAtime    |                                                              |
| Share          |                                                              |
| Empty          |                                                              |
| OperId         |                                                              |



### 递归获取文件列表信息

```go
list2, _ := sdk.ApiFileList.FileListRecursion(&types.FileListRecursionReq{Path: "/apps/novel"})
```

**请求参数**

| 参数  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| Path  | 需要获取文件列表的目录，例如/apps（必填）                    |
| Order | 排序字段：默认为name；  <br/>  time表示先按文件类型排序，后按修改时间排序；  <br/>  name表示先按文件类型排序，后按文件名称排序；<br/>  size表示先按文件类型排序，后按文件大小排序。 |
| Desc  | 默认为升序，设置为1实现降序 （注：排序的对象是当前目录下所有文件，不是当前分页下的文件） |
| Start | 起始位置，从0开始                                            |
| Limit | 查询数目，默认为1000，建议最大不超过1000                     |
| Web   | 值为1时，返回dir_empty属性和缩略图数据                       |

**响应参数**

| 参数      | 说明     |
| --------- | -------- |
| Errno     | 错误码   |
| GuidInfo  |          |
| Guid      |          |
| RequestId | 请求id   |
| Errmsg    | 错误信息 |
| List      | 文件信息 |

| List参数       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| FsId           | 文件在云端的唯一标识ID                                       |
| Path           | 文件的绝对路径                                               |
| ServerFilename | 文件名称                                                     |
| Size           | 文件大小                                                     |
| ServerMtime    | 文件在服务器修改时间                                         |
| ServerCtime    | 文件在服务器创建时间                                         |
| LocalMtime     | 文件在客户端修改时间                                         |
| LocalCtime     | 文件在客户端创建时间                                         |
| Isdir          | 是否为目录，0 文件、1 目录                                   |
| Category       | 文件类型，1 视频、2 音频、3 图片、4 文档、5 应用、6 其他、7 种子 |
| Md5            | 云端哈希（非文件真实MD5），只有是文件类型时，该字段才存在    |
| DirEmpty       | 该目录是否存在子目录，只有请求参数web=1且该条目为目录时，该字段才存在， 0为存在， 1为不存在 |
| Thumbs         | 只有请求参数web=1且该条目分类为图片时，该字段才存在，包含三个尺寸的缩略图URL |
| HasMore        | 是否还有下一页，0表示无，1表示有                             |
| Cursor         | 当还有下一页时，为下一次查询的起点                           |
| Privacy        |                                                              |
| Unlist         |                                                              |
| ServerAtime    |                                                              |
| Share          |                                                              |
| Empty          |                                                              |
| OperId         |                                                              |



### 获取文件信息

```go
	fm, _ := sdk.ApiFileMeta.FileMeta(&types.FileMetaReq{
		Fsids: []int64{384256196473636},
	})
```

**请求参数**

| 参数  | 说明                     |
| ----- | ------------------------ |
| Fsids | 需要查询的文件id（必填） |



**响应参数**

| 参数      | 说明             |
| --------- | ---------------- |
| Errmsg    | 错误信息         |
| Errno     | 错误码           |
| RequestId | 请求id           |
| List      | 文件信息集合     |
| Names     | 共享目录相关信息 |



| List里的参数 | 说明 |
| ------------ | ---- |
| Category                | 文件类型，含义如下：1 视频， 2 音乐，3 图片，4 文档，5 应用，6 其他，7 种子 |
| Filename                | 文件名称                                                     |
| FsId                    | 文件的id                                                     |
| Isdir                   | 是否为目录                                                   |
| Md5                     | 文件的md5                                                    |
| OperId                  |                                                              |
| Path                    | 文件的路径                                                   |
| RverCtime               | 文件的服务器创建Unix时间戳，单位秒                           |
| ServerMtime             | 文件的服务器修改Unix时间戳，单位秒                           |
| Size                    | 文件的大小                                                   |
| Dlink                   | 文件下载地址                                                 |



### 文件搜索

```go
fsr, _ := sdk.APiFileSearch.Search(&types.FileSearchReq{Key: "test", Dir: "/apps/novel"})
```

**请求参数**

| 参数      | 说明                                  |
| --------- | ------------------------------------- |
| Key       | 搜索关键字（必填）                    |
| Dir       | 搜索目录，默认根目录                  |
| Page      | 页数，从1开始，缺省则返回所有条目     |
| Num       | 默认为500，不能修改                   |
| Recursion | 是否递归搜索子目录 1:是，0:否（默认） |
| Web       | 默认0，为1时返回缩略图信息            |



**响应参数**

| 参数        | 说明                             |
| ----------- | -------------------------------- |
| Errno       | 错误码                           |
| RequestId   | 请求Id                           |
| Errmsg      | 错误信息                         |
| Contentlist |                                  |
| HasMore     | 是否还有下一页，0表示无，1表示有 |
| List        | 搜索到的内容                     |

| List里面的参数 | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| FsID           | 文件Id                                                       |
| Path           | 文件路径                                                     |
| ServerFilename | 服务器文件名称                                               |
| Size           | 文件大小                                                     |
| ServerMtime    | 服务器文件修改时间                                           |
| ServerCtime    | 服务器文件创建时间                                           |
| LocalMtime     | 本地文件修改时间                                             |
| LocalCtime     | 本地文件修改时间                                             |
| Isdir          | 是否是目录                                                   |
| Category       | 文件类型，含义如下：1 视频， 2 音乐，3 图片，4 文档，5 应用，6 其他，7 种子 |
| Share          |                                                              |
| OperID         |                                                              |
| ExtentTinyint1 |                                                              |
| Md5            | 文件的md5                                                    |
| Thumbs         | 缩略图地址                                                   |
| DeleteType     |                                                              |
| OwnerId        |                                                              |
| Wpfile         |                                                              |



### 文件上传

文件上传默认会递归上传当前文件夹下面的所有文件，可以同时指定多个文件或文件夹，远程路径必须使用绝对路径

```go
	reqs := []*types.UploadReq{
		{
			SrcPath:    "C:\\Users\\76680\\Desktop\\test.msi",
			RemotePath: "/apps/novel",
		},
		{
			SrcPath:    "C:\\Users\\76680\\Desktop\\e",
			RemotePath: "/apps/novel",
		},
		{
			SrcPath:    "./test",
			RemotePath: "/apps/novel",
		},
	}

	_, err = sdk.ApiUpload.Upload(reqs, func(sfs types.SuperfileStatu, count int, complete int) {
		log.Printf("正在上传：%v ，上传进度：%.0f%%", sfs.Slicing.Name, float32(complete)/float32(count)*100)
	})
	if err != nil {
		log.Println(err)
	}
```



**请求参数**

| 参数               | 说明                                 |
| ------------------ | ------------------------------------ |
| reqs               | 上传文件需要传入的参数，切片（必填） |
| superfileStatuFunc | 上传切片的时候触发的回调             |

| Reqs里面的参数 | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| SrcPath        | 本地需要上传的文件/文件夹路径（必填）                        |
| BlockSize      | 分片大小，当文件大于4MB时，文件需要分片上传，因此这里指定分片大小，普通4M，超级会员32M |
| RemotePath     | 网盘的路径，根路径为/，应用的根路径为/apps/应用名（必填）    |
| Rtype          | 文件冲突时的策略 <br/>1 表示当path冲突时，进行重命名 <br/>2 表示当path冲突且block_list不同时，进行重命名<br/>3 当云端存在同名文件时，对该文件进行覆盖<br/>默认会对上传文件进行重复检验，如果重复则加上时间戳 |

| superfileStatuFunc的参数 | 说明                                             |
| ------------------------ | ------------------------------------------------ |
| SuperfileStatu           | 切片文件状态信息，结构体                         |
| Current                  | SuperfileStatu的参数，正在上传的分片编号         |
| Err                      | SuperfileStatu的参数，上传失败时的错误信息       |
| Slicing                  | SuperfileStatu的参数，正在上传的分片信息，结构体 |

| Slicing的参数 | 说明           |
| ------------- | -------------- |
| Id            | 切片的序号     |
| Name          | 文件名称       |
| Path          | 切片的本地路径 |
| Size          | 切片的大小     |
| Md5           | 切片的md5值    |



**返回参数**

返回的为一个切片，下面为切片内元素的参数

| 参数           | 说明                                                       |
| -------------- | ---------------------------------------------------------- |
| Errno          | 错误码                                                     |
| FsId           | 文件在云端的唯一标识ID                                     |
| Md5            | 文件的MD5，只有提交文件时才返回，提交目录时没有该值        |
| ServerFilename | 文件名                                                     |
| Category       | 分类类型, 1 视频 2 音频 3 图片 4 文档 5 应用 6 其他 7 种子 |
| Path           | 上传后使用的文件绝对路径                                   |
| Size           | 文件大小，单位B                                            |
| Ctime          | 文件创建时间                                               |
| Mtime          | 文件修改时间                                               |
| Isdir          | 是否目录，0 文件、1 目录                                   |
| Name           |                                                            |
| FromType       |                                                            |



### 下载文件

下载文件有三种方式：根据文件id，下载指定文件路径（不会下载子目录文件），递归下载指定文件路径



#### 根据文件Id下载文件

```go
_ = sdk.ApiDownload.DownloadByFsId("temp", []int64{763443200151440}, nil)
```

**请求参数**

| 参数              | 说明                   |
| ----------------- | ---------------------- |
| downloadDir       | 指定存放下载文件的路径 |
| fsids             | 需要下载的文件id       |
| downloadStatuFunc | 下载时回调             |

| downloadStatuFunc的参数 | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Category                | 文件类型，含义如下：1 视频， 2 音乐，3 图片，4 文档，5 应用，6 其他，7 种子 |
| Filename                | 文件名称                                                     |
| FsId                    | 文件的id                                                     |
| Isdir                   | 是否为目录                                                   |
| Md5                     | 文件的md5                                                    |
| OperId                  |                                                              |
| Path                    | 文件的路径                                                   |
| RverCtime               | 文件的服务器创建Unix时间戳，单位秒                           |
| ServerMtime             | 文件的服务器修改Unix时间戳，单位秒                           |
| Size                    | 文件的大小                                                   |
| Dlink                   | 文件下载地址                                                 |



**响应参数**

| 参数 | 说明           |
| ---- | -------------- |
| err  | 下载的报错信息 |






#### 根据路径下载文件

```go
_ = sdk.ApiDownload.DownloadByPath("temp", "/apps/novel", nil)
```

**请求参数**

| 参数              | 说明                           |
| ----------------- | ------------------------------ |
| downloadDir       | 指定存放下载文件的路径         |
| path              | 需要下载的路径（网盘绝对路径） |
| downloadStatuFunc | 下载时回调                     |

| downloadStatuFunc的参数 | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Category                | 文件类型，含义如下：1 视频， 2 音乐，3 图片，4 文档，5 应用，6 其他，7 种子 |
| Filename                | 文件名称                                                     |
| FsId                    | 文件的id                                                     |
| Isdir                   | 是否为目录                                                   |
| Md5                     | 文件的md5                                                    |
| OperId                  |                                                              |
| Path                    | 文件的路径                                                   |
| RverCtime               | 文件的服务器创建Unix时间戳，单位秒                           |
| ServerMtime             | 文件的服务器修改Unix时间戳，单位秒                           |
| Size                    | 文件的大小                                                   |
| Dlink                   | 文件下载地址                                                 |

**响应参数**

| 参数 | 说明           |
| ---- | -------------- |
| err  | 下载的报错信息 |



#### 根据路径递归下载文件

```go
_ = sdk.ApiDownload.DownloadByPathRecursion("temp", "/apps/novel",nil)
```

**请求参数**

| 参数              | 说明                           |
| ----------------- | ------------------------------ |
| downloadDir       | 指定下载目录                   |
| path              | 需要下载的路径（网盘绝对路径） |
| downloadStatuFunc | 下载时回调                     |

| downloadStatuFunc的参数 | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Category                | 文件类型，含义如下：1 视频， 2 音乐，3 图片，4 文档，5 应用，6 其他，7 种子 |
| Filename                | 文件名称                                                     |
| FsId                    | 文件的id                                                     |
| Isdir                   | 是否为目录                                                   |
| Md5                     | 文件的md5                                                    |
| OperId                  |                                                              |
| Path                    | 文件的路径                                                   |
| RverCtime               | 文件的服务器创建Unix时间戳，单位秒                           |
| ServerMtime             | 文件的服务器修改Unix时间戳，单位秒                           |
| Size                    | 文件的大小                                                   |
| Dlink                   | 文件下载地址                                                 |

**响应参数**

| 参数 | 说明           |
| ---- | -------------- |
| err  | 下载的报错信息 |



### 创建网盘文件夹

```go
ufr, err := sdk.ApiFileManager.CreateDir("7777", "/apps/novel")
```

**请求参数**

| 参数       | 说明                 |
| ---------- | -------------------- |
| dirname    | 目录名称             |
| remotePath | 需要在哪个路径下创建 |

**响应参数**

| 参数           | 说明                                                       |
| -------------- | ---------------------------------------------------------- |
| Errno          | 错误码                                                     |
| FsId           | 文件在云端的唯一标识ID                                     |
| Md5            | 文件的MD5，只有提交文件时才返回，提交目录时没有该值        |
| ServerFilename | 文件名                                                     |
| Category       | 分类类型, 1 视频 2 音频 3 图片 4 文档 5 应用 6 其他 7 种子 |
| Path           | 上传后使用的文件绝对路径                                   |
| Size           | 文件大小，单位B                                            |
| Ctime          | 文件创建时间                                               |
| Mtime          | 文件修改时间                                               |
| Isdir          | 是否目录，0 文件、1 目录                                   |
| Name           |                                                            |
| FromType       |                                                            |



### 删除文件/文件夹

```go
	fd, err := sdk.ApiFileManager.FileDelete(&types.FileManagerReq{
		Async: "1",
		Filelist: []types.FileManagerInfo{
			{
				Path: "/apps/novel/7777",
			},
		},
		Ondup: "",
	})
```

**请求参数**

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Async    | 0 同步，1 自适应，2 异步                                     |
| Ondup    | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |
| Filelist | 需要操作的文件/文件夹集合                                    |

| Filelist里的参数 | 说明                      |
| ---------------- | ------------------------- |
| Path             | 需要删除的文件/文件夹路径 |



**响应参数**

| 参数      | 说明                 |
| --------- | -------------------- |
| Errno     | 错误码               |
| RequestId | 请求id               |
| Errmsg    | 错误信息             |
| Taskid    | 异步任务id           |
| Info      | 文件信息，结构体切片 |
| Errno     | Info里的参数，错误码 |
| Path      | 已经处理的文件路径   |



### 重命名文件/文件夹

```go
fr, err := sdk.ApiFileManager.FileRename(&types.FileManagerReq{
		Async: "1",
		Filelist: []types.FileManagerInfo{
			{
				Path:    "/apps/novel/aaa",
				Newname: "哈哈",
			},
		},
		Ondup: "",
	})
```

**请求参数**

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Async    | 0 同步，1 自适应，2 异步                                     |
| Ondup    | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |
| Filelist | 需要操作的文件/文件夹集合                                    |

| Filelist里的参数 | 说明                        |
| ---------------- | --------------------------- |
| Path             | 需要重命名的文件/文件夹路径 |
| Newname          | 新的名称                    |

**响应参数**

| 参数      | 说明                 |
| --------- | -------------------- |
| Errno     | 错误码               |
| RequestId | 请求id               |
| Errmsg    | 错误信息             |
| Taskid    | 异步任务id           |
| Info      | 文件信息，结构体切片 |
| Errno     | Info里的参数，错误码 |
| Path      | 已经处理的文件路径   |



### 移动文件/文件夹

```go
	fm, err := sdk.ApiFileManager.FileMove(&types.FileManagerReq{
		Async: "1",
		Filelist: []types.FileManagerInfo{
			{
				Path:    "/apps/novel/aaa",
				Dest:    "/apps/novel/bbb",
				Newname: "bbb",
				Ondup:   "fail",
			},
		},
		Ondup: "",
	})
```

**请求参数**

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Async    | 0 同步，1 自适应，2 异步                                     |
| Ondup    | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |
| Filelist | 需要操作的文件/文件夹集合                                    |

| Filelist里的参数 | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Path             | 需要移动的文件/文件夹路径                                    |
| Newname          | 新的文件/文件夹名称                                          |
| Dest             | 移动到此路径下                                               |
| Ondup            | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |

**响应参数**

| 参数      | 说明                 |
| --------- | -------------------- |
| Errno     | 错误码               |
| RequestId | 请求id               |
| Errmsg    | 错误信息             |
| Taskid    | 异步任务id           |
| Info      | 文件信息，结构体切片 |
| Errno     | Info里的参数，错误码 |
| Path      | 已经处理的文件路径   |



### 复制文件/文件夹

```go
	fc, err := sdk.ApiFileManager.FileCopy(&types.FileManagerReq{
		Async: "1",
		Filelist: []types.FileManagerInfo{
			{
				Path:    "/apps/novel/aaa",
				Dest:    "/apps/novel",
				Newname: "bbb",
				Ondup:   "fail",
			},
		},
		Ondup: "",
	})
```

**请求参数**

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Async    | 0 同步，1 自适应，2 异步                                     |
| Ondup    | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |
| Filelist | 需要操作的文件/文件夹集合                                    |

| Filelist里的参数 | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Path             | 需要复制的文件/文件夹路径                                    |
| Newname          | 新的文件/文件夹名称                                          |
| Dest             | 复制到此路径下                                               |
| Ondup            | 遇到重复文件的处理策略,<br/>fail(默认，直接返回失败)、newcopy(重命名文件)、overwrite（覆盖）、skip（跳过） |

**响应参数**

| 参数      | 说明                 |
| --------- | -------------------- |
| Errno     | 错误码               |
| RequestId | 请求id               |
| Errmsg    | 错误信息             |
| Taskid    | 异步任务id           |
| Info      | 文件信息，结构体切片 |
| Errno     | Info里的参数，错误码 |
| Path      | 已经处理的文件路径   |

