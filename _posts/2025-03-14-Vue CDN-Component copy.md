---
title: 1️⃣ Vue Component 부모,자식 컴포넌트 이해하기.
author: minjo
date: 2025-03-14 10:00:00 +08
categories: [Vue, Javascript]
tags: [Vue]
math: true
toc: true
pin: true
mermaid: true
---

{: .mt-2 .mb-0 }

# 1️⃣ Vue Component 부모,자식 컴포넌트 이해하기.

# 035 CDN To Component (CDN 방식에서 컴포넌트 방식으로)

저번시간에 SSR,CSR의 장단점을 가지고 페이지를 수정하는 예시를 간단하게 들려고 합니다.

하지만 현재 프로젝트에서는 Vue server를 따로 분리하지 않았기에 운영중인 서비스를 위해서라도
component 형식이 필요한 페이지들은 현재 방식으로 개발한 이후, Vue server를 도입하면 유지보수가 빠르다는 생각이 들어 CDN 방식의 component를 이용해보려고 합니다.

탭(Tab)을 사용하는 페이지처럼 화면 전환 없이 동작해야 하는 UI라면, **SPA(Single Page Application)** 방식이 더 적합합니다.  
그래서 기존에는 CDN 방식으로 Vue를 사용했지만, 이제는 **Vue Component 방식**으로 전환하고자 합니다.

> ✅ Vue Version : 2.6.11  
> ✅ CDN 방식에서 component를 이용하려고 하며, `.vue` 파일이 아닌 `.js` 파일로 구성

가볍게 **검색 조건 바** 하나를 Vue 컴포넌트로 만들어 호출해보겠습니다.

---

## 1. 컴포넌트 파일 생성 및 작성

```javascript
Vue.component("search-condition", {
  template: `
    <!-- html 내용 정리 -->
    <div>
      <!-- 예: 년도 선택 영역 -->
    </div>
  `,
  props: {
    condition: Object // 상위에서 받은 검색조건 데이터
  },
  data() {
    return {
      // 컴포넌트 내부 상태 정의
    };
  },
  watch: {
    // 필요 시, prop이나 data 변화 감지
  },
  methods: {
    // 예: 사용자 입력 → 조건 업데이트 시 부모로 emit
    updateYear(currentYear) {
      this.$emit("update-condition", { key: "year", value: currentYear });
    }
  },
  filters: {
    // 추후 computed나 methods로 대체 가능
  }
});
```

---

## 2. 컴포넌트 네이밍 규칙 (Kebab Case)

Vue에서는 컴포넌트를 템플릿 내에서 사용할 때, **케밥 케이스(kebab-case)**를 **권장**합니다.

### ✅ 케밥식(kebab-case)이란?

- 모두 **소문자**로 작성
- **단어 사이에 하이픈(-)** 사용
- 마치 꼬치 케밥처럼 단어가 연결되어 있음

| 예시   | 설명             |
| ------ | ---------------- |
| 원문   | USER LOGIN LOG   |
| 케밥식 | `user-login-log` |

> 📌 케밥 케이스는 Vue 외에도 yml 설정 파일, URL 등에서도 자주 사용됩니다.

---

## 3. 스크립트 호출 (CDN 방식)

컴포넌트 JS 파일을 사용하기 위해 `<script>`로 외부 JS를 호출합니다.

`<script th:inline="javascript" th:src="@{/js/vue/search/searchCondition.js}"></script>`

---

## 4. 템플릿에서 컴포넌트 사용

`<search-condition :condition="condition" @update-condition="updateCondition"></search-condition>`

- `:condition="condition"`
  → 부모의 `condition` 데이터를 **props**로 자식 컴포넌트에 전달합니다 (읽기 전용).
- `@update-condition="updateCondition"`
  → 자식 컴포넌트에서 `$emit('update-condition')` 이벤트를 발생시키면, 부모는 이 메서드를 실행합니다.

---

## 5. 자식에서 부모로 데이터 전달 ($emit)

Vue는 **단방향 데이터 흐름**을 따르기 때문에,
자식 컴포넌트가 직접 부모의 데이터를 수정하는 것은 **권장되지 않습니다**.

대신, `$emit`을 통해 이벤트를 발생시켜 부모에게 **“값을 바꿔달라”고 요청**합니다.

### 📍 자식 컴포넌트

`this.$emit('update-condition', { key: 'year', value: currentYear });`

### 📍 부모 컴포넌트 (예: exampleVue)

`methods: {     updateCondition({ key, value }) {       this.condition[key] = value;     }   }`

---

## 🙋‍♂️ 마무리

지금은 Vue를 .vue 파일 없이 사용하는 CDN 방식이지만,
SPA에 가까운 구조로 바꾸고자 한다면 이런 식의 **컴포넌트 모듈화**부터 점진적으로 적용해보는 것이 좋습니다.
단방향 흐름, props와 $emit 사용 패턴을 익혀두면 향후 Vue CLI 기반 프로젝트로도 자연스럽게 넘어갈 수 있습니다.
