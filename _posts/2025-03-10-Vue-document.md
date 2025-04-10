---
title: Vue와 jQuery를 함께 사용할 때 이벤트 위임은 누구에게?
author: minjo
date: 2025-03-10 10:00:00 +08
categories: [Vue, Javascript]
tags: [Vue, Javascript]
math: true
toc: true
pin: true
mermaid: true
---

## 🤷‍♂️ Vue와 jQuery를 함께 사용할 때 이벤트 위임은 누구에게?

실무에서 Vue와 외부 라이브러리(jQuery 등)를 혼용해서 사용하는 경우가 종종 발생한다.  
특히 레거시 프로젝트를 유지보수하거나, Vue에서 감지하지 못하는 데이터 변화를 처리할 때 jQuery를 사용하는 경우가 많다.

이 과정에서 이벤트 위임을 Vue 쪽으로 처리해야 하는지, jQuery 쪽으로 처리해야 하는지 헷갈리는 경우가 생긴다.  
현재 진행 중인 프로젝트에서도 Vue와 jQuery를 같이 사용 중인데, Vue의 이벤트가 작동하지 않는 경우가 있어 원인을 분석하게 되었다.

---

### 코드로 알아보기.

✅ 예시:회의 참석자를 선택하는 select2 컴포넌트
간단한 예시로, 회의실 참석자를 설정하기 위한 태그를 구현했다.
select2를 이용해 렌더링하고, 참석자 정보를 바인딩하려는 상황이다.

```html
<div id="meetingRoom_area">
  <dl class="form">
    <dt><span class="asterisk bold-text">참석자</span></dt>
    <dd>
      <div class="el-select1" style="width: 100%">
        <select
          id="meeting_target_select"
          class="form-select"
          multiple
          :disabled="meeting.dto.id != '' && !(meeting.updatable)"
        ></select>
      </div>
    </dd>
  </dl>
</div>
```

API를 통해 데이터를 비동기로 바인딩한 후, select2 라이브러리를 적용시키는 구조다.
이처럼 DOM이 완성된 이후에 select2가 렌더링되기 때문에 Vue의 @click이나 @change 이벤트가 작동하지 않는 문제가 발생한다.

---

### Vue와 DOM 탐지 구조의 차이점

Vue는 **초기 `data()`에 정의된 반응형 데이터**만 감지할 수 있다.  
즉, 컴포넌트가 렌더링되는 시점 이전에 존재하던 데이터에 대해서만 Vue는 DOM을 자동으로 업데이트할 수 있다.  
이걸 **"반응형(Reactivity) 시스템"**이라고 부른다.

```javascript
	data() {
  return {
    attendanceList: [], // 여기서 정의된 값만 Vue가 반응형으로 추적함
  }
}
```

하지만 API 호출 이후 동적으로 DOM이 생성되거나 외부 라이브러리를 통해 조작되는 경우, Vue는 이를 감지하지 못한다.
이럴 땐 jQuery를 통해 DOM을 직접 다루는 방식이 필요하다.

---

### 해결방안 : jQuery의 `$(document).on()` 사용 예시

```javascript
$(document).on("change", "#meeting_target_select", function () {
  let selectedData = $(this).select2("data")[0];
  let alreadyExists = popupVue.meeting.dto.attendanceList.some(
    (item) => item.typeId == selectedData.id
  );

  if (!alreadyExists) {
    popupVue.meeting.dto.attendanceList.push({
      typeId: selectedData.id,
      organizationType: selectedData.type
    });
  }
});
```

이벤트를 탐지할 수 없는 이유는,  
Vue에서 렌더링할 때 해당 DOM 요소(`<select id="meeting_target_select">`)가 존재하지 않기 때문이다.  
DOM 생성 이후 select2로 렌더링되므로 Vue는 이를 감지하지 못한다.

---

### 🤔 그럼 그러면 $(document).on으로 이벤트 다 다루면 되는거 아닌가요 ?

처음에 나는 이렇게 생각했다. 하지만 $(document).on은 \***\*이벤트 위임이 최상위 객체(document)**에 등록되므로,  
모든 이벤트가 DOM 트리를 타고 `document`까지 버블링된 후 처리된다. 이는 성능에 좋지 않다.
가능하다면 Vue의 `@click`, `@change` 등 자체 이벤트 바인딩 방식을 사용하는 것이 좋다.  
Vue에서 감지할 수 있는 범위 내라면 jQuery 없이도 충분히 처리할 수 있다.

---

### ✅ 정리

- Vue는 컴포넌트가 렌더링될 때 기준으로 반응형 데이터를 추적한다.
- 동적으로 생성된 DOM 요소나 외부 라이브러리로 만들어진 UI는 Vue가 감지하지 못한다.
- 이 경우 `$(document).on()`을 사용하면 이벤트를 위임할 수 있다.
- 그러나 성능상 이유로 **Vue에서 처리 가능한 영역은 Vue로, 불가피한 경우에만 jQuery를 사용**하는 것이 좋다.
