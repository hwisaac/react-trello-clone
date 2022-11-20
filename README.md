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

- Droppable 의 children 함수에 전달되는 인자는 provided라는 객체이다.

```javascript
interface DroppableProvided {
  innerRef: (element: HTMLElement | null) => any;
  placeholder?: React.ReactElement<HTMLElement> | null | undefined;
  droppableProps: DroppableProvidedProps;
}
```

- 이 객체에는 droppableProps 가 있는데 ul 이 필요로 하는 props 가 들어있다.

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
```
