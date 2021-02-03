---
title: '使用Ant design Form组件获取Upload组件的fileList信息'
tags:
  - React.js
  - Ant Design
categories:
  - tutorial
  - technology
---

&emsp;&emsp;简述下我的业务场景,一个Modal组件中使用了Form组件,而这个Form组件又包含了一些别的组件,然后在点击Modal的确认按钮把Form表单的数据全部发送给Api.
[不看过程直接看代码的](#3附录)

## 1.问题出现

&emsp;&emsp;在完成一切后,我发现点击确认按钮后并不能获取到Form组件下面Upload组件的信息,当时就想应该这个组件Api是支持的,然后看了一遍Form组件和Upload组件的发现了一个[官方Demo](https://ant-design.gitee.io/components/form-cn/#components-form-demo-validate-other)然后肯定就直接Copy代码下来开整啊.但是发现不生效

## 2.解决过程

&emsp;&emsp;咦?官方Demo都不生效,怎么回事呢.仔细研究了下代码

```jsx

const normFile = (e) => {
  console.log('Upload event:', e);

  if (Array.isArray(e)) {
    return e;
  }

  return e && e.fileList;
};

<Form.Item
  name="upload"
  label="Upload"
  valuePropName="fileList"
  getValueFromEvent={normFile} 
  extra="longgggggggggggggggggggggggggggggggggg"
>
  <Upload name="logo" action="/upload.do" listType="picture">
    <Button icon={<UploadOutlined />}>Click to upload</Button>
  </Upload>
</Form.Item>

```

&emsp;&emsp;我发现这个getValueFromEvent属性是有别于别的基础组件(Input之类的),这个是特别为Upload组件新加的.然后看getValueFromEvent属性的值是normFile.我看到这个normFile是一个函数,我就反应过来了.应该是Form组件封装了一些东西可以吧normFile当回调函数传递给Upload组件.想到这里果断开始Debug,猜的没错只要Form.Item给了`getValueFromEvent={normFile}` 这个属性, Upload组件Props中会多出一个onChange回调函数这个属性.那么我们在Upload组件中使用这个onChange这个函数呢?猜了下,在Upload组件中的onChange中使用,然后我一试,果然在Form组件这个文件的normFile函数输出了文件信息.当然我的这个业务场景也没这么简单,我还需要使用到Ref.[详细代码](#3附录)

## 3.附录

&emsp;&emsp;代码非全部代码,只取了关键部分代码

```jsx
import React, { useState, useRef } from 'react';

const ScriptManagement = () => {
  // 在Modal组件的确认按钮函数的文件中定义 getCommonFormValue
  const getCommonFormValue = useRef();
  const [visible, setVisible] = useState(false);

  const handleOpenModal = () => {
    setVisible(true);
  };

  const handleCloseModal = () => {
    setVisible(false);
  };

  const handleOk = () => {
    const commonFieldsValue = getCommonFormValue?.current?.formFields?.getFieldsValue();
    console.log(commonFieldsValue);
  };

  const modalProps = {
    visible,
    onOk: handleOk,
    onCancel: handleCloseModal,
  };

  return (
    <>
      <Row>
        <List handleOpenModal={handleOpenModal} />
      </Row>
      <BasicModal {...modalProps} >
        <UploadScript getCommonFormValue={getCommonFormValue} />
      </BasicModal>
    </>
  );
};

export default ScriptManagement;

```

```jsx
// 着中间这层组件只是起到了传递Props的作用,为了完整性才展示出来的
// 注意在这一层组件中改变了getCommonFormValue在pros中的名字变成了ref
import React from 'react';

const UploadScript = (props) => {
  const { getCommonFormValue } = props;
  return (
    <>
      <Tabs defaultActiveKey="1" size="large" centered>
        <TabPane tab="tab1" key="1">
          <UploadCommonScript ref={getCommonFormValue} />
        </TabPane>
      </Tabs>
    </>
  );
};

export default UploadScript;

```

```jsx
// 这里就是Form组件所在的那个文件了
import React, { useImperativeHandle, useRef, forwardRef } from "react";
import { Form, Input, Col } from 'antd';
import AliOSSUpload from '@/components/Common/AliOSSUpload';

const UploadCommonScript = (props, ref) => {
  const [form] = Form.useForm();
  const formRef = useRef();

  // 这个也是必不可少的函数
  useImperativeHandle(ref, () => ({
    formFields: form,
  }));

  const normFile = (e) => {
    console.log('Upload event:', e);
    if (Array.isArray(e)) {
      return e;
    }
    return e && e.fileList;
  };

  return (
    <>
      <Form 
        form={form} 
        ref={formRef}
        layout="horizontal"
        name="uploadCommonScript"
      >
        <Col span={24}>
          <Form.Item label="label" name="name" getValueFromEvent={normFile}>
            <AliOSSUpload accept='.mp4' name="name" />
          </Form.Item>
        </Col>
      </Form>
    </>
  );
};

// 需要使用forwardRef包装下这个组件
const WrappedUploadCommonScript = forwardRef(UploadCommonScript);
export default WrappedUploadCommonScript;

```

```jsx
// 这里就是Upload组件所在的那个文件了
import React, { useState, useEffect } from 'react';

const AliOSSUpload = (props) => {
  const onChange = (fileInfo) => {
    if (fileInfo?.file?.status === "error") {
      message.error('上传失败,请重新上传');
    }
    // 这里需要从props里面取到onChange出来然后在Upload onChange事件中调用这个回调函数
    // 注意如果Form组件不是直接包含的Uploa组件,在着两者之间还有别的组件的话记得在中间组件中解构onChange传递给Upload组件
    const { onChange: formCallback } = props;
    if (typeof formCallback === 'function') {
      formCallback(fileInfo);
    }
    setFileList(fileInfo.fileList);
  };

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
        <Button icon={<UploadOutlined />}>{ButtonName}</Button>
      </Upload>
    </>
  );
};

export default AliOSSUpload;

```