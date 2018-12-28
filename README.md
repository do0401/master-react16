React 16 마스터하기
=========================
이 프로젝트는 [Nomad Coders](https://academy.nomadcoders.co/)의 니콜라스님 강의를 토대로 진행했습니다.

## `시작하기`
### create-react-app
`npm install -g create-react-app`
- create-react-app를 설치한다.<br>

`create-react-app [Project Name]`
- create-react-app으로 프로젝트를 생성한다.

`cd [Project Name]`<br>
`npm start`
- 프로젝트 폴더 경로로 이동하여 실행한다.

## `1일차`
### Return Types
```javascript
render() {
    return (
        <header></header>,
        <div></div>,        // 이렇게 여러 개의 element를 리턴할 수 없었다.
        <footer></footer>   // 그래서 array 작업을 하거나 하나의 element에 담아야 했다.
    )
}
```
- 이전 React에서는 컴포넌트 또는 null 만 return 할 수 있었다.
- string 을 return 하거나 2개의 element를 리턴할 수 없었다.
- 불편하고, 필요없는 코드가 너무 많았다.
```javascript
import React, { Component, Fragment } from 'react';

class ReturnTypes extends Component {
    render() {
        return (
        <Fragment>  // 또는 <>
            <header></header>,
            <div></div>,
            <footer></footer>
        </Fragment> // 또는 </>
        )
    }
}
```
- React 16에서는 Fragment를 활용하여 여러 개의 element를 return 할 수 있다.
- return 하려는 element를 `<Fragment></Fragment>`나 `<></>`로 감싸면 된다.
- span 처럼 생겼으나, span 보다 훨씬 명확하고 span 처럼 웹사이트에 들어가지 않는다.
- 그리고 React 16에서는 string 을 return 할 수 있다.

### Portals
- ReactJS 는 기본적으로 `<div id="root">`를 찾아서 마운트한다.
- Portals를 사용하면 React root 밖을 변경할 수 있다.
- 즉, Portals는 React root 밖에 React를 넣을 수 있도록 해준다.
```javascript
// index.html
<body>
<header>
    <h1>Can't touch this</h1>
    <span id="touchme"></span>
</header>
<div id="root"></div>

// App.js
import { createPortal } from 'react-dom';

class Portals extends Component {
    render() {
        return createPortal (               // Portals 사용
        <Message />,
        document.getElementById('touchme')  // 어디에 마운트 할지 알려준다.
        );
    }
}

const Message = () => "Just touched it!";
```
- Portals 에게 마운트할 곳을 알려주면 root 밖에 있는, React 밖에 있는 태그에 접근할 수 있다.
- iframe 이거나 html을 변경하지 못하거나 워드프레스 작업을 할 때 유용하다.

### Error Boundaries
- 컴포넌트로 하여금 컴포넌트 children의 error를 관리할 수 있게 해준다.
```javascript
class ErrorMaker extends Component {
    state = {
        friends: ['jisu','flynn','daal','kneeprayer']
    };
    componentDidMount = () => {
        setTimeout(() => {
        this.setState({
            friends: undefined
        });
        }, 2000);
    };
    render() {
        const {friends} = this.state;
        return friends.map(friend => ` ${friend} `);
    }
}
```
- error를 발생시킬 컴포넌트를 추가했고, 해당 error로 인해 react 앱이 실행되지 않는다.
```javascript
class App extends Component {
    componentDidCatch = (error, info) => {
        console.log(`catched ${error} the info i have is ${JSON.stringify(info)}`)
    };
    // skip
}
```
- 발생한 error를 잡기 위해 error가 발생한 컴포넌트(ErrorMaker)의 부모 컴포넌트(App)에 componentDidCatch 라이프사이클을 추가한다.
- error가 발생하면 error를 잡고 정보를 알려준다.
```javascript
const ErrorFallback = () => "Sorry something went wrong"

class App extends Component {
    state = {
        hasError: false
    };
    componentDidCatch = (error, info) => {
        console.log(`catched ${error} the info i have is ${JSON.stringify(info)}`)
        this.setState({
        hasError: true
        })
    };
    render() {
        const { hasError } = this.state;
        return ( 
        <Fragment>
            <ReturnTypes />
            <Portals />
            {hasError ? <ErrorFallback /> : <ErrorMaker />}
            <ErrorMaker />
        </Fragment>
        );
    }
}
```
- error 분기 처리를 위해 state에 hasError를 추가하고, error 발생 시 hasError 값을 true로 변경한다.
- 그러면 error가 발생하더라도 error가 발생한 컴포넌트에서만 error 메시지를 보여주고 나머지 컴포넌트는 정상적으로 동작한다.
- 이렇게 error를 구분하고 error에 대응할 수 있다.(더 프로페셔널한 개발자로 보인다!)
- 다만, 이런 작업 방식(컴포넌트마다 true/false가 있는 것)은 실제로 사용하지 않으며 단지 컨셉을 보여주기 위한 코드이다.

### Error Boundaries Higher Order Components
- 위에서 말했듯이 컴포넌트마다 true/false가 있는 방식으로 작업하고 싶지 않으므로 Higher-Order Component를 만든다.
```javascript
const BoundaryHOC = protectedComponent => 
class Boundary extends Component {
    state = {
        hasError: false
    };
    componentDidCatch = () => {
        this.setState({
        hasError: true
        });
    }
    render() {
        const { hasError } = this.state;
        if (hasError) {
        return <ErrorFallback />;
        } else {
        return <protectedComponent />;
        }
    }
};
```
- HOC는 위와 같이 만든다.(Error Boundaries에서 App 내에 추가했던 코드와 비슷하다.)
```javascript
const PErrorMaker = BoundaryHOC(ErrorMaker)
const PPortals = BoundaryHOC(Portals)

// skip
render() {
        const { hasError } = this.state;
        return ( 
        <Fragment>
            <ReturnTypes />
            <PPortals />
            <PErrorMaker />
        </Fragment>
        );
    }
```
- 그리고 보호하고자 하는 컴포넌트를 HOC에 넣어주면 된다.
- 이렇게 우리가 생성한 HOC에 의하여 컴포넌트를 보호하고, error 처리를 한다.

### this.setState(null)
- setState는 컴포넌트를 업데이트한다.(그런데 업데이트를 콘트롤할 수 있다면?!)
- 바로 setState null이 컴포넌트를 언제 업데이트할지 결정할 수 있는 기능을 제공한다.
```javascript
const MAX_PIZZAS = 20;

const eatPizza = (state, props) => {
    const { pizzas } = state;
    if (pizzas < MAX_PIZZAS) {
        return {
            pizzas: pizzas + 1
        };
    } else {
        return null;        // return null을 활용하면 업데이트를 컨트롤 할 수 있다.
    }
};

class Controlled extends Component {
    state = {
        pizzas: 0
    };
    render() {
        const { pizzas } = this.state;
        return (
        <button onClick={this._handleClick}>
            {`I have eaten ${pizzas} ${pizzas ===1 ? "pizza" : "pizzas"}`}
        </button>
        );
    }
    _handleClick = () => {
        this.setState(eatPizza);
    };
}
```
- 위 코드와 같이 return null을 활용하여 state를 죽이지 않고 업데이트를 컨트롤할 수 있다.
- 이전 react에서는 return null을 하면 state를 바꿨다.
- 하지만 React 16에서는 state를 업데이트 하지 않고, 컴포넌트를 업데이트 하지도 않는다.

<br><br>
---
This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.<br>
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.<br>
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.<br>
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.<br>
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.<br>
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (Webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: https://facebook.github.io/create-react-app/docs/code-splitting

### Analyzing the Bundle Size

This section has moved here: https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size

### Making a Progressive Web App

This section has moved here: https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app

### Advanced Configuration

This section has moved here: https://facebook.github.io/create-react-app/docs/advanced-configuration

### Deployment

This section has moved here: https://facebook.github.io/create-react-app/docs/deployment

### `npm run build` fails to minify

This section has moved here: https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify
