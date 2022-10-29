# State와 Props

## State
state는 기본적으로 데이터가 저장되는 곳이다. React 애플리케이션에 값이 바뀔 데이터들을 담아줄 수 있다.<br>

먼저 다음과 같은 코드를 보자.
```jsx
<script type="text/babel">
    const root = document.getElementById("root");
    let counter = 0;
    function countUp() {
        counter = counter + 1;
    }
    const Container = () => (
        <div>
            <h3>Total Clicks: {counter}</h3>
            <button onClick={countUp}>Click me!</button>
        </div>
    );
    ReactDOM.render(<Container />, root);
</script>
```

의도한 것은 버튼을 클릭했을 때 화면의 `Total Clicks: {counter}`가 바뀌는 것을 기대했었다. 하지만 counter의 값은 증가하지만 화면이 바뀌지 않는 것을 확인할 수 있었다.

왜냐하면 우리가 페이지를 로드할 때 `function countUp()`과 `Container`는 바로 실행되지 않고 `ReactDOM.render()`가 바로 실행되기 때문이다. 

처음에 `render` 함수가 호출될 때는 `counter`의 값이 0이고 `Container`를 렌더링 하는데 이때 `Container`는 0이라는 `counter` 값을 가지고 있다.

그러면 다음과 같은 코드로 작성해보자.
```jsx
function countUp() {
    counter = counter + 1;
    ReactDOM.render(<Container />, root);
}
```
이렇게 하면 당연히 페이지를 다시 렌더링하기 때문에 동작한다.<br>
>그런데 여기서 React의 특징을 보기위해 브라우저에서 개발자모드를 키고 요소들을 보면 html 전체를 리렌더링 하지 않고 바뀐 부분만 다시 렌더링 하는 것을 볼 수 있다. <br>

하지만 이는 렌더링을 다시 해주는 함수를 매번 호출해줘야 하는 문제가 있다. 이때 사용하는 것이 `state`이다.

그전에 `state`가 어떻게 생겼는지 보자.
```jsx
const root = document.getElementById("root");
    
function App() { 
    const data = React.useState(0);
    console.log(data);
    return (
        <div>
            <h3>Total Clicks: 0</h3>
            <button>Click me!</button>
        </div>
    )
}

ReactDOM.render(<App />, root);
```
```
(2) [0, ƒ]
```
콘솔에는 위와 같이 배열이 나오게 된다.<br>
배열의 첫 번째 값은 초기값이고 두 번째 값은 첫 번째 요소를 바꾸는 함수이다.

이를 이용해서 아래와 같이 코드를 작성하면
```jsx
function App() { 
    const [counter, modifier] = React.useState(0);
    const onClick = () => {
        modifier(counter + 1);
    }
    return (
        <div>
            <h3>Total Clicks: {counter}</h3>
            <button onClick={onClick}>Click me!<button>
        </div>
    )
}
```

버튼을 클릭했을 때 우리가 따로 리렌더링하는 함수를 호출하지 않아도 값이 바뀌는 것을 확인할 수 있다. 즉, state가 바뀌면 알아서 rerendering이 일어나는 것을 확인할 수 있다.

그런데 이런 방법은 `counter`의 값이 다른 곳에서 변경될 가능성이 존재한다. 이전 값을 바탕으로 현재 값을 설정하고 싶다면 다음과 같이 작성한다.

```jsx
const onClick = () => {
    modifier((current) => current + 1);
}
```
`modifier` 함수에 함수를 전달할 수 있는데 해당 함수의 첫 번째 argument는 현재 값이 되고 반환 값은 새로운 state를 의미한다.

## Props
`props`는 부모 컴포넌트로부터 자식 컴포넌트에 데이터를 보낼 수 있는 방법이다.

```jsx
function SaveBtn() {
    return <button style={{
        backgroundColor: "tomato",
        color: "white",
        padding: "10px 20px",
        border: 0,
        borderRadius: 10
    }}>Save Changes</button>
}

function ConfirmBtn() {
    return <button>Confirm</button>
}

function App() {

    return (
        <div>
            <SaveBtn />
            <ConfirmBtn />
        </div>
    );
}
```

다음과 같은 코드가 있을 때 여러 버튼에 동일한 스타일을 적용하고 싶다고 가정하자. 오로지 버튼의 텍스트만 수정하고 싶다고 했을 때 props를 이용해 간단하게 설정할 수 있다.

```jsx
function Btn(props) {
    return <button style={{
        backgroundColor: "tomato",
        color: "white",
        padding: "10px 20px",
        border: 0,
        borderRadius: 10
    }}>{props.text}</button>
}

function ConfirmBtn() {
    return <button>Confirm</button>
}

function App() {

    return (
        <div>
            <Btn text="Save Changes"/>
            <Btn text="Confirm"/>
        </div>
    );
}
```

이와 같이 자식 컴포넌트에게 데이터를 전달함으로써 간결하게 코드를 작성할 수 있다.

## React.memo()
React에 또다른 기능인 memo에 대해 알아만 두자.

React는 컴포넌트를 렌더링 한 후 이전 렌더링된 결과와 비교해서 DOM 업데이트를 결정한다. 결과가 다르다면 DOM을 업데이트 한다.

위의 코드의 렌더링을 알아보기 위해 `console.log`를 이용해 보자.
`Btn` 컴포넌트에 `console.log(props.text + " rendered");`를 사용해 렌더링 결과를 출력하면 다음과 같다.

```jsx
function Btn(props) {
    return <button 
        console.log(props.text + " rendered");
        onClick={props.onClick} 
        style={{
            backgroundColor: "tomato",
            color: "white",
            padding: "10px 20px",
            border: 0,
            borderRadius: 10
        }}>
        {props.text}
    </button>
}

function App() {
    const [value, setValue] = React.useState("Save Changes");
    const changeValue = () => setValue("Revert Changes");
    return (
        <div>
            <Btn text={value} onClick={changeValue} />
            <Btn text="Confirm" />
        </div>
    );
}
```

```
Save Changes rendered
Confirm rendered
Revert Changes rendered
Confirm rendered
```

이때 `React.memo()`를 사용해 `Btn` 컴포넌트를 래핑하면 React는 컴포넌트를 렌더링하고 결과를 메모이징한다. 후에  컴포넌트의 `props`가 같다면 React는 메모이징한 내용을 재사용한다.

```jsx
const MemorizedBtn = React.memo(Btn);
function App() {
    const [value, setValue] = React.useState("Save Changes");
    const changeValue = () => setValue("Revert Changes");
    return (
        <div>
            <MemorizedBtn text={value} onClick={changeValue} />
            <MemorizedBtn text="Confirm" />
        </div>
    );
}
```
```
Save Changes rendered
Confirm rendered
Revert Changes rendered
```

변하지 않는 `props`에 대해서는 메모이징해놓은 `MemorizedBtn`을 사용하고 있는 것을 확인할 수 있다.

## 참고자료
[NomadCoder - React로 영화 웹 서비스 만들기](https://nomadcoders.co/react-for-beginners/lobby)