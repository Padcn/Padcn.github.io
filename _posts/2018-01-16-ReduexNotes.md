# Redux 
> https://www.imooc.com/video/13207 教程视频
> http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html 阮一锋
flux:




## reducer:
1. 是响应的抽象
2. 是纯方法，根据输入返回输出，方法中不依赖比如时间之类
3. 传入旧的状态state和action
4. 返回新的状态

## Store

一个保存数据的地方，整个应用只有一个Store
redux提供createStore这个函数来生成Store

1. action作用在store上 
2. reducer根据store响应
3. store是唯一的
4. store包含了完整的state
5. state完全可以预测

```jsx
import {createStore} from 'redux';
const store=createStore(fn);
```
# State
一个State对应以额View。只要State相同，view就相同，反之亦然。 
某个时间点Store的快照，的数据集合叫做state

```jsx
import {createStore} from 'redux';
const store=createStore(fn);

const state=store.getState();
```

# Action
Action就是View发出的通知，通知State发生变化

```jsx
const action={
	type:'ADD_TODO',
    payload:'Learn Redux'
}
```
Action描述当前发生的事情，改变State的唯一办法，就是使用Action.他会运送数据到Store。
## container和componenet

||container|component|
|--|--|--|
|目的|如何工作(数据获取,状态更新)|如何显示(样式，布局)|
|是否在redux数据流中|是|否|
|读取数据|从redux获取state|从props获取数据|
|修改数据|向redux派发actions|从props调用回调函数|
|实现方式|由react-redux生成|手写|


