# Comparable을 구현할지 고려하라
compareTo는 다른 메서드들과 다르게 Object의 메서드가 아니다. 2가지 성격만 제외한다면, Object의 equals와 같다. <br>
1. compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.
   -> Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 존재하다는 것을 뜻한다. <br>
   -> 그래서 Comparable을 구현한 객체들의 배열은 `Arrays.sort(a);` 처럼 손쉽게 정렬할 수 있다. <br>
<br>

## compareTo 메서드의 일반 규약
compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다. <br>

이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수로 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다. <br>
다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다. <br>

- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compartTo(y)) == -sgn.(y.compareTo(x))여야 한다(따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다).
- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.
- 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. (x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다.
  "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다." <br>

  <br>
모든 객체에 대해 전역 동치관계를 부여하는 equals 메서드와 달리, compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다. <br>
이 규약에서는 다른 타입 사이의 비교도 허용하는데, 보통은 비교할 객체들이 구현한 공통 인터페이스를 매개로 이뤄진다. <br>

<br>
hashCode 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. <br>

## compareTo 규약
1. 첫 번째 규약은 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 것이다.
2. 두 번째 규약은 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다는 뜻이다.
3. 세 번째 규약은 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 뜻이다.
<br>
위 규약들은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다.
<br>
<br>
## compareTo 메서드 작성 요령
compareTo 메서드 작성 요령은 equals와 비슷하다. 몇 가지 차이점만 주의하면 된다. <br>
- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다.
- 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다. 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.
- null을 인수로 넣어 호출하면 NullPointerException을 던져야 하며, 물론 실제로도 인수(이 경우 null)의 멤버에 접근하려는 순간 이 예외가 던져질 것이다.
- compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다.
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.
- 비교자는 직접 만들거나 자바가 제공하는 것 중에 골라 쓰면 된다.

### 객체 참조 필드가 하나뿐인 비교자
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private String s;

    @Override
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    // ...
}
```
위 코드는 CaseInsensitiveString이 Comparable을 구현했다. 이는 CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있다는 뜻이며, 일반적으로 Comparable을 구현할 때 따르는 패턴이다. <br>
자바 7부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용하면 된다. <br>
compareTo 메서드에서 관계 연산자 <와 >를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다. <br>

### 기본 타입 필드가 여럿일 때의 비교자
```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private Short areaCode;
    private Short prefix;
    private Short lineNum;

    @Override
    public int compareTo(PhoneNumber phoneNumber) {
        int result = Short.compare(areaCode, phoneNumber.areaCode); // 가장 중요한 필드
        if (result == 0) {
            result = Short.compare(prefix, phoneNumber.prefix); // 두 번째로 중요한 필드
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber.lineNum); // 세 번째로 중요한 필드
            }
        }
        return result;
    }
}
```
위 코드는 클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해지는데, 가장 핵심적인 필드부터 비교하도록 한다. <br>
비교 결과가 0이 아니라면, 즉 순서가 결정되면 거기서 끝이며 결과를 곧장 반환해야 한다. <br>
가장 핵심이 되는 필드가 똑같다면, 똑같지 않은 필드를 찾을 때까지 그다음으로 중요한 필드를 비교해 나간다. <br>

<br> <br>

## 자바 8과 Comparator 인터페이스
자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는 데 활용할 수 있다. <br>

### 비교자 생성 메서드를 활용한 비교자
```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
            .thenComparingInt(phoneNumber -> phoneNumber.prefix)
            .thenComparingInt(phoneNumber -> phoneNumber.lineNum);

    private Short areaCode;
    private Short prefix;
    private Short lineNum;


    public int compareTo(PhoneNumber phoneNumber) {
        return COMPARATOR.compare(this, phoneNumber);
    }
}
```
이 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다. <br>
그 첫 번째인 comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드다. <br>
두 번째 비교자 생성 메서드인 thenComparingInt는 Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력받아 다시 비교자를 반환한다.  <br>
thenComparingInt는 원하는 만큼 연달아 호출할 수 있다. <br>
자바의 타입 추론 능력이 최초의 comparingInt를 호출할 때에는 타입을 알아낼 만큼 강력하지 않기 때문에 입력 인수의 타입인 PhoneNumber를 명시해줬지만, 그 이후에 thenComparingInt를 호출할 때는 충분히 타입 추론이 가능하므로 명시하지 않았다. <br>
<br>

## 해시코드 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다!
이따금 '값의 차'를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo나 compare 메서드와 마주할 것이다. <br>

```java
public class HashCodeOrderComparator {
    static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
    };
}
```

위 코드 방식은 사용하면 안 된다. 이 방식은 정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다. <br>

<br>

다음 두 방식 중 하나를 사용하자. <br>

## 정적 compare 메서드를 활용한 비교자
```java
static Comparator<Object> staticHashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
    };
```

## 비교자 생성 메서드를 활용한 비교자
```java
static Comparator<Object> comparatorHashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

<br>

## ⭐️ 핵심 정리
순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하는 것이 좋다.
