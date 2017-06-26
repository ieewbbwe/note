#概述
最近准备研究一下ReactNative，于是先来看一下React相关的内容，基础部分还是蛮简单的，没有想象中的复杂。本文只是做一个大致的概要记录，具体还需要去找demo自己看。

这里是我的学习地址：http://www.runoob.com/react/react-install.html
#React安装
自己去[FaceBook的仓库](https://github.com/facebook/react/releases)下载吧，主要就是几个js文件，不想下的话可以先用
```
<script src="https://cdn.bootcss.com/react/15.4.2/react.min.js"></script>
<script src="https://cdn.bootcss.com/react/15.4.2/react-dom.min.js"></script>
<script src="https://cdn.bootcss.com/babel-standalone/6.22.1/babel.min.js"></script>
```

IDE用的是WebStorm，因为之前再用的Android Studio，同一家公司的产品上手比较快

#jsx
React支持html与JavaScript混合写。

举个栗子，在react模块你可以这么写
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <script src="https://cdn.bootcss.com/react/15.4.2/react.min.js"></script>
    <script src="https://cdn.bootcss.com/react/15.4.2/react-dom.min.js"></script>
    <script src="https://cdn.bootcss.com/babel-standalone/6.22.1/babel.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      ReactDOM.render(
        <h1>Hello, world!{1+1}</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>
```

#创建组件
React支持自建组件，类似Android里面的自定义View，java里面STL。

举个例子
```
var HelloMessage = React.createClass({
  render: function() {
    return <h1>Hello World！</h1>;
  }
});
 
ReactDOM.render(
  <HelloMessage />,
  document.getElementById('example')
);
```

这样就可以在浏览器上输出一个Hello Word！

#State
一般用来做变量的定义，比如：
```
var HelloMessage = React.createClass({
 getInitialState: function() {
          return {name: 'picher'};
        },
  render: function() {
    return <h1>Hello，{name} ！</h1>;
  }
});
 
ReactDOM.render(
  <HelloMessage />,
  document.getElementById('example')
);
```

这会输出一个Hello，picher！
#Props
类似常量了，他就是组件里面的属性，比如你可以这样写：
```
var HelloMessage = React.createClass({

  render: function() {
    return <h1>Hello，{this.props.msg} ！</h1>;
  }
});
 
ReactDOM.render(
  <HelloMessage msg="Hello Props!"/>,
  document.getElementById('example')
);
```
这会在浏览器输出一行Hello Props!
#生命周期
组件的生命周期，我是这样理解的。绘制（官方应该叫挂载），更新数据，移除。

每个状态又分为即将发生和已经发生，即：

- componentWillMount 在渲染前调用,在客户端也在服务端。
- componentDidMount : 在第一次渲染后调用，只在客户端。
- componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
- componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
- componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。

另外还需要单独记两个：

- componentWillReceiveProps 在组件接收到一个新的prop时被调用。
- shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。

```
var Button = React.createClass({
  getInitialState: function() {
    return {
      data:0
    };
  },
  setNewNumber: function() {
    this.setState({data: this.state.data + 1})
  },
  render: function () {
      return (
         <div>
            <button onClick = {this.setNewNumber}>INCREMENT</button>
            <Content myNumber = {this.state.data}></Content>
         </div>
      );
    }
})
var Content = React.createClass({
  componentWillMount:function() {
      console.log('Component WILL MOUNT!')
  },
  componentDidMount:function() {
       console.log('Component DID MOUNT!')
  },
  componentWillReceiveProps:function(newProps) {
        console.log('Component WILL RECEIVE PROPS!')
  },
  shouldComponentUpdate:function(newProps, newState) {
        return true;
  },
  componentWillUpdate:function(nextProps, nextState) {
        console.log('Component WILL UPDATE!');
  },
  componentDidUpdate:function(prevProps, prevState) {
        console.log('Component DID UPDATE!')
  },
  componentWillUnmount:function() {
         console.log('Component WILL UNMOUNT!')
  },
 
    render: function () {
      return (
        <div>
          <h3>{this.props.myNumber}</h3>
        </div>
      );
    }
});
ReactDOM.render(
   <div>
      <Button />
   </div>,
  document.getElementById('example')
);
```

#Ajax
也可以进行网络请求,看Demo吧，很简单
```
var UserGist = React.createClass({
  getInitialState: function() {
    return {
      username: '',
      lastGistUrl: ''
    };
  },
 
  componentDidMount: function() {
    this.serverRequest = $.get(this.props.source, function (result) {
      var lastGist = result[0];
      this.setState({
        username: lastGist.owner.login,
        lastGistUrl: lastGist.html_url
      });
    }.bind(this));
  },
 
  componentWillUnmount: function() {
    this.serverRequest.abort();
  },
 
  render: function() {
    return (
      <div>
        {this.state.username} 用户最新的 Gist 共享地址：
        <a href={this.state.lastGistUrl}>{this.state.lastGistUrl}</a>
      </div>
    );
  }
});
 
ReactDOM.render(
  <UserGist source="https://api.github.com/users/octocat/gists" />,
  mountNode
);
```

#表单与事件
React一样可以处理一些事件，和JS里面的一样。常见的事件
- onClick
- onChange
- onScroll
- onFocus

也举个例子吧
```
var HelloMessage = React.createClass({
  getInitialState: function() {
    return {value: 'Hello Runoob!'};
  },
  handleChange: function(event) {
    this.setState({value: event.target.value});
  },
  render: function() {
    var value = this.state.value;
    return <div>
            <input type="text" value={value} onChange={this.handleChange} /> 
            <h4>{value}</h4>
           </div>;
  }
});
ReactDOM.render(
  <HelloMessage />,
  document.getElementById('example')
);
```
会同步更新输入的值
#Refs
类似于起名字，让你在别的地方能通过名字找到这个控件。
```
var MyComponent = React.createClass({
  handleClick: function() {
    // 使用原生的 DOM API 获取焦点
    this.refs.myInput.focus();
  },
  render: function() {
    //  当组件插入到 DOM 后，ref 属性添加一个组件的引用于到 this.refs
    return (
      <div>
        <input type="text" ref="myInput" />
        <input
          type="button"
          value="点我输入框获取焦点"
          onClick={this.handleClick}
        />
      </div>
    );
  }
});
 
ReactDOM.render(
  <MyComponent />,
  document.getElementById('example')
);

```
