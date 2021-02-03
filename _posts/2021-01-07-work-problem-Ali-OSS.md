---
title: '前端使用Ant design Upload组件上传文件到阿里云OSS的坑'
tags:
  - Ali-OSS
  - React.js
  - Ant Design
categories:
  - Issue
---
&emsp;&emsp;业务需求需要上传文件,那就想办法整呗,一顿操作过后选择了阿里云OSS.


## 1.问题出现

&emsp;&emsp;跟着阿里云文档一步步操作过来的,但是还是出现了一个403错误
```xml
<Error>
  <Code>SignatureDoesNotMatch</Code>
  <Message>The request signature we calculated does not match the signature you provided. Check your key and signing method.</Message>
</Error>
```

&emsp;&emsp;当时就想,这个问题很明显嘛,签名生成错误呗,应该是哪里写错了吧,回去对比下文档和代码吧.en?怎么回事,明明是完全按照文档的来啊,[文档](https://help.aliyun.com/document_detail/31988.html)的附录写着签名方法是`Signature = base64(hmac-sha1(base64(policy), AccessKeySecret))`这样生成的啊.我的确也是这样做的啊`const Signature = CryptoJS.HmacSHA1(policyBase64, secret)` 为啥我还是报错.Google找了半个小时资料愣是没找到答案.~~想骂人了~~

&emsp;&emsp;后来我发现在附录的下面有一个更多参考[JavaScript客户端签名直传](https://help.aliyun.com/document_detail/31925.html?spm=a2c4g.11186623.2.15.6d017a33df1Er2)
在这个页面里面我找到了一份参考代码---[浏览器客户端代码](http://gosspublic.alicdn.com/doc/oss-h5-upload-js-direct.zip")打开一看,好家伙这里面的签名的生成方式竟然是这样的

```js
// 完整代码可以点击浏览器客户端代码下载
var bytes = Crypto.HMAC(Crypto.SHA1, message, accesskey, { asBytes: true }); 
var signature = Crypto.util.bytesToBase64(bytes);
```

果然我一修改后,就上传成功了.[完整上传组件代码请查看附录](#2附录)

## 2.附录
自己封装了上传文件到阿里云OSS的 Ant design Upload组件的代码
```jsx
import React, { useState, useEffect } from 'react';
import { Upload, Button, message } from 'antd';
import { UploadOutlined } from '@ant-design/icons';
import _get from 'lodash/get';
import CryptoJS from 'crypto-js';
import { encrypt } from '@/utils/crypto';
import OSS_CONFIG from '@/config/OSSConfig';
import { getUploadAccess } from '@/services/uploadFile';

const AliOSSUpload = (props) => {
  const { 
    ButtonName = "点击上传文件", 
    maxFileListLength = 4, 
    onPreview, 
    listType, 
    uploadButton, 
    accept, 
    uploadAccessInfo: propsUploadAccessInfo,
  } = props;
  const [OSSData, setOSSData] =  useState({});
  const [fileList, setFileList] = useState([]);

  const onChange = (fileInfo) => {
    if (fileInfo?.file?.status === "error") {
      message.error('上传失败,请重新上传');
    }
    const { onChange: formCallback } = props;
    if (typeof formCallback === 'function') {
      formCallback(fileInfo);
    }
    setFileList(fileInfo.fileList);
  };

  const onRemove = (file) => {
    const files = fileList.filter((v) => v.url !== file.url);
  };

  const handleOSSData = (uploadAccessInfo) => {
    try {
      const expire = _get(uploadAccessInfo, 'sts_token.expiration', new Date());
      const secret = _get(uploadAccessInfo, 'sts_token.access_key_secret', '');
      const uuid = _get(uploadAccessInfo, 'script_uuid', '');
      const accessId = _get(uploadAccessInfo, 'sts_token.access_key_id', '');
      const token = _get(uploadAccessInfo, 'sts_token.security_token', '');

      const policyText = {
        expiration: expire,
        conditions: [
          { bucket: OSS_CONFIG.bucket },
        ],
      };

      const policyBase64 = encrypt(JSON.stringify(policyText));
      // must add { asBytes: true }, otherwise oss will return 403: SignatureDoesNotMatch
      const bytes = CryptoJS.HmacSHA1(policyBase64, secret, { asBytes: true });
      const signature = bytes.toString(CryptoJS.enc.Base64);

      const data = {
        dir: `${OSS_CONFIG.dir}/${uuid}/`,
        expire,
        host: OSS_CONFIG.host,
        accessId,
        policy: policyBase64,
        signature,
        token,
        successActionStatus: 200,
      };
      return data;
    } catch (error) {
      console.log(error);
    }
  };

  const init = (uploadAccessInfo) => {
    const initOSSData = handleOSSData(uploadAccessInfo);
    setOSSData(initOSSData);
  };

  const fetchUploadAccess = () => {
    getUploadAccess().then((response) => {
      if (response.success) {
        init(response.data);
      }
    }).catch((error) => {
      console.error(error);
      message.error('获取上传权限失败');
    });
  }
  
  const beforeUpload = async (file) => {
    if (OSSData.expire < Date.now()) {
      fetchUploadAccess();
    }

    const suffix = file.name.slice(file.name.lastIndexOf('.'));
    const filename = Date.now() + suffix;
    file.key = `${OSSData.dir}${filename}`;
    file.url = `${OSSData.host}/${OSSData.dir}${filename}`;

    return file;
  };

  const getExtraData = (file) => {
    return {
      key: file.key,
      OSSAccessKeyId: OSSData.accessId,
      policy: OSSData.policy,
      Signature: OSSData.signature,
      'x-oss-security-token': OSSData.token,
      success_action_status: OSSData.successActionStatus,
    };
  };

  useEffect(() => {
    if (propsUploadAccessInfo === undefined) {
      fetchUploadAccess();
    } else {
      init(propsUploadAccessInfo);
    }
  }, [propsUploadAccessInfo]);

  const uploadProps = {
    name: "file",
    fileList,
    action: OSSData.host,
    listType,
    onPreview,
    accept,
    onChange,
    onRemove,
    data: getExtraData,
    beforeUpload,
  };

  return (
    <>
      <Upload {...uploadProps} >
        {  
          listType ? (fileList.length >= maxFileListLength ? null : uploadButton) : (<Button icon={<UploadOutlined />}>{ButtonName}</Button>)
        }
      </Upload>
    </>
  );
};

export default AliOSSUpload;

```

