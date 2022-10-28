# React
프론트엔드 코드를 작성할 필요를 느껴 React 학습을 시작하려고 한다.

## React란?
React는 javascipt 라이브러리로 Facebook에서 만든 오픈 소스 프로젝트이다.<br>
가장 큰 사용목적은 SPA(Single Page Application)을 위해 사용자 인터페이스를 구축하는데 있다고 생각한다.

React를 사용하려면 React와 ReactDOM을 로드할 필요가 있는데, 각각은 다음과 같다.

- React : React 최상위 API
- ReactDOM : DOM 전용 메서드 추가
    - 우리가 React라이브러리를 통해 코드를 작성하면 ReactDOM은 코드를 html 위로 올려준다고 생각


## JSX
`JSX(JavaScript XML)`는 단순히 javascript를 확장한 문법이다. <br>
브라우저는 `JSX`를 이해하지 못하기 때문에 렌더링하기 전에 Babel을 사용해서 일반적인 javascript 형태의 코드로 변환한다.

```javascript
const btn = React.createElement(
    "button",   // html tag
    {
        onClick: () => console.log("im clicked"),
    },  // proerty
    "Click me!" // content
)
```

```javascript
const Button = () => {
    <button
        onClick={() => console.log("im clicked")}
    >
        Click me!
    </button>
}
```

위와 같이 `JSX`는 실제로 HTML이 아니지만, 코드 작성 방식이 HTML과 유사하다.<br>
몇 가지 특징은 다음과 같다.
- `JSX`에서의 프로퍼티와 메서드는 CamelCase를 따른다.
- Self-closing tag들은 반드시 `/`로 끝나야 한다.

## Component
리액트는 컴포넌트 기반의 라이브러리이다. 컴포넌트는 웹 페이지에서 화면을 이루는 작은 요소들이라고 생각하자.<br>
리액트의 거의 모든 것은 컴포넌트로 이루어져 있는데, `class component`나 `simple component`들이 될 수 있다.

### class component
```javascript
class Table extends Component {
  render() {
    return (
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Job</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Charlie</td>
            <td>Janitor</td>
          </tr>
          <tr>
            <td>Mac</td>
            <td>Bouncer</td>
          </tr>
        </tbody>
      </table>
    )
  }
}
```
위와 같이 `class` 키워드를 통해서 컴포넌트를 정의하는 것을 확인할 수 있다.

### simple component
`simple component`는 `class component`와는 다르게 `class`키워드를 사용하지 않고 화살표 함수를 사용한다.
```javascript
const TableHeader = () => {
  return (
      <thead>
        <tr>
          <th>Name</th>
          <th>Job</th>
        </tr>
      </thead>
    )
}
```
```javascript
const TableBody = () => {
  return (
    <tbody>
      <tr>
        <td>Charlie</td>
        <td>Janitor</td>
      </tr>
      <tr>
        <td>Mac</td>
        <td>Bouncer</td>
      </tr>
    </tbody>
  )
}
```
```javascript
const Table = () => {
  render() {
    return (
      <table>
        <TableHeader />
        <TableBody />
      </table>
    )
  }
}
```

이렇게 컴포넌트들은 한 번 정의해두면 다른 곳에서도 재사용할 수 있다는 장점이 존재한다.

## 참고 자료
[리액트 시작하기](https://www.taniarascia.com/getting-started-with-react/)