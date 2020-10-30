# React为什么需要Hook

![React](images/001.png)

- Component非UI逻辑复用困难
- 组件的生命周期函数不适合side effect逻辑的管理
- 不友好的Class Component

## Component非UI逻辑复用困难

对于React或者其它的基于Component的框架来说，页面是由一个个UI组件构成的。独立的组件可以在同一个项目中甚至不同项目中进行复用，这十分有利于前端开发效率的提高。可是除了UI层面上的复用，一些状态相关（stateful）或者副作用相关（side effect）的非UI逻辑在不同组件之间复用起来却十分困难。对于React来说，你可以使用高阶组件（High-order Component）或者renderProps的方法来复用这些逻辑，可是这两种方法都不是很好，存在各种各样的问题。如果你之前没有复用过这些非UI逻辑的话，我们可以先来看一个高阶组件的例子。

假如你在开发一个社交App的个人详情页，在这个页面中你需要获取并展示当前用户的在线状态，于是你写了一个叫做UserDetail的组件：
```
class UserDetail extends React.Component {
  state = {
    isOnline: false
  }

  handleUserStatusUpdate = (isOnline) => {
    this.setState({ isOnline })
  }

  componentDidMount() {
    // 组件挂载的时候订阅用户的在线状态
    userService.subscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
  }

  componentDidUpdate(prevProps) {
    // 用户信息发生了变化
    if (prevProps.userId != this.props.userId) {
      // 取消上一个用户的状态订阅
      userService.unSubscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
      // 订阅下一个用户的状态
      userService.subscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
    }
  }

  componentWillUnmount() {
    // 组件卸载的时候取消状态订阅
    userService.unSubscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
  }

  render() {
    return (
      <UserStatus isOnline={this.state.isOnline}>
    )
  }
}
```
从上面的代码可以看出其实在UserDetail组件里面维护用户状态信息并不是一件简单的事情，我们既要在组件挂载和卸载的时候订阅和取消订阅用户的在线状态，而且还要在用户id发生变化的时候更新订阅内容。因此如果另外一个组件也需要用到用户在线状态信息的话，作为一个优秀如你的程序员肯定不想简单地对这部分逻辑进行复制和粘贴，因为重复的代码逻辑十分不利于代码的维护和重构。接着让我们看一下如何使用高阶组件的方法来复用这部分逻辑：
```
// withUserStatus.jsx
const withUserStatus = (DecoratedComponent) => {
  class WrapperComponent extends React.Component {
   state = {
      isOnline: false
    }

    handleUserStatusUpdate = (isOnline) => {
      this.setState({ isOnline })
    }

    componentDidMount() {
      // 组件挂载的时候订阅用户的在线状态
      userService.subscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
    }

    componentDidUpdate(prevProps) {
      // 用户信息发生了变化
      if (prevProps.userId != this.props.userId) {
        // 取消上一个用户的状态订阅
        userService.unSubscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
        // 订阅下一个用户的状态
        userService.subscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
      }
    }

    componentWillUnmount() {
      // 组件卸载的时候取消状态订阅
      userService.unSubscribeUserStatus(this.props.userId, this.handleUserStatusUpdate)
    }

    render() {
      return <DecoratedComponent
        isOnline={this.stateIsOnline}
        {...this.props}
      />
    }
  }

  return WrapperComponent
}
```

在上面的代码中我们定义了用来获取用户在线状态的高阶组件，它维护了当前用户的在线状态信息并把它作为参数传递给被装饰的组件。接着我们就可以使用这个高阶组件来重构UserDetail组件的代码了：
```
import withUserStatus from 'somewhere'

class UserDetail {
  render() {
    return <UserStatus isOnline={this.props.isOnline}>
  }
}

export default withUserStatus(UserDetail)
```
我们可以看到使用了withUserStatus高阶组件后，UserDetail组件的代码一下子变得少了很多，现在它只需要从父级组件中获取到isOnline参数进行展示就好。而且这个高阶组件可以套用在其它任何需要获取用户在线状态信息的组件上，你再也不需要在前端维护一样的代码了。

这里要注意的是上面的高阶组件封装的逻辑和UI展示没有太大关系，它维护的是用户在线状态信息的获取和更新这些和外面世界交互的side effect，以及用户状态的存储这些和组件状态相关的逻辑。虽然看起来似乎代码很优雅，不过使用高阶组件来封装组件的这些逻辑其实会有以下的问题：

- 高阶组件的开发对开发者不友好
- 高阶组件之间组合性差: 如果嵌套多个高阶组件，例如这样的代码：withAuth(withRouter(withUserStatus(UserDetail)))。这种嵌套写法的高阶组件可能会导致很多问题，其中一个就是props丢失的问题
- 容易发生wrapper hell

和高阶组件类似，renderProps也会存在同样的问题。基于这些原因，React需要一个新的用来复用组件之间非UI逻辑的方法，所以Hook就这么诞生了。总的来说，Hook相对于高阶组件和renderProps在复用代码逻辑方面有以下的优势：

- 写法简单：每一个Hook都是一个函数，因此它的写法十分简单，而且开发者更容易理解。
- 组合简单：Hook组合起来十分简单，组件只需要同时使用多个hook就可以使用到它们所有的功能。
- 容易扩展：Hook具有很高的可扩展性，你可以通过自定义Hook来扩展某个Hook的功能。
- 没有wrapper hell：Hook不会改变组件的层级结构，也就不会有wrapper hell问题的产生。

## 组件的生命周期函数不适合side effect逻辑的管理

在上面UserDetail组件中我们将获取用户的在线状态这个side effect的相关逻辑分散到了componentDidMount，componentWillUnmount，componentDidUpdate三个生命周期函数中，这些互相关联的逻辑被分散到不同的函数中会导致bug的发生和产生数据不一致的情况。除了这个，我们还可能会在组件的同一个生命周期函数放置很多互不关联的side effect逻辑。举个例子，如果我们想在用户查看某个用户的详情页面的时候将浏览器当前标签页的title改为当前用户名的话，就需要在组件的componentDidMount生命周期函数里面添加document.title = this.props.userName这段代码，可是这段代码和之前订阅用户状态的逻辑是互不关联的，而且随着组件的功能变得越来越复杂，这些不关联而又放在一起的代码只会变得越来越多，于是你的组件逐渐变得难以测试。由此可见Class Component的生命周期函数并不适合用来管理组件的side effect逻辑。

那么这个问题Hook又是如何解决的呢？由于每个Hook都是一个函数，所以你可以将和某个side effect相关的逻辑都放在同一个函数（Hook）里面（useEffect Hook）。这种做法有很多好处，首先关联的代码都放在一起，可以十分方便代码的维护，其次实现了某个side effect的Hook还可以被不同的组件进行复用来提高开发效率。举个例子，我们就可以将改变标签页title的逻辑封装在一个自定的Hook中，如果其它组件有相同逻辑的话就可以使用这个Hook了：

```
// 自定义Hook
function useTabTitle(title) {
  React.useEffect(() => {
    document.title = title
  }, [title])
}

// UserDetail中使用useTabTitle Hook
function UserDetail = (props) => {
  useTabTitle(props.userName)
  ...
}
```

## 不友好的Class Component

其实Class Component除了生命周期函数不适合side effect的管理之外，还有一些其它的问题。

首先Class Component对开发者不友好。如果你要使用Class Component首先你得理解JS里面的this是怎么使用的。

除了对开发者不友好，Class Component对机器也很不友好。例如Class Component的生命周期函数很难被minified。
