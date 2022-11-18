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

## selector 의 'set' property

- get 은 데이터를 가져올 수 있었다.
- set 은 원하는 state 데이터를 수정할 수 있게 해준다.
