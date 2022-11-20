# Trello clone

## 0.1 Recoil 을 이용해서 단위 환산 앱 만들기

1. 입력 레이아웃을 만든다.

```javascript
// App.tsx;

function App() {
  return (
    <div>
      <input type='number' placeholder='Minutes' />
      <input type='number' placeholder='Hours' />
    </div>
  );
}
export default App;
```

2. minutes 와 hour 에 대한 atom 을 만들어 내보내준다.

```javascript
// atoms.tsx
import { atom } from "recoil";

export const minuteState = atom({
  key: "minutes",
  default: 0,
});
```

3. useRecoilState 를 이용해서 atom 을 받아온다.
4. input 의 value 와 value 를 세팅하는 함수를 만들어서 연결해준다.

```javascript
//App.tsx
import React from "react";
import { useRecoilState } from "recoil";
import { minuteState } from "./atoms";

const [minutes, setMinutes] = useRecoilState(minuteState)
const onMinutesChange = (event) => {
  setMinutes(+event.currentTarget.value); // + 를 붙여서 string 을 number 로 바꿔줌.
}
return (
  <div>
    <input value={minutes} onChange={} type="number" placeholder="minutes">
    <input type="number" placeholder="hours">
  </div>
)
```

5. **selector** 를 이용하면 get함수로 minute atom 의 state를 가져와서 수정하고 output 을 내보낼 수 있다.

```javascript
// atoms.tsx
import { selector } from "recoil";

export const hourSelector = selector({
  key: "hours",
  get: ({ get }) => {
    const minutes = get(minuteState); // minuteState atom 을 가져오는 get
    return minutes / 60;
  },
});
```

6. 다시 App.tsx 로 돌아와서 useRecoilValue로 위의 output 을 받아보자

```javascript
// App.tsx
import React from "react";
import { useRecoilState, useRecoilValue } from "recoil";
import { minuteState } from "./atoms";

const [minutes, setMinutes] = useRecoilState(minuteState)
const hours = useRecoilValue(hourSelector) // value 만 가져오면 되니까 useRecoilValue
const onMinutesChange = (event) => {
  setMinutes(+event.currentTarget.value);
}
return (
  <div>
    <input value={minutes} onChange={} type="number" placeholder="minutes">
    <input value={hours} type="number" placeholder="hours"> // hours값을 입력해준다.
  </div>
)
```

- 그런데 여기까지 하면 hour 인풋에는 입력을 할 수 없다 onChange 이벤트가 없기 때문이다.
- hour 입력해서 minutes 를 돌려받고 싶을 때
  1. selector 가 state를 가져오고 수정된 값을 돌려주는 방법
  2. set 프로퍼티를 이용해서 state 수정하기 (두개의 atom 을 만들 필요가 없다.)

## selector 의 'set' property

- (get 은 데이터를 가져올 수 있었다.)
- set 은 원하는 state 데이터를 수정할 수 있게 해준다.
- set함수는 첫번째 argument 로 option 객체를 받는다. 그건 setRecoilState 를 가지고 있다.
- set함수의 두번쨰 argument 는 우리가 보내는 새로운 값을 넣는다.

```javascript
// atoms.tsx
export const hourSelector =
  selector <
  number >
  {
    key: "hours",
    get: ({ get }) => {
      const minutes = get(minuteState);
      return minutes / 60;
    },
    set: ({ set }, newValue) => {
      const minutes = Number(newValue) * 60;
      // set(아톰, 바꿀값)
      set(minuteState, minutes);
    },
  };
```

- selector 사용하기 : 셀렉터를 useRecoilState 에 넣으면 get 옵션으로 받아오는 value 와 set 옵션으로 정의한 세팅함수를 반환받는다.

```javascript
// App.tsx
function App() {
  const [minutes, setMinutes] = useRecoilState(minuteState);
  // hours는 get : ({get})=>  return minutes/60;
  // setHours 는 set : ({set},newValue) => set(minuteState, newValue*60);
  const [hours, setHours] = useRecoilState(hourSelector); // selector를 useRecoilState
  const onMinutesChange = (event: React.FormEvent<HTMLInputElement>) => {
    setMinutes(+event.currentTarget.value);
  };
  const onHoursChange = (event: React.FormEvent<HTMLInputElement>) => {
    setHours(+event.currentTarget.value);
  };
  return (
    <div>
      <input
        value={minutes}
        onChange={onMinutesChange}
        type='number'
        placeholder='Minutes'
      />
      <input
        onChange={onHoursChange}
        value={hours}
        type='number'
        placeholder='Hours'
      />
    </div>
  );
}
```

## react-beautiful-dnd : 드래그&드랍 만들기

- 설치
- `npm i react-beautiful-dnd`
- `npm i --save-dev @types/react-beautiful-dnd`

### 시작

```javascript
import { DragDropContext } from "react-beautiful-dnd";
const onDragEnd = () => {};
function App() {
  return <DragDropContext onDragEnd={onDragEnd}></DragDropContext>;
}
```

- DragDropContext 를 드래그앤드롭을 할 영역에 생성시킨다.
- onDragEnd 라는 prop 에 함수를 전달해야 한다. 이 함수는 드래그를 끝낸 시점에서 불려지는 함수이다.
- DragDropContext 는 children 요소를 필요로 한다.
- Drappable 은 드래그드롭을 하는 영역(e.g. 리스트)이다. Draggable 은 드래그를 할 영역이다
- Drappable과 Draggable 의 children 은 **함수**여야 한다.
- Drappable 안에 컴포넌트를 넣으면 바로 사용할 수 있는 뭔가를 얻는다
- Draggable 의 prop 에는 draggableId 와 index 를 전달해야 한다.
- DragDropContext > Droppable > Draggable

![DragDrapContext의구조](https://user-images.githubusercontent.com/2182637/53607406-c8f3a780-3c12-11e9-979c-7f3b5bd1bfbd.gif)

```javascript
<DragDropContext onDragEnd={onDragEnd}>
  <div>
    <Droppable droppableId='one'>
      // <ul></ul> 는 안됨
      {() => (
        <ul>
          <Draggable draggableId='first' index={0}>
            {() => <li>one</li>}
          </Draggable>
          <Draggable draggableId='second' index={1}>
            {() => <li>two</li>}
          </Draggable>
        </ul>
      )}
    </Droppable>
  </div>
</DragDropContext>
```

## 7.3 Drag and Drop part Two

#### DroppableProvided

- Droppable 의 children 함수에 전달되는 인자는 provided라는 객체이다.

```javascript
interface DroppableProvided {
  innerRef: (element: HTMLElement | null) => any;
  placeholder?: React.ReactElement<HTMLElement> | null | undefined;
  droppableProps: DroppableProvidedProps;
}
```

- innerRef 는 ul 의 ref 라는 prop 에 전달해줄 데이터다
- 이 provided 객체에는 droppableProps 가 있는데 ul 이 필요로 하는 props 가 들어있다.

```javascript
<Droppable droppableId='one'>
  {(magic) => (
    <ul ref={magic.innerRef} {...magic.droppableProps}>
      <Draggable draggableId='first' index={0}>
        {(magic) => (
          <li ref={magic.innerRef} {...magic.draggableProps}>
            <span {...magic.dragHandleProps}>🔥</span>
            One
          </li>
        )}
      </Draggable>
    </ul>
  )}
</Droppable>

// 웹사이트에서 ul 의 모습. magic 이 one 과 1을 전달해줌
<ul data-rbd-droppable-id="one" data-rbd-droppable-context-id="1">
```

- 이 provided 객체 안에는 provided.placeholder 가 있는데 드래그로 자리에서 빼내도 그 빈자리에 채워진 것처럼 만들어준다.

```javascript
<Droppable droppableId='one'>
  {(magic) => (
    <Board ref={magic.innerRef} {...magic.droppableProps}>
      {toDos.map((toDo, index) => (
        <Draggable draggableId={toDo} index={index}>
          {(magic) => (
            <Card
              ref={magic.innerRef}
              {...magic.dragHandleProps}
              {...magic.draggableProps}>
              {toDo}
            </Card>
          )}
        </Draggable>
      ))}
      {magic.placeholder} // 드래그 해도 사이즈가 축소되지 않음.
    </Board>
  )}
</Droppable>
```

#### DraggableProvided

```javascript
export interface DraggableProvided {
  innerRef: (element?: HTMLElement | null) => any;
  draggableProps: DraggableProvidedDraggableProps;
  dragHandleProps?: DraggableProvidedDragHandleProps | undefined;
}
```

- 드래그 되길 바라는 요소의 prop에는 `draggableProps`를 추가 한다.(요소가 드래그에 딸려서 움직인다)
- `dragHandleProps` 는 잡을 부위(손잡이?) 요소에 prop으로 추가해주면 된다.

- li를 어떤 위치에서도 드래그해서 옮기도록 하고 싶으면 li 에 `draggableProps` 와 `dragHandleProps` 를 함께 넣어준다.

```javascript
<Draggable draggableId='first' index={0}>
  {(magic) => (
    <li ref={magic.innerRef}
        {...magic.draggableProps}
        {...magic.dragHandleProps}>
      One
    </li>
  )}
</Draggable>

<Draggable draggableId='first' index={0}>
  {(magic) => (
    <li ref={magic.innerRef} {...magic.draggableProps}>
      <span {...magic.dragHandleProps}>🔥</span>  //불꽃이모지만 잡을수 있다 One 은 잡을수없음
      One
    </li>
  )}
</Draggable>
```

## 7.5 Reordering

- `onDragEnd` 는 어떤 일이 일어났는지에 대한 정보로 많은 `argument`를 준다.
  - `draggableId` : 방금 드래그 했던 것
  - `destination` : 드래그를 놓은 곳(Drappable). 없을 수도 있다 엉뚱한 데에 놓으면
  - `source` : 드래그가 출발한 곳(Drappable)
- **주의! Draggable 의 `key` 는 `draggableId` 는 같아야 한다!**

```javascript
const onDrageEnd = (args) => {
  console.log(args);
  /*
    {
    draggableId: 'b' //현재 드래그 했던 것
    type: 'DEFAULT'
    source: {...}
    combine:null
    destination: {droppableId: 'one' , index: 0} // 드래그를 놓은 곳 drappable
    mode: "FLUID"
    reason: "DROP"
    source: {index: 1, dorppableId: 'one'} // 드래그가 출발한 drappable
    }
  */
};
```

- 재정렬 하기

```javascript
const onDrageEnd = (args) => {
  console.log(args);
  /*
    {
    draggableId: 'b' //현재 드래그 했던 것
    type: 'DEFAULT'
    source: {...}
    combine:null
    destination: {droppableId: 'one' , index: 0} // 드래그를 놓은 곳 drappable
    mode: "FLUID"
    reason: "DROP"
    source: {index: 1, dorppableId: 'one'} // 드래그가 출발한 drappable
    }
  */
};
```

#### 문제 발생

- 재배치 할 때마다 모든 cards를 다시 랜더링 하면서 글씨가 깜빡이는 현상이 나타난다.

#### 문제 해결 : Optimization

원인

- 리액트에서 컴포넌트의 state가 변하면 해당 컴포넌트의 모든 children 을 다시 랜더링이 된다.
- 그런데 하나만 움직여도 모든 리스트(children)가 반복적으로 랜더링이 되면서 리소스를 잡아먹는다
- parent 컴포넌트가 커질수록 수많은 children 이 랜더링이 되면 이와 같은 일이 생길 가능성이 크다.

해결방법: **react memo** 를 사용한다

- react memo 는 prop 이 변하지 않는다면 컴포넌트를 랜더링 하지 않게 할 수 있다. -`export default DraggabbleCard` 에서 `export default React.memo(DragabbleCard);` 로 변경하면 DraggableCard의 prop 이 변경되지 않는 한 다시 랜더링 하지 않게 된다.
