---
title: Spring @Transactional 어노테이션 완벽 가이드
author: minjo
date: 2025-04-26 10:00:00 +08
categories: [DB, Transaction, Spring]
tags: [DB, Transaction, Spring]
math: true
toc: true
pin: true
mermaid: true
---

# Transaction Annotation과 일관성(Consistency)

---

Transaction의 특성중 C(Consistency)를 공부하는 도중 Transactional 어노테이션을 이용하면  
단순히 DB에서뿐만아니라 비즈니스로직과의 연결을 이용하여  
**하나의 메소드를 트랜잭션의 단위**로 본다는 것을 알게되었다.

---

## 1.@Transactional

```java
# 개발자 입장에서 하나의 메소드를 @Transactional 어노테이션을 이용하여 하나의 트랜잭션으로 보는 것이다.

@Transactional
public void saveService(MyServiceDTO dto) {
		if(dto.size() < 0) {
			throw new IllegalArgumentException("아무것도 존재하지 않음");
		}

		save(dto);
	}
```

더욱 상세하면 설명하자면

- `saveService`라는 메소드는 Spring의 어노테이션을 이용하여 트랜잭션 단위가 되었다.
- 코드 상에서 에러(Exception)이 발생하거나
- DB작업(쿼리)가 실패하면 트랜잭션은 `ROLLBACK` 된다.
- 모든 코드와 DB 쿼리가 정상적으로 작동하여야 트랜잭션이 커밋(Commit) 된다.

---

### 🚨 조심할 점

- **런타임 예외(RunTimeException)가** 발생하면 트랜잭션은 자동으로 `ROLLBACK`
- **체크 예외(Exception)은** 자동으로 `ROLLBACK`되지 않음
  → 필요하면 `@Transactional(rollbackFor = Exception.class)` 이렇게 설정
- **DB 레벨 에러**(ex. 제약조건 위반, Deadlock 등)가 발생하면 → 역시 롤백된다.

---

## 2.@Transactional 을 사용해야 하는 상황

그렇다면 `@Transactional` 을 사용해야 하는 상황은 언제일까 ?

개념적으로 이해한다면
트랜잭션은 **"어떤 작업을 하나의 논리적 단위로 보고" 성공/실패를 원자적으로 관리**할 때 필요하다  
작업을 모두 성공시키거나, 모두 취소시켜야 하는가를 기준으로 선택한다.  
-> 이것을 3개(쓰기 작업, 복잡한 읽기 작업, 트랜잭션이 필요없는 경우)의 큰 틀로 정의하고자 한다.

---

### 1.쓰기 작업("변경 작업")을 할 때

- insert, update, delete -> 무조건 트랜잭션이 필요하다.
- 하나라도 실패하면 데이터의 신뢰도가 떨어진다.
- 비즈니스 로직 단위로 트랜잭션을 묶어야한다.

### 2.복잡한 읽기 작업

- 평범한 `select`는 트랜잭션이 필요없다.
- 그런데 여러 테이블을 join 하거나 도중에 상태가 변할 위험이 있는 경우는
  ex) A 테이블 조회 -> B 테이블 조회 -> A 테이블의 상태가 다른 작업에 의해서 변할 수 있는경우

```java
@Transactional(readOnly = true)
public List<Product> getProductsAndOrders(Long customerId) {
    List<Product> products = productRepository.findByCustomerId(customerId);
    // 여기서 Order 상태를 같이 조회하고, 그 사이에 상태 변동이 있을 수 있는 경우
    return products;
}
```

이러한 경우에는 특별하게 "읽기 전용 트랜잭션" 을 이용한다.

```java
@Transactional(readonly = true)
```

`readOnly = true` 옵션을 주면

- 성능 최적화(DB가 dirty checking 같은 걸 안 함 → JPA를 사용하는 경우)
- 읽기 작업이라는 걸 명시적으로 표시할 수 있다.

### 3.트랜잭션이 필요 없는 경우

- 단순 조회
- 단순 계산(DB 관련 없는 경우)
- 외부 API만 호출

---

## ✨ 트랜잭션 사용법 정리

| 상황                        | 트랜잭션 필요 여부          | 비고                        |
| --------------------------- | --------------------------- | --------------------------- |
| insert/update/delete        | ✅ 필요                     | 데이터 손상 방지            |
| 단순 select                 | ❌ 불필요                   | 읽기 전용                   |
| 복잡한 select + 일관성 필요 | ✅ 필요 (readOnly 트랜잭션) | join/동시성 이슈 있을 때    |
| 계산, API 호출 등           | ❌ 불필요                   | DB 작업 없으면 안 걸어도 됨 |
