## 어노테이션의 어노테이션

### 정리

특정한 속성을 가진다는 것을 알려주는 마커 인터페이스는 유용하다. 마커 어노테이션은 런타임에서야 오류를 잡아내지만, 마커 인터페이스는 컴파일 타임에서 오류를 잡아낼 수 있다. 대표적인 예가 Serializable이다. 또한, 애너테이션이 `ElementType.TYPE`으로 설정되면 추가적으로 세밀한 타입을 지정할 수는 없다. 하지만 마커 인터페이스는 특정 인터페이스의 하위 타입으로 설정할 수도 있다. 대신, 마커 애너테이션은 프레임워크 수준에서 지원을 통해서 사용의 일관성을 지원받을 수도 있다. 중요한 것은 마킹된 객체가 매개변수로 받는 경우가 있는지를 고려하는 것이다. 결국 마

### 심화 탐구

**출발점**

인터페이스를 작성하는 법은 어노테이션을 작성하는 법보다 익숙하다. 그렇다면 어노테이션은 어떻게 작성되고, `Element.TYPE`은 무엇인가? 개발 과정에서의 효율을 높여주는 애노테이션은 어떻게 작성되는지 살펴보겠다. 특히, 아주 익숙한 오버라이드 메소드가 좋은 예시가 될 것이다.

**설명**

1. `@Override`의 예시

```java

package java.lang;

import java.lang.annotation.*;

/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 8.4.8 Inheritance, Overriding, and Hiding
 * @jls 9.4.1 Inheritance and Overriding
 * @jls 9.6.4.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

```

`@Override` 어노테이션의 소스코드는 위와 같다. `@Target`에서는 해당 어노테이션을 적용가능한 대상을, `@Retention`에서는 어노테이션이 유지되는 정책(기간)을 설정한다. `ElementType`의 대상은 TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR 등이 있다. 이는 직관적이니 넘어가겠다. `Retention`의 대상으로는 SOURCE, CLASS, RUNTIME이 있다. SOURCE는 어노테이션이 소스코드에서만 존재하고 컴파일 시에는 사라지며, CLASS는 클래스 파일에는 포함되지만 JVM에서는 인식하지 않고, RUNTIME은 JVM에 읽혀지며 리플렉션으로도 접근가능하다. 

즉, `@Override`는 메소드를 대상으로 사용이 가능하며 소스코드에서만 존재하는 어노테이션이라고 볼 수 있겠다.

2. `@Target`

자바 어노테이션에서 `ElementType.TYPE`은 어노테이션이 적용될 수 있는 대상을 정의한다. 이는 `@Target`과 함께 
쓰이면서, 해당 어노테이션이 적용되는 대상이 클래스, 인터페이스, Enum 열거형임을 알려준다. 그러면 `@Target` 어노테이션은 어떻게 구성되어 있을까?

```java

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation interface
     * can be applied to.
     * @return an array of the kinds of elements an annotation interface
     * can be applied to
     */
    ElementType[] value();
}

```

놀랍게도 `@Target` 어노테이션에도 `@Target`이 적용되어 있다. 이는 `@Target` 어노테이션이 우선적으로 정의되기에 자신에게 붙여도 오류가 일어나지 않는 까닭이다.

또한 주의해야할 점은, 어노테이션 작성시 `@Target`에는 하나의 ElementType만 올 수 있다는 것이다. 실제로 Oracle 문서에서도 아래와 같은 코드를 이용할 수 없다고 명시한다.

```java

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.FIELD})
public @interface Bogus {
    ...
}

```

### 나가며

어노테이션은 메타데이터를 제공함으로써, 리플렉션을 통해 해당 어노테이션이 붙어있는 ElementType에 대해 특정한 작업을 처리할 수 있도록 돕는다. 실습으로 어노테이션을 생성하고, 리플렉션을 통해 그러한 메타데이터가 있는 작업을 처리하는 코드를 작성해보아도 좋겠다. 

### References

https://docs.oracle.com/en%2Fjava%2Fjavase%2F11%2Fdocs%2Fapi%2F%2F/java.base/java/lang/annotation/Target.html

https://stackoverflow.com/questions/15101139/how-targetelementtype-annotation-type-works