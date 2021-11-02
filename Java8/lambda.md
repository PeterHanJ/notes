## lambda中局部变量引用的问题

```java
public static void main(String[] args) {
        List<Apple> apples = AppleFilter.getApples();
        // local variable must be final or not changed
        // which can be used in lambda
        int i = 0;
        Integer ref = new Integer(1);

        apples.stream().filter(apple -> {
            TestLocalVariable l = new TestLocalVariable();
            l.t += 12;
            //Variable used in lambda expression should be final or effectively fina
            //i++;
            System.out.println("-------------------------");
            //can use if not chage
            System.out.println(ref);
            System.out.println(l.t);
            System.out.println("-------------------------");
            return apple.getColor().equals("red");

        }).collect(Collectors.toList());
        //if local variable changed, compile error in lambda
        //i = 1;
        //ref = new Integer(2);
    }
```

