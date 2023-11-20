# React.js

## React Elements
기존의 객체지향적인 UI Model은 render시 각 component의 instance들을 appendChild, removeChild, destroy등을 직접 조작해야 하고, 부모 컴포넌트가 자식 컴포넌트의 instance를 직접 생성하고 호출하기 때문에 결합력이 강해진다.

### What is Elements?
- Component instance를 묘사하기 위한 순수 객체
- Instance가 아니며, React가 브라우저에 렌더링하기 위해 필요한 정보를 갖고 있는 JS Object
- Element는 type, props를 갖는다.
- type: string | ReactClass
- props : object
- key, ref
- Element가 HTML Tag일 경우 type은 string이며, props는 tag의 attributes이다.
  ```jsx
  {
    type: 'button',
    props: {
      className: 'button button-blue',
      children: {
        type: 'b',
        props: {
          children: 'OK!'
        }
      }
    }
  }
  <button class='button button-blue'>
    <b>
      OK!
    </b>
  </button>
  ```
- React Component elements일 경우 type은 component 이름, 브라우저가 렌더링을 하기 위한 DOM Tree의 정보
  ```jsx
  {
    type: Button,
    props: {
      color: 'blue',
      children: 'OK!'  // 다른 element를 children으로 갖음으로써 tree 구조를 가질 수 있다.
    }
  }

  const App = ({ content }) => <div>{content}</div>
  ReactDOM.render(<App key="1" content="Deep dive react" />, container)

  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type, // function App()
    key: key, // 1
    props: props, // { content: 'deep dive react' }
    ref: ref, // undefined
  }
  ```
- ReactDOM은 props를 component에게 전달하며 return element를 요청한다.
- React의 Renderer는 React Element로부터 정보를 받아 해당 Component에게 정보를 return받는다. 이 과정은 type이 DOM Element(HTML Tag)일 때까지 반복한다.
  - -> Top-down Reconciliation
### What's benefit for doing this?
UI를 렌더링하기 위해 필요한 정보를 React Element만을 갖고 DOM과 독립적으로 UI를 관리할 수 있다. 

## Components
### React Component란?
Props를 받아 Element를 반환하는 것(Functional component 기준)

## Reconciliation

## Fiber
### Fiber Architecture
React는 v16 이전까지는 Reconciler가 Stack 구조를 갖고 렌더링을 하였는데, stack 구조는 rendering이나 작업의 순서를 바꿀 수 없었다. call stack이 비어야(React의 DOM 조작이 끝나야) browser가 렌더링을 시작했다. 이를 개선하기 위해 Fiber를 도입하였다.
### Fiber란?
- V-DOM의 노드 객체이며, React Elements의 내용이 DOM에 반영되기 위한 정보가 확장된 객체.
- React Element에 더해 컴포넌트의 state, life cycle, hook의 정보가 관리된다.
[Fiber의 구현 코드](https://github.com/facebook/react/blob/b53ea6ca05d2ccb9950b40b33f74dfee0421d872/packages/react-reconciler/src/ReactFiber.js#L255)
```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime = NoWork;
  this.childExpirationTime = NoWork;

  this.alternate = null;
```

# React 패키지 & 구성 요소
## 구성 요소
### react core
- 컴포넌트를 정의한다.
### renderer
- ReactDOM - Client의 렌더링 환경에 의존. Client와 React를 연결한다.
- reconciler에 의존성을 갖는다.
### event(legacy-events)
- SyntheticEvent - 브라우저의 Event를 wrapping한 확장된 시스템
### scheduler
- React의 비동기 task를 실행.
### reconciler
- Fiber에서 V-DOM을 재조정하고, 컴포넌트를 호출하는 곳

## Virtual DOM
실제 DOM과는 달리 메모리상에만 존재하는 가상의 DOM이며, React에서는 V-DOM을 Reconcil하는 것을 Render라고 표현한다.
### 더블 버퍼링
React는 V-DOM을 두개의 트리로 관리하는데, 이를 더블 버퍼링이라고 한다. 하나는 현재 반영된 tree로 current라고 부르며, Render phase ~ Commit phase동안 작업이 진행되는 tree를 workInProgress라고 부른다.
Commit phase가 완료되면 workInProgress tree가 Root node에 연결되어 current가 되며, 교체된 current로부터 자가 복제하여 새로운 workInProgress를 만든다.
V-DOM의 각 fiber 노드들은 current <-> workInProgress를 서로 참조하고 있다.
### What's benefit for V-DOM and double buffering?
UI의 변화가 발생할 때마다 DOM에 Push하고 브라우저가 Paint하게 되면 속도가 느려지고 자원의 소모가 많다. 이를 방지하기 위해 UI의 변화는 V-DOM에서 작업하고, 작업이 완료되면 이를 DOM에 Commit하여 한번에 Paint하는 과정을 거친다.

## Hooks
### Hooks란?

## Portal
- ReactDOM.createPortal() 로 만들 수 있다. 렌더링할 React element와 이 element가 들어갈 DOM 요소를 인자로 받는다.
- DOM 트리에서는 다른 위치에 있지만, React DOM Tree에서는 위치가 변하지는 않는다. 
### What's benefit for doing this?
- UI가 표시되는 DOM Element의 위치를 다른 곳으로 바꾸지만, 이벤트 버블링은 React DOM Tree 기반으로 발생한다.
- DOM 트리의 최상위에 위치시켜 z-index나 CSS의 적용에 예외를 둘 수 있다.
- 사용 예시 : Toast, Modal, Tooltip, Pop-up 등
```jsx
function App() {
  return (
    <div onClick={() => console.log("div 클릭됨")}>
      부모 div
      <PortalComponent />
    </div>
  );
}

function PortalComponent() {
  return ReactDOM.createPortal(
    <button onClick={() => console.log("버튼 클릭됨")}>Portal 버튼</button>,
    document.body
  );
}
```


