# SpringBoot 에서, DI 란 무엇인가?

---

#### DI - Dependency Inejction

    한국말로 하면 '의존성  주입' 이라고 한다.

말로만 들어서는 선뜻 와닿지 않는 말이다.

의존성 주입? 무슨 말이지?

사실 말이 거창해서 그렇지 별건 아니다. 어떤 코딩 방식의 종류일 뿐.

임의의 class A 내부에서 사용하는 class B 를 어떤 식으로 선언하느냐의 문제이다.

예시를 보자.

```java
class Memory {
    int size;

    Memory(int size){
        this.size = size;
    }

    public void randomAccess(){
        System.out.println("randomAccess occurred.");
    }
}

class Computer {
    Memory mem = new Memory(16);

    public void getData(){
        mem.randomAccess();
    }
}
```

---

위의 코드에서는 Computer 의 객체 안에서 Memory 객체를 직접 생성하여 사용한다. 이를 두고 '두 객체의 의존성이 높다' 고 표현한다.

사실 의존성 주입에 대해 찾아 볼 때, 이에 대해 서술해 놓은 사람들이 모두

##### '이런 구조에서는 두 객체의 의존성이 높아질 수 밖에 없다'

는 말을 하더라. 나는 이게 잘 와닿지 않았는데, 다음 예시를 보니 알겠더라.

---

```java
class Memory {
    int size;

    Memory(int size){
        this.size = size;
    }

    public void randomAccess(){
        System.out.println("randomAccess occurred.");
    }
}

class Computer {
    Memory mem;

    Computer(Memory mem){
        this.mem = mem;
    }

    public void getData(){
        mem.randomAccess();
    }
}
```

---

첫 번째 예시와 뭐가 달라졌는지 눈치챘나?

Memory 객체를 Computer 객체 안에서 생성하지 않고 Computer 객체의 생성자에 Parameter 로 받아오는 것을 알 수 있다.

Computer 객체 안에서 사용될 Memory 객체를 Computer 객체의 바깥에서 주입받도록 변경 된 것이다.

왜 이런 짓을 하는걸까?

앞 서 말했듯이 두 객체간의 의존성을 낮추려고 하는 짓이다.

 '의존성을 낮춘다' 는 것은

#### '한번 작성한 코드로 여러 상황에 대응할 수 있다'

라는 의미라고 생각한다.

첫번째 예시에서는 

size 가 16 인 Memory 객체를 사용하는 Computer 와

size 가 32 인 Memory 객체를 사용하는 Computer 를 사용하려면 각각 다른 Computer 객체를 만들어야 한다.

Computer 객체가 Memory 객체와 의존성이 높기 때문에 다른 Memory 객체를 사용하려면 코드의 분기가 불가피해진다.

그러나 두번째 예시를 사용한다면

Computer 객체를 생성할 때 size 가 서로 다른 Memory 객체를 만들어 넣어주면 하나의 코드로 두가지 상황에 대응이 가능하다.

---

이를 Dependency Inejction, 즉 의존성 주입이라 한다.

Java Spring 프레임워크를 알아보다가 알게 된 개념인데, 사실 이름만 몰랐지 이미 개발을 하면서 사용하고 있는 개념이다. 다만 이렇게 명확하게 정리하고 있었던 것은 아닌데 이 기회에 좋은 공부를 한 것 같다.

참고로, Java Spring 프레임 워크에서는 위의 예시와 같이 생성자를 통해 DI 를 구현하는 방법도 있고, @Autowired 와 같은 annotation 을 이용한 방법도 있다.

방법이야 어찌 되었든 간에, 위에서 설명한 개념을 위한 코딩 패러다임일 뿐이니 해당 개념을 이해만 하고 있으면 좋을 것 같다. 구현 방법이야 찾아보면 되니까...

근데 그러고 보니 Java 에서 annotaion 이 정확히 어떤 거고 어떤 식으로 돌아가는 건지 잘 모르겠다. 이거 한번 찾아봐야겠네.

또 Dependency Injection 이 단위 테스트를 하기 위해 아주 좋은 구현 방식이라고 하던데, 어렴풋이 이해는 되지만 이에 대해서도 알아봐야겠다.

- [ ] 
