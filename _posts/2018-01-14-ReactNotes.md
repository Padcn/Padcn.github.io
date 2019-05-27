# React学习笔记

> 官方入门中文小案例:https://doc.react-china.org/tutorial/tutorial.html#%E6%95%99%E7%A8%8B%E7%AE%80%E4%BB%8B

父组件传递给子组件变量是通过this.props.value获取:
```js
class Child extends React.Component{
	render(){
    	return (
        	<button 
            onClick={()=>this.props.onClick()}
            >
            {this.props.value}
            ></button>
        );
    }
}
class Parent extends React.Component{
	handleClick(i){
    	console.log(i);
    }
	renderChild(i){
    	return (
        	<Child value={i} onClick={()=>this.handleClick(i)}/>
        );
    }
}
```


如果该组件只有一个render方法，那么可以考虑使用函数式定义
```js
function Square(props){
	return (
    	<button onClick={props.onClick}>{props.value}</button>
    );
}
```
> 这里onClick={proprs.onClick}不能写成onClick={proprs.onClick()}因为这种写法会在页面渲染时触发。

声明函数需要在组件类外声明，否则会报错