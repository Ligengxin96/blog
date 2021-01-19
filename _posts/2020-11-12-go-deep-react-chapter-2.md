---
title: '深入React技术栈第2章总结'
tags:
  - thinking
  - learning
  - reading
categories:
  - thinking
---
&emsp;&emsp;深入React技术栈第2章总结
 
## 1.React 的事件机制
&emsp;&emsp;React有自己的事件机制，所以你在React使用事件函数比如在onClick的时候，你会发现他的写法是驼峰式命名,而原生的事件直接就是onclick.而且在常用的事件,比如onclick的回调函数中，获取的event并不是一个原生事件，打印出来是一个SyntheticEvent对象(__proto__: SyntheticEvent).当然如果需要使用原生事件,通过使用SyntheticEvent事件的nativeEvent属性来获取.看完这个小节后我问了自己一个问题.

* React为什么要封装自己的事件机制,有什么好处呢?
  1. 最主要的原因还是因为需要跨浏览器兼容,Java 之所以兼容全平台就是因为代码其实不是跑直接跑在机器上的,而是跑在机器上运行的JVM上的.所以可以把SyntheticEvent对象比喻成JVM,这样就能在任何机器(浏览器)处理相同的代码(事件)
  2. 性能优化,React 的事件机制把**几乎**所以事件绑定在document对象上,而不像原生事件是在DOM对象本身,所以简化了DOM事件的处理机制,减少了内存开销.
***几乎所有事件都代理到了 document，说明有例外，比如audio、video标签的一些媒体事件（如 onplay、onpause 等），是 document 所不具有，这些事件只能够在这些标签上进行事件进行代理，但依旧用统一的入口分发函数（dispatchEvent）进行绑定*** 

## 2.CSS 相关介绍
&emsp;&emsp; 不知道为什么对CSS相关的内容我有一种抗拒感,就简单聊聊学到的一个应该有用知识点
* 实现 CSS 与 JavaScript 变量共享

可以用 :export 关键字可以把 CSS 中的变量输出到 JavaScript 中，例如：
```jsx
/* config.scss */

$primary-color: #f40;
:export {
 primaryColor: $primary-color;
}
/* app.js */
import style from 'config.scss';
// 会输出 #F40
console.log(style.primaryColor);
```

## 3.组件通信
* 父组件向子组件通信
1. 这个无需多言 props
2. 当嵌套多层组件后使用,层层传递props显得不是很优雅,所以可以使用context,使用 context 比较好的场景是真正意义上的全局信息且不会更改，例如界面主题、用户信息等. [Context使用实例](#6context使用实例)

* 子组件向父组件通信
1. 可以使用回调函数
2. 使用自定义事件机制

* 无嵌套关系组件通信
1. 使用自定义事件机制
1. 借助Node.js的Events模块来实现发布/订阅模式进行通信

## 4.HOC(高阶组件)
* HOC可以用来管理组件的公共逻辑,我认为这是一个很重要的技能,用的好的话可以让你的代码更加简洁并且更加易于维护,~~当然还有我觉得这个很Cool~~不过奈何目前我经验不是很足够,而且目前在职工作并不写前端,我也只限于会使用所以自己的理解能讲的也不是很多,待后期用得够熟练有自己的理解后再来填坑.

## 5.性能优化

&emsp;&emsp;这里主要聊聊React方面可以做到的性能优化,其他的性能优化可以参考[JavaScript高级程序设计第4章总结](/posts/go-deep-react-chapter-2)

* 重写ShouldComponentUpdate生命周期函数

使用ShouldComponentUpdate这个生命周期方法来减少组件重新Render次数是一个常见的性能优化方法.一个用下面的简单代码举个例子,P组件下有C1和C2两个子组件,C1只得到了P组件的state中countC1这个值,而C2只得到了P组件的state中countC2这个值,当不手动重写ShouldComponentUpdate方法话,当P组件的State的只改变了countC1的值改变都会导致C1和C2都重新Render,尽管C2组件状态没任何变化

```jsx
// P组件的State
this.state = {
  countC1: 0, // 只传递给C1组件
  countC2: 0, // 只传递给C2组件
}
```

所以要想C2组件避免不必要的渲染,只需要给C2组件重写shouldComponentUpdate生命周期函数,这样当P组件的State的只改变了countC1的值的时候就不会导致C2组件重新Render了.

```jsx
// C2组件中添加的shouldComponentUpdate生命周期函数
shouldComponentUpdate(nextProps) {
  return this.props.countC2 !== nextProps.countC2;
}
```

当然也可以使用PureComponent,让你的组件不继承React.Component而是继承React.PureComponent来简化你的代码,React.PureComponent会隐式的重写shouldComponentUpdate来进行浅比较.但是浅比较存在一个问题,如果P组件中的state中的一个属性是一个数组,有一个操作只是向这个数组中push了一个数据,但是这个数组的地址是没有改变的,所以浅比较无法检测到state的改动,导致不会重新Render.

为了性能优化而引入一个bug这并不是一个正确的方法.当然你这个时候应该会想到的就是使用深比较this.props和nextProps,但是这个时候如果这个引用数据类型深度很深怎么办,深比较将耗费大量时间.所以这个时候就需要引入不可变值: immutable.js这个库

* 不可变值immutable.js

immutable 对象就是一旦创建，就不能再被更改的数据。对 immutable 对象的任何修改或添加删除操作都会返回一个新的 immutable 对象接下来简单介绍下 immutable.js 这个优点

1. immutable.js 降低了 mutable 带来的复杂度

```js
function touchAndLog(touchFn) {
  let data = { key: 'value' };
  touchFn(data);
  console.log(data.key); // 输出结果是什么?
}
```

在不使用 immutable 和你不知道 touchFn 函数源码的情况下,你永远也不会知道输出的结果是什么,但是你使用 immutable ,那么永远输出的是 `value`, 而 touchFn 函数还是会得到相同的结果

2. 节省内存

immutable 会尽量复用内存,没有被引用的对象会被垃圾回收,可以看下如下的代码。

```js
import { Map } from 'immutable';
let a = Map({
  select: 'users',
  filter: Map({ name: 'Cam' })
})
let b = a.set('select', 'people');

// 以下断言都是正确的
assert.equal(a === b, false); // 因为 b 是 a 修改数据后返回的新的 immutable 对象,所以这里直接 === 比较是 false
assert.equal(a.get('filter') === b.get('filter'), true);  // 而这里是 true 就证明了复用了内存所以共享了这部分数据
```

好了,接下来我们就来看看如何 shouldComponentUpdate 使用 immutable 吧,这里还需要介绍下is 这个 api,免得对下面的demo有什么疑问

```js
// map1 和map2 都是 immutable对象
import { Map, is } from 'immutable';
const map1 = Map({ a: 1, b: 1, c: 1 });
const map2 = Map({ a: 1, b: 1, c: 1 }); 

// 以下断言都是正确的
assert.equal(map1 !== map2, true);
assert.equal(Object.is(map1, map2), false);
assert.equal(is(map1, map2), true);
```

```jsx
// 这样做的话需要用 immutable 封装一次js原生的 props 和 state 对象
import { is } from 'immutable';

shouldComponentUpdate (nextProps = {}, nextState = {}) => {
  const thisProps = this.props || {}, thisState = this.state || {};

  if (Object.keys(thisProps).length !== Object.keys(nextProps).length 
      || Object.keys(thisState).length !== Object.keys(nextState).length
     ) {
    return true;
  }

  for (const key in nextProps) {
    if (thisProps[key] !== nextProps[key] || !is(thisProps[key], nextProps[key])) {
      return true;
    }
  }

  for (const key in nextState) {
    if (thisState[key] !== nextState[key] || !is(thisState[key], nextState[key])) {
      return true;
    }
  }
  return false;
}
```

## 6.Context使用实例

话不多说直接上代码

```jsx
// 父组件代码
import React from 'react';
import { Tabs } from 'antd';
import useFetchUploadAccess from '@/hooks/useFetchUploadAccess';
import UploadCommonScript from './uploadCommonScript';
import UploadRoleScript from './uploadRoleScript';

const { TabPane } = Tabs;

 // 在父组件(顶层组件)创建一个 Context  [] 是这个Context的默认值
export const UploadAccessInfoContext = React.createContext([]);

const UploadScript = ({ getCommonFormValue, geRoleFormValue }) => {
  const [uploadAccessInfo] = useFetchUploadAccess();
  
  return (
    <>
      <Tabs defaultActiveKey="1" size="large" centered>
        <TabPane tab="tab1" key="1">
         { /* UploadAccessInfoContext.Provider 包装下子组件 value 就是你传递给这个父组件下面所以子组件的props */}
          <UploadAccessInfoContext.Provider value={[uploadAccessInfo]}>
            <UploadCommonScript ref={getCommonFormValue} />
          </UploadAccessInfoContext.Provider>
        </TabPane>
        { /* ... */}
      </Tabs>
    </>
  );
};

export default UploadScript;
```

```jsx
// 子组件代码
import React, { useContext } from 'react';
import AliOSSUpload from '@/components/Common/AliOSSUpload';
import { UploadAccessInfoContext } from '声明了Context组件(定成父组件的)的路径';


const UploadCommonScript = (props) => {
  // 使用useContext就能直接取到顶层父组件传递下来的props 避免了层层传递props的麻烦
  const [uploadAccessInfo] = useContext(UploadAccessInfoContext);
  return (
    <>
      <AliOSSUpload {...props} uploadAccessInfo={uploadAccessInfo} />
    </>
  );
};

export default UploadCommonScript;

```

## 7.Ref
1. [React Document](https://reactjs.org/docs/handling-events.html)
2. [immutable-js docs](https://immutable-js.github.io/immutable-js/docs/#/)
3. [博客园](https://www.cnblogs.com/forcheng/p/13187388.html)
4. [掘金](https://juejin.cn/post/6846687604130185230#heading-1)