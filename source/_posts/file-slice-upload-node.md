---
title: 文件分片上传——Node.js部分
date: 2018-08-23 20:52:33
tags: [upload, 分片上传, nodejs]
---

## 单请求文件上传

对于文件上传，我们通常的做法是使用单个请求来上传所有的文件内容，这样的做法在上传较小文件时比较适用。但是，当上传的文件达到百兆甚至GB级别时，使用单请求来上传文件就会遇到一些问题。

首先，请求超时时间和请求体大小的问题。我们通常需要在服务器给请求统一设置更大的请求体大小限值和超时时间限值，这对其他请求和服务器安全是十分不友好的。

再者，用户上传过程中的网络波动也容易造成大文件的上传失败，而单请求上传文件是不具备断点续传的条件的，一次上传失败后第二次仍然需要从头开始上传，比较反人类。

所以才出现了文件分片上传的概念。

## 分片上传原理

其实文件分片上传的原理很好理解，用一句话来说就是：**将大文件分割成固定长度的小块，然后挨个上传，后端进行拼接还原**。

那么，具体实现时，需要注意哪些细节呢？

1. 服务端需要知道当前得到的片是谁上传的哪个文件。

2. 服务端需要知道当前得到的片是第几片

3. 服务端需要知道上传的文件类型是什么（后缀）

4. 浏览器需要知道是否之前上传过？如果是，上传到了多少？接下来应该传什么给服务端

5. …

大概就这么多吧。

前端文件分片的方式采用HTML5的File API获取文件信息，并且使用form-data的形式上传给后端，在这里不做展开。

本文主要介绍如何使用Node.js实现文件分片上传的接收与合并。

demo地址：https://github.com/Ash-sc/slice-file-upload-server

## 接口字段约定

根据上面提到的细节，我们约定：前端在文件分片上传过程中，除了文件分片的blob数据之外，还需要传给后端以下字段。

key、currentChunk、totalChunk、offset、chunkSize、filename

首先是key。很好理解，每一个文件都应该有一个单独的key，服务端用以区分当前上传的片是属于哪个文件的，我采用的办法是第0个片的md5值+文件的size。

然后是currentChunk和totalChunk，表示当前上传的是第几片和总共的片数，类似于Array的index和length。

offset和chunkSize表示当前片的偏移量以及片大小，比如我每片大小为200,000b的话，那么第2片(第三个请求，因为我第一个请求是第0片)的offset和chunkSize分别为400,000、200,000。

filename 服务端用来获取文件后缀。

然后才是带有blob数据file字段。

控制台示例大致如下：

![console-eg](http://web-site-files.ashshen.cc/slice-file-upload/upload-example.png)

## Node.js实现

Node.js部分我使用的是express框架，当然你也可以使用koa2，原理都是一致的。

express下可以使用connect-multiparty作为中间件，然后使用req.body获取form-data信息。

大致流程：

1. 服务端接收到文件上传请求，并解析其参数

2. 创建一个空文件

2. 若发现currentChunk = 0 & totalChunk = 1，表示只有一个片，直接读取上传的文件buffer然后存入文件中，返回url至前端

3. 若发现有多个片，则根据req参数向文件中固定位置写入buffer

4. 直到最后一个一个片写入完毕，返回url至前端


在这里，文件读写采用的均是Node自带的文件系统fs，关于fs文件操作可以看[这篇文章](https://www.html-js.cn/details/E1CJFSes.html)。

### 关于断点续传

断点续传的原理也很简单。

服务端只需要在每次收到前端请求时，根据前端传入的key去数据库中查找，如果没有找到对应的数据，则在数据库中存入一条数据（key和currentChunk）。如果有找到数据，则返回这条数据的currentChunk+1给前端，并且更新这条记录的currentChunk。

前端根据后端返回的currentChunk，重新截取新的片给后端。

后端收到最后一个文件片信息后，删除数据库中关于该文件key的记录。

demo中我使用的是lowdb，当然你也可以用其他任何数据库。

### 上传部分源码（express）
``` js
router.post('/upload', multipartyMiddleware, function (req, res) {
  const {
    key,
    currentChunk,
    totalChunk,
    filename,
    chunkSize
  } = req.body
  const tmpBuffer = fs.readFileSync(req.files.file.path) // 上传文件的buffer信息
  const distFileName = key + '.' + filename.split('.').pop() // 生成新文件名称
  const downloadUrl = config.serverName + ':' + config.port + '/static/' + distFileName // 下载地址

  // 文件只有一片，直接保存
  if (currentChunk === '0' && totalChunk === '1') {
    fs.open(path.join(__dirname, '../static/' + distFileName), 'a', (err, fd) => {
      if (err) console.error(err)
      fs.writeSync(fd, tmpBuffer, 0, tmpBuffer.length, currentChunk * chunkSize)
      fs.closeSync(fd)
      res.status(200).json({
        errno: 0,
        data: {
          path: downloadUrl
        }
      })
    })
  } else {
    let nextIndex = 0 // 当前已经保存的片
    const savedFileInfo = db.getFileInfo(key) // 查询数据库中是否已经有记录
    if (savedFileInfo) nextIndex = savedFileInfo.savedIndex + 1 // 有记录则更新变量

    try {
      fs.open(path.join(__dirname, '../static/' + distFileName), 'a', (err, fd) => {
        if (err) console.error(err)
        fs.writeSync(fd, tmpBuffer, 0, tmpBuffer.length, currentChunk * chunkSize)
        fs.closeSync(fd)

        if (parseInt(currentChunk, 10) + 1 === parseInt(totalChunk, 10)) {
          db.deleteFileInfo(key) // 全部存取完毕 删除数据库信息
          res.status(200).json({
            errno: 0,
            data: {
              path: downloadUrl
            }
          })
        } else {
          db.saveFileInfo(key, parseInt(currentChunk, 10)) // 更新数据库信息
          res.status(200).json({
            errno: 0,
            data: nextIndex ? {
              nextIndex: nextIndex
            } : {}
          })
        }
      })
    } catch (err) {
      res.status(200).json({
        errno: 1,
        errorInfo: err
      })
    }
  }
})
```

### 不足之处：

1. 对于同一个文件，两个用户同时上传时应该互不干涉，所以，在实际应用中生成key的方式应该结合用户唯一的标识或者由后端接口生成给前端，不过后端接口生成给前端的话则难以实现刷新页面后也能断点续传。

2. 断点续传的逻辑应该更加完善，比如，用户断点续传时的第一个请求中currentChunk仍然是0，此时上传成功后后端返回新的currentChunk不应该把数据库中currentChunk更新成0，因为如果下一个请求挂了，那么用户就无法再断点续传了。

demo只是为了介绍上传原理，对于其中的漏洞不做深究。



