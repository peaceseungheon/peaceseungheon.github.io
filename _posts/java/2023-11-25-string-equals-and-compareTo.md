---
  layout: single
  title: String의 equals와 compareTo 메서드 비교
  categories:
    - JAVA
  toc: true
  toc_sticky: true
  tags:
    - JAVA
    - 문자열 비교
---

F-lab 멘토링 시간에 String 객체의 equals와 compareTo의 차이에 대해 질문이 있었다. 해당 질문에 잘 답변하지 못해 정확히 이해하고자 코드를 분석하면서 이해해보고자 글을 적는다.

## equals

```java
public boolean equals(Object anObject) {
  if (this == anObject) {
    return true;
  }
  return (anObject instanceof String aString)
        && (!COMPACT_STRINGS || this.coder == aString.coder)
        && StringLatin1.equals(value, aString.value);
}
```

코드는 아주 간단하다.

1. == 비교로 두 객체의 주소 값이 같으면 true를 반환.
2. 주소 값이 같이 않을 때는
    1. anObject가 String 객체여야하고
    2. COMPACT_STRING이 아니거나 this.coder == aString.coder가 같고
    3. StringLatin1.equals(value, aString.value) 결과가 true일 때
3. 2번에서의 조건 중 모두 참일 때 true를 반환하고 하나라도 아니면 false를 반환한다.

2번의 i는 당연한 조건이므로 패스하고 나머지를 하나씩 짚어보면

### COMPACT_STRING이 아니거나 this.coder == aString.coder가 같고

- COMPACT_STRING

COMPACT_STRING은 boolean 값으로 주석을 살펴보면 아래와 같이 설명되어 있다.

>If String compaction is disabled, the bytes in {@code value} are lways encoded in UTF16.

>The actual value for this field is injected by JVM. The static initialization block is used to set the value here to communicate that this static final field is not statically foldable, and to avoid any possible circular dependency during vm initialization.

요약해보자면 문자열에 대한 압축이 가능한지를 나타내는 값으로 JVM에서 할당하는 값이다. 영어 문자의 경우 1byte로 표현이 가능하지만 한글과 같이 유니코드 형식의 경우 2byte로 표현이 가능하다. 따라서 1byte로 압축할 수 있는 문자의 경우 압축하여 메모리에 저장하는 것이 성능적으로 유리하다. COMPACT_STRING은 압축여부를 의미하고 기본적으로 true가 할당되어 있다. 변경하려면 vmOption에서 설정해야 한다.

- coder

String 클래스에서 coder 인스턴스 변수는 byte 타입이고 해당 문자열의 인코딩 형식이 LATIN1이냐 UTF-16이냐를 식별하는 값이다. 쉽게 말해 영어면 0, 한글이면 1이다. 즉 문자열의 인코딩 방식이 같은지를 비교하는 것이다.

따라서 위 조건을 다시 해석해보면 문자열 압축이 가능하지 않다면( !COMPACT_STRINGS = = true ) 모두 UTF-16으로 인코딩하기 때문에 뒤의 조건인 인코딩 방식을 비교할 필요 없이 true, 문자열 압축이 가능하다면 같은 인코딩 방식인지 확인, 같으면 true 다르면 false.

### StringLatin1.equals(value, aString.value)

이 조건까지 오면 일단 두 문자열이 같은 인코딩 방식을 따른다는 셈이다. 따라서 문자열의 길이와 값을 검증하면 된다.

```java
public static boolean equals(byte[] value, byte[] other) {
  if (value.length == other.length) {
    for (int i = 0; i < value.length; i++) {
      if (value[i] != other[i]) {
        return false;
      }
    }
    return true;
  }
  return false;
}
```

코드에서 알 수 있듯이

1. 길이가 다르면 false
2. 길이가 같으면 반복문을 돌면서 같은 값인지 검증 하나라도 다르면 false, 모두 같으면 true

## compareTo

```java
public int compareTo(String anotherString) {
  byte v1[] = value;
  byte v2[] = anotherString.value;
  byte coder = coder();
  if (coder == anotherString.coder()) {
    return coder == LATIN1 ? StringLatin1.compareTo(v1, v2)
                          : StringUTF16.compareTo(v1, v2);
  }
  return coder == LATIN1 ? StringLatin1.compareToUTF16(v1, v2)
                        : StringUTF16.compareToLatin1(v1, v2);
}
```

위에 설명했던 내용을 바탕으로 보면 이해하는데 어렵지 않은 코드이다.

1. `coder == anotherString.coder()` : 두 문자열의 인코딩 방식이 같다면
    1. LATIN1 방식일 때는 StringLatin1.compareTo(v1, v2)
    2. 아닌 경우 StringUTF16.compareTo(v1, v2)의 값을 반환
2. 두 문자열의 인코딩 방식이 다르면
    1. LATIN1 방식일 때는 StringLatin1.compareToUTF16(v1, v2)
    2. 아닌 경우 StringUTF16.compareToLatin1(v1, v2)의 값을 반환

모든 메소드를 살펴보긴 어려우므로 StringLatin1.compareTo(v1, v2)의 코드를 확인해보자.

```java
public static int compareTo(byte[] value, byte[] other) {
  int len1 = value.length;
  int len2 = other.length;
  return compareTo(value, other, len1, len2);
}

public static int compareTo(byte[] value, byte[] other, int len1, int len2) {
  int lim = Math.min(len1, len2);
  for (int k = 0; k < lim; k++) {
    if (value[k] != other[k]) {
      return getChar(value, k) - getChar(other, k);
    }
  }
  return len1 - len2;
}

...

public static char getChar(byte[] val, int index) {
  return (char)(val[index] & 0xff);
}
```

`compareTo(byte[] value, byte[] other, int len1, int len2)`이 메소드가 핵심이다.

1. byte 배열의 길이가 같고 배열의 요소들이 모두 같으면 반복문을 통과하므로 0을 반환
2. byte 배열의 길이에 차이가 있다면
    1. 두 배열 중 가장 작은 배열의 길이만큼 반복문을 돈다.
    2. 만약 배열의 요소가 다르면 char타입으로 빼기 연산을 수행하고 그 값을 반환한다.
    3. 배열의 요소가 같다면 반복문을 통과하고 배열 길이의 차이만큼을 반환한다.

2번의 ii를 예시를 들어 설명해보면 a, b라는 char 타입의 값이 있을 때 두 값을 아스키 코드 값으로 변환해보면 a = 97, b = 98이다. 때문에 a - b는 -1을 반환한다.

정리해보자면 배열 요소가 다를 때는 각 요소의 char 타입의 빼기 연산만큼의 값을 반환한다.

이러한 compareTo 메소드의 특징 때문에 주로 문자열을 정렬하는데 사용된다.

## 결론

equals와 compareTo 메소드의 동작 방식을 살펴보면서 문자열 비교라는 비슷한 역할을 하지만 그 방식이 전혀 다르다는 걸 잘 알게 되었다. 또한 인코딩 방식이 문자열 비교에 큰 역할을 한다는 것도 새롭게 알게 됐다.
