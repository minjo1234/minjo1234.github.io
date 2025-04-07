---
title: Java 프로젝트에 Kotlin 도입하기
author: minjo
date: 2025-03-06 10:00:00 +08
categories: [Java, Kotlin, Spring]
tags: [Kotlin, Java]
math: true
toc: true
pin: true
mermaid: true
---

## 🧭 **사내 Java → Kotlin 전환 제안 요약**

### 도입배경

- 기존 사내 웹 서비스는 **Java 기반**으로만 구성됨
- 유지보수 효율과 생산성 향상을 위해 Kotlin 병행 도입을 검토

### 전제조건

- 자바 개발자에게 친숙한 언어여야함 (기존 Java 개발자를 고려)
- 변환과정이 복잡하지 않아야함(convert to Java To Kotlin 이용

### Kotlin 장점

#### 1.Null Safety, 컴파일 타입에 null 가능성을 체크할 수 있어, NullPointException을 방지하는데 강력

#### 2.컬렉션 처리의 간결함 , 함수형 프로그래밍 스타일 예시 (map, filter, reduce, groupby 제공)

```kotlin

val sales = listOf(100, 200, 150, 300)
val average = sales.filter { it >= 200 }.average()

```

```java

List<Integer> sales = Arrays.asList(100, 200, 150, 300);
double avg = sales.stream()
                  .filter(i -> i >= 200)
                  .mapToInt(i -> i)
                  .average()
                  .orElse(0);


```

### 3. **data class 제공**

- `@Data`, `@Builder`, `@Getter`, `@Setter` 없이도 자동 생성
- `copy()`, `equals()`, `hashCode()` 등 기본 제공
- ex) SalesTypeStatusDTO.kt

  4.간결한 테스트 코드 5.확장 함수와 DSL 지원

### 6. Kotlin <-> JPA

개인적으로 둘 간에 가장 어울리지 않는 부분은 불변과 가변에 대한 개념이라고 생각합니다.
Kotlin은 open class를 이용하지않는 이상 기본적으로 final 그렇다는 말은 = JPA 프록시 객체를 이용하지 못함, data class를 적극 사용하고 가변인 `var`보다는 불변인 `val`을 사용하는 것을 권장합니다.

JPA를 다룰때는 두 가지 선택지가 존재합니다.

## 1.Entity -> Java 나머지 Kotlin 변경

1.JPA를 Java클래스로 나머지를 kt를 사용한다.

- Kotlin ↔ Java 혼용은 **코드 통일성 저하**
- Kotlin 쪽에서 Java Entity를 쓸 때 `nullability` 관련 주의 필요 (`@NotNull` 없어도 nullable로 추정됨)
- Entity가 많아질수록 **관리 복잡도 증가**

## 2.JPA open class로 설정하면 프록시 객체를 이용할 수 있다. -> 코드 통일성이 올라감

### 단점

- 플러그인 의존도가 있음 (`all-open`, `no-arg`)
- 런타임 프록시 동작에 대한 **이해가 필요**
- 실수로 `open` 안 붙이면 런타임 에러 가능 (플러그인 없으면)

---

### 수정순서

현실적으로 영향도가 적은 DTO,Controller를 수정하고 복잡하지않은 Service단을 먼저 수정하며 Kotlin문법을 익혀나가는게 좋을 것 같습니다. 추후 Entity에 대하여 Kotlin 파일로 변경할지 Java파일을 그대로 이용할지 논의해나가면 좋을 것 같습니다.

---

## 🛠 Kotlin 적용 우선순위

| 단계 | 대상              | 설명                                        |
| ---- | ----------------- | ------------------------------------------- |
| ✅ 1 | DTO               | `data class` 활용, 가장 안전하고 효과 큼    |
| ✅ 2 | Controller        | API 호출 로직, Null-safe 파라미터 처리 용이 |
| ✅ 3 | Service / UseCase | 비즈니스 로직 Kotlin화로 가독성 향상        |
| ✅ 4 | Test              | MockK + Kotest로 테스트 코드 간결화         |
| ✅ 5 | Util / Mapper     | 확장 함수 등 활용 가능                      |
|      |                   |                                             |

---

### 참조문서

JPA <-> Kotlin : [INFLAB-Tech](https://tech.inflab.com/20240110-java-and-kotlin/)<br>
Jpa <-> Kotlin [Kotlin Jpa 설계](https://catsbi.oopy.io/ecfb2d3a-4095-41a9-ae21-0d36a93f552c)<br>
Java To Kotlin - Kakao Tech - [KakaoTech](https://tech.kakaopay.com/post/kotlin-migration/)

---
