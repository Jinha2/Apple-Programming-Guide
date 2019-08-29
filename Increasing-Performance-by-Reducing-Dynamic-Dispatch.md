# Increasing Performance by Reducing Dynamic Dispatch

[Increasing Performance by Reducing Dynamic Dispatch - Swift Blog](https://developer.apple.com/swift/blog/?id=27)

다른 많은 언어들 처럼, Swift는 클래스에서 superclass에 선언된 메소드와 프로퍼티를 override 할 수 있습니다. 이것은 프로그램이 런타임에 어떤 메소드나 프로퍼티가 참조 되었는지 결정해야 하고 직접 호출 또는 간접 접근을 수행해야 한다는 것을 의미합니다. 다이나믹 디스패치(dynamic dispath)로 불리는 이 기술은, 각각의 간접적 사용에 따라 거듭되는 런타임 오버헤드의 비용으로 언어의 표현성을 증가시킵니다. 성능에 민감한 코드에서 그러한 오버헤드는 바람직하지 않습니다. 이 글은 dynamism을 제거함으로써(final, private, 모듈 최적화) 성능을 향상시키는 세가지 방법을 보여줍니다. 

아래 예시를 살펴 봅시다.

    class ParticleModel {
    	var point = ( 0.0, 0.0 )
    	var velocity = 100.0
    
    	func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
    		point = newPoint
    		velocity = newVelocity
    	}
    
    	func update(newP: (Double, Double), newV: Double) {
    		updatePoint(newP, newVelocity: newV)
    	}
    }
    
    var p = ParticleModel()
    for i in stride(from: 0.0, through: 360, by: 1.0) {
    	p.update((i * sin(i), i), newV:i*1000)
    }

작성 된 대로, 컴파일러는 동적으로 디스패치 호출 할 것입니다. 다음을 하기 위해:

1. p의 update를 호출
2. p의 updatePoint 호출
3. p의 point 튜플 프로퍼티 접근
4. p의 velocity 프로퍼티 접근

이것이 당신이 이 코드를 보고 기대한 것이 아닐 수도 있습니다.ParticleModel의 서브클래스들이 연산 프로퍼티로 point 또는 velocity를 override하거나 새로운 구현으로 updatePoint()나 update() 로 인해 동적 호출은  필요합니다.

Swift에서, 동적 디스패치 호출은 메소드 테이블에서 함수를 찾고 간접적인 호출을 수행하도록 구현되었습니다. 이것은 직접 호출을 수행하는 것보다는 느립니다. 게다가, 간접 호출은 많은 컴파일러 최적화를 방지하기 때문에, 더 많은 비용이 들게 합니다. 성능이 중요한 코드에서 성능 향상을 필요 할 때 이런 동적인 행동을 제한 할 수 있는 기술이 있습니다. 

### override 할 필요가 없는 속성 선언을 할 때 final을 사용하라

final 키워드는 클래스, 메소드 또는 프로퍼티의 선언이 override 될 수 없다는 제한을 해줍니다. 이것은 컴파일러가 간접 동적 디스패치를 안전하게 생략 할 수 있도록 해준다. 예를 들어, 아래의 point와 velocity는 객체의 저장 프로퍼티를 통해 간적접으로 접근되고 updatePoint()는 직접 함수 호출을 통해 호출 될 것입니다. 반대로, update()는 서브클래스가 커스터마이징 된 함수로 update()를 오버라이드 할 수 있도록 허용하면서 여전히 동적 디스패치를 통해 호출 됩니다.

    class ParticleModel {
    	final var point = ( x: 0.0, y: 0.0 )
    	final var velocity = 100.0
    
    	final func updatePoint(newPoint: (Double, Double), newVelocity: Double) {
    		point = newPoint
    		velocity = newVelocity
    	}
    
    	func update(newP: (Double, Double), newV: Double) {
    		updatePoint(newP, newVelocity: newV)
    	}
    }

전체 클래스에 final을 표시 가능합니다.