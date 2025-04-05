---
title: Vue [오류해결] v-for로 hover 각각의 tooltip 띄우기
author: minjo
date: 2025-03-05 10:00:00 +08
categories: [Vue, Layout]
tags: [jetpack compose - layout]
math: true
toc: true
pin: true
mermaid: true
---

# v-for로 hover 각각의 tooltip 띄우기

{: .mt-6 .mb-0 }

{: .mt-4 .mb-0 }

## 개발 라이브러리 : Vue + Jquery

{: .mt-2 .mb-0 }

#### 회의실을 예약하는 상황에서 예약된 회의실의 경우 tooltip을 활용하여 예약현황을 보여주고자 하였다.

vue 문법중에서 v-for @mouseover, @mouseleave를 사용하면 간단하게 해결할 수 있는데 한 가지 오류가 발생했다.

{: .mt-4 .mb-0 }

## 기존코드

```html
<div
  v-for="(rooms, i) in meeting.rooms"
  class="usingRoom"
  @mouseover="toolTip[room.id] = true"
  @mouseleave="toolTip[room.id] = false"
>
  <div v-for="(room, i) in rooms">
    <div
      v-for="(schedule, i) in getTargetSchedule(room)"
      class="meetingRoom_tooltip"
      v-show="schedule.id == room.id"
    ></div>
  </div>
</div>
```

간단하게 미팅룸을 용도에 맞게 그룹화하여 데이터를 가져왔고, 그룹화된 미팅룸을 반복문을 이용하여 일정이 존재한다면 **hover**가 된 상황에서
**tooltip**을 보여지게 하고자하였다.

<font color="#ff0000">하지만</font> v-show의 마우스를 갖다대도 상태에서 상태가 변화되지않았다.
이유를 찾아보니 공식문서에 Vue는 렌더링이전 상황의 데이터에 대해서만 상태변화를 감지한다는 것을 알게되었다. [Vue 공식문서](https://v2.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)

오류해결 : Jquery를 이용하여 currentTarget을 찾아 내부에 div태그에 display를 설정할수있도록 함수를 변경하여 해결하였다.
다른 화면에서는 초기데이터가 주어진 상황에서 v-show를 할 일밖에 없다보니 Vue와 렌더링에 관한 개념을 놓치고 있었는데 알게되어 다행이다
아 근데 다 수정하고나니 v-tooltip이란거를 발견했다.
또 공부해봐야겠다.

## 수정코드

```html
<div v-for="(rooms, i) in meeting.rooms" class="usingRoom"
     @mouseover="toggleTooltip($event, true)"
     @mouseleave="toggleTooltip($event)>
	<div v-for="(room, i) in rooms">
      <div v-for="(schedule, i) in getTargetSchedule(room)" class="meetingRoom_tooltip">
      	</div>
    </div>
</div
```

```vue
methods: { toggleTooltip(e, onHover=false) { let visible = onHover ? 'block' :
'none' $(e.currentTarget()).find('div').css('display', visible) } }
```

> 공부하다 보니 v-tooltip이라는 것도 존재한다는 것을 알았다. 처음부터 이걸 썼으면 좋았을 텐데라고 생각하며 v-tooltip도 공부해봐야겠다.
