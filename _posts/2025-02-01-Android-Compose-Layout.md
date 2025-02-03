---
title: 안드로이드 컴포즈 레이아웃 1.Column, Row, Box
author: minjo
date: 2025-02-01 10:00:00 +08
categories: [Android, Layout]
tags: [jetpack compose - layout]
math: true
toc: true
pin: true
mermaid: true
---

# 안드로이드 컴포즈 레이아웃 1.Column, Row, Box

{: .mt-6 .mb-0 }

{: .mt-4 .mb-0 }

**Compose 구성방식을 설정하지 않은 경우**:

{: .mt-2 .mb-0 }

1. compose의 구성 가능한 함수를 내보낼 수 있는데, 구성가능한 방식을 설정하지 않을경우.
2. 텍스트 요소를 겹치게 보이게 됩니다.

{: .mt-4 .mb-0 }

```kotlin
@Preview
@Composable
fun ArtistCard() {
    Text("레이아웃을 잡아주지 않으면")
    Text("겹치게 됩니다.")
}
```

<img src="assets/img/for_post/20250201-1.png" style="width: 100%; height: auto;">

## 표준 레이아웃 구성요소

![Desktop View](assets/img/for_post/20250201-2.png){: .normal }

### Column - 수직 정렬

```kotlin
@Preview
@Composable
fun ColumnCard() {
    Column(modifier = Modifier.background(Color.White)) {
        Text("Column은 수직정렬")
        Text("수직정렬완료")
    }
}
```

<img src="assets/img/for_post/20250201-3.png" style="width: 100%; height: auto;">

2. Column 요소를 사용하여 수직 정렬이 가능하다.

### Row - 수평 정렬

```kotlin
@Preview
@Composable
fun RowCard() {
    Row(
        modifier = Modifier.background(Color.White)
    ) {
            Text("Row는 수평정렬")
            Text("수평정렬")
        }
}
```

<img src="assets/img/for_post/20250201-4.png" style="width: 100%; height: auto;">

1. Row를 설정하여 수평정렬이 가능하다.

### Box - 요소위에 요소가 겹치도록

```kotlin
@Preview
@Composable
fun BoxCard() {
    Row(
        modifier = Modifier.background(Color.White)
    ) {
            Text("Row는 수평정렬")
            Text("수평정렬")
        }
}
```

<img src="assets/img/for_post/20250201-5.png" style="width: 100%; height: auto;">

1.  Box를 사용하여 요소를 다른 위에 요소 위에 겹쳐둘 수 있다.
2.  Compose 구성방식을 사용하지 않는 경우와의 차이는 Box는 영역지정과, 요소의 위치를 지정하는 것이 가능하다는 점이다.
