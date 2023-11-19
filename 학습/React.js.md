# React.js

## React Elements
기존의 객체지향적인 UI Model은 render시 각 component의 instance들을 appendChild, removeChild, destroy등을 직접 조작해야 하고, 부모 컴포넌트가 자식 컴포넌트의 instance를 직접 생성하고 호출하기 때문에 결합력이 강해진다.

### What is Elements?
- Component instance를 묘사하기 위한 순수 객체
- Instance가 아니며, React가 브라우저에 렌더링하기 위해 필요한 정보를 갖고 있는 JS Object
- Element는 type, props를 갖는다.
- type: string | ReactClass
- props : object
- Element가 HTML Tag일 경우 type은 string이며, props는 tag의 attributes이다.
  ```typescript
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
  ```typescript
  {
    type: Button,
    props: {
      color: 'blue',
      children: 'OK!'  // 다른 element를 children으로 갖음으로써 tree 구조를 가질 수 있다.
    }
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
React는 v16 이전까지는 Stack 구조를 갖고 렌더링을 하였는데, stack 구조는 rendering이나 작업의 순서를 바꿀 수 없었다. 이를 위해 Fiber를 도입하였다.

## Hooks
### Hooks란?
