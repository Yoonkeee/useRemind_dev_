# useState

## fiber, hook, queue, update

```ts
fiber = [ hook1, hoo2, hook3, ... ]  // hook들은 linked list로 연결되어 있음.
hook = [ queue ]
queue = [ update1, update2, update3, ... ]
```

mountState에서 mountWorkInProgressHook()로 전달된 값을 hook으로 할당한다.  

```ts
function mountWorkInProgressHook() : Hook {
  const hook: Hook = {
    memoizedState: null,  // 마지막으로 알고 있는 저장된 state 값

    baseState: null,
    queue: null,  // hook 객체가 갖고 있는 객체
    baseUpdate: null,

    next: null,  // 다음 hook에 할당될 key 값을 갖는 linked list
  }

  if (workInProgressHook === null) {
    // 만약 workInProgressHook이 null이라면, 첫 hook이다.
    firstWorkInProgressHook = workInProgressHook = hook;
  } else {
    // 아니라면 다음 노드에 연결한다.
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```

## mountState()
```ts
function mountState() {
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function) {
    // useState를 생성할 때 할당한 초기값이 함수라면 여기서 실행한다.
    // ex) const [state, setState] = useState(() => 1+1)
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;

  const queue = (hook.queue = {
    last: null,  // update 객체를 모두 실행하고 queue 안에 담겨있는 마지막 값을 할당한다.
    dispatch: null,  // queue에 update 객체를 push하는 함수
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  })

  const dispatch = (queue.dispatch = (
    dispatchAction.bind(
      null,
      ((currentlyRenderingFiber), 
      queue)
  )));
  // 이 구조가 useState의 return인 [state, setState]이다.
  return [hook.memoizedState, dispatch];
}
```

# setState()가 값을 업데이트하는 과정
1. update 객체 생성
2. update를 queue에 저장 (circular linked list)
3. 불필요한 렌더링을 방지하는 최적화
4. update를 적용하기 위해 work를 scheduler에 예약 (scheduling)
  work? : reconciler가 컴포넌트의 변경을 DOM에 적용하기 위해 수행하는 일

## dispatchAction()
```ts
function dispatchAction(fiber, queue, action) {
  const alternate = fiber.alternate;
  if(
    // currentlyRenderingFiber = workInProgressFiber이다.
    // 즉, 현재 fiber가 workInProgressFiber라면 render phase인 것이다.
    fiber === currentlyRenderingFiber ||
    // alternate = 다른 트리의 참조값
    (alternate !== null && alternate === currentlyRenderingFiber)
  ) {
    // 렌더 페이즈의 업데이트이다.
    didScheduleRenderPhaseUpdate = true;
    const update = {
      expirationTime,  // work가 진행될 때 work가 여기에 값을 할당한다.
      suspenseConfig,
      action,  // setState를 호출할 때 들어온 parameter
      eagerReducer: null,  // 불필요한 렌더링 최적화에 필요
      eagerState: null,  // 불필요한 렌더링 최적화에 필요
      next: null,  // 다음 노드의 주소값
    }
    if (renderPhaseUpdates === null) {
      // 렌더 페이즈에서 발생한 업데이트들을 임시로 저장하는 공간
      renderPhaseUpdates = new Map();
    }
    const firstRenderPhaseUpdate = renderPhaseUpdates.get(queue);
    if (firstRnderPhaseUpdate === undefined) {
      // 아직 render phase update가 저장된게 없다면
      renderPhaseUpdates.set(queue, update);
    } else {
      let lastRenderPhaseUpdate = firstRenderPhaseUpdate;
      while (lastRenderPhaseUpdate.next !==) {
        lastRenderPhaseUpdate = lastRenderPhaseUpdate.next;
      }
      // 마지막 노드에 update를 저장
      lastRenderPhaseUpdate.next = update;
    }

  } else {
    // 아무런 업데이트가 없는 idle 상태이다.
    const update = {
      expirationTime,  // work가 진행될 때 work가 여기에 값을 할당한다.
      suspenseConfig,
      action,  // setState를 호출할 때 들어온 parameter
      eagerReducer: null,  // 불필요한 렌더링 최적화에 필요
      eagerState: null,  // 불필요한 렌더링 최적화에 필요
      next: null,  // 다음 노드의 주소값
    }
    // update: 새로운 업데이트. update 객체는 circular linked list이다.
    const last = queue.last;  // queue의 마지막 값을 갖고온다.
    if (last === null) {  // null이라면 첫 값이다.
      // 최초 업데이트이므로 next에 연결하고, queue.last에도 update를 할당하여,
      // 다음 업데이트시 이곳으로 오지 않는다.
      update.next = update;
    } else {
      const first = last.next;  // queue.last에 연결된 next 즉, 이 queue의 head
      if (first !== null) {
        update.next = first;  // update의 next를 head로 연결하여 circular 구조 유지
      }
      last.next = update;  // last의 next에 새로운 update를 할당
    }
    queue.last = update;  // queue의 last에 새로운 update를 할당
  }

  if (  // fiber가 update를 수행하고 있지 않은지? 즉, 이 업데이트가 work를 scheduling하는지?
    fiber.expirationTime === NoWork &&
    (alternate === null || alternate.expirationTime === NoWork)
  ) {  
    // action의 결과값이 현재 상태 값과 같다면 함수 실행 중지
    // mountState()에서 lastRenderedReducer에 basicStateReducer 할당했음.
    const lastRenderedReducer = queue.lastRenderedReducer;
    if (lastRenderedReducer !== null) {
      let prevDispatcher;
      const currentState = queue.lastRenderedState;  // 마지막 상태값을 갖고온다.
      // action: setState에서 받은 인자값
      const eagerState = lastRenderedReducer(currentState, action);
      update.eagerReducer = lastRenderedReducer;  // 바뀌어야 할 state를 계산하는 것
      update.eagerState = eagerState;  // 바뀌어야 할 state 값
      if (is(eagerState, currentState)) {  // 업데이트될 값이 같다면?
        return;  // dispatch 종료
      }
    }
  }
  scheduleWork(fiber, expirationTime);  // 스케쥴러에 work를 업데이트
}
```

## render phase 도중 update가 되는 경우
```jsx
function Foo() {
  const [count, setCount] = useState(0);
  if (count === 1) setCount(prev => prev + 1);
  
  return <button onClick={() => setCount(prev => prev + 1)}> {count} +1 </button>
}
```
- 위 함수를 렌더하면 count가 0인 상태에서 버튼이 만들어진다.
- 버튼을 클릭하면 state가 변하므로 다시 렌더링이 시작된다.
- JSX Element가 return되기 전, if 조건문을 만족하여 setCount의 +1이 한번 더 실행된다.
- 이 지점이 render phase에서 update가 되는 경우이다.
```ts
if(
  // currentlyRenderingFiber는 workInProgressFiber와 동일. 이름만 다르다.
  fiber === currentlyRenderingFiber ||
  // alternate = 다른 트리의 참조값
  // alternate가 있으며, 그 alternate가 workInProgressFiber일 경우
  // render phase에서의 update인 것.
  (alternate !== null && alternate === currentlyRenderingFiber)
  )
```


eof
