# 1.基本介绍

Java 8 Lambda 表达式的代码实际操作例子，主要是把jdk8里面的lambda常用的例子摆一摆，忘记了看一看就知道怎么使用，方便回忆。

# 2.怎么使用

### 2.1.使用的循环体字段的代码，也就是操作对象，数据结构之类的，这里是简单的 list 数据。

```
private final List<BigDecimal> prices = Arrays.asList(
            new BigDecimal("10"), new BigDecimal("30"), new BigDecimal("17"),
            new BigDecimal("20"), new BigDecimal("15"), new BigDecimal("18"),
            new BigDecimal("45"), new BigDecimal("12"));

    private final List<String> friends = Arrays.asList("Brian", "Nate", "Neal", "Raju", "Sara", "Scott");
    private final List<String> editors = Arrays.asList("Brian", "Jackie", "John", "Mike");
    private final List<String> comrades = Arrays.asList("Kate", "Ken", "Nick", "Paula", "Zach");
    private final List<Person> people = Arrays.asList(
            new Person(20, "John"),
            new Person(21, "Sara"),
            new Person(21, "Jane"),
            new Person(35, "Greg")
    );
```

### 2.2.初识，简单对比lambda和常规操作的不同

```
   /**
     * 假设超过20块的话要打九折
     */
    private void test11() {
        BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;
        for (BigDecimal price : prices) {
            if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
                totalOfDiscountedPrices = totalOfDiscountedPrices.add(price.multiply(BigDecimal.valueOf(0.9)));
            }
        }
        System.out.println("Total of discounted prices: " + totalOfDiscountedPrices);
    }

    private void test12() {
        final BigDecimal totalOfDiscountedPrices =
                prices.stream()
                        .filter(price -> price.compareTo(BigDecimal.valueOf(20)) > 0)
                        .map(price -> price.multiply(BigDecimal.valueOf(0.9)))
                        .reduce(BigDecimal.ZERO, BigDecimal::add);
        System.out.println("Total of discounted prices: " + totalOfDiscountedPrices);
    }
```

运行结果：

![](/static/image/20200422141523610.png)

### 2.3.集合的使用

```
   /**
     * 集合的使用
     */
    @Test
    public void test2() {
        friends.forEach(new Consumer<String>() {
            @Override
            public void accept(final String name) {
                System.out.println(name);
            }
        });

        friends.forEach((final String name) -> System.out.println(name));

        friends.forEach((name) -> System.out.println(name));

        friends.forEach(name -> System.out.println(name));

        friends.forEach(System.out::println);
    }
```

运行结果：  
![](/static/image/2020042214154563.png)  
都是从传统的写法上，慢慢的过度到Lambda的写法。

### 2.4.列表的转化

```
 private void test31() {
        final List<String> uppercaseNames = new ArrayList<String>();
        for (String name : friends) {
            uppercaseNames.add(name.toUpperCase());
        }
        System.out.println(uppercaseNames);
    }

    private void test32() {
        final List<String> uppercaseNames = new ArrayList<String>();
        friends.forEach(name -> uppercaseNames.add(name.toUpperCase()));
        System.out.println(uppercaseNames);
    }

    /**
     * Steam的map方法可以用来将输入序列转化成一个输出的序列
     * map方法把lambda表达式的运行结果收齐起来，返回一个结果集
     */
    private void test33() {
        friends.stream()
                .map(name -> name.toUpperCase())
                .forEach(name -> System.out.print(name + " "));
        System.out.println();
    }

    /**
     *
     */
    private void test34() {
        friends.stream()
                .map(name -> name.length())
                .forEach(count -> System.out.print(count + " "));
    }

    /**
     *
     */
    private void test35() {
        friends.stream()
                .map(String::length)
                .forEach(count -> System.out.print(count + " "));
    }
```

运行结果：

![img](/static/image/20200422141625541.png)  
都是从传统的写法上，慢慢的过度到Lambda的写法。

### 2.5.在集合中查找元素

```
 private void test41() {
        final List<String> startsWith = new ArrayList<String>();
        for (String name : friends) {
            if (name.startsWith("N")) {
                startsWith.add(name);
            }
        }
        System.out.println(startsWith);
    }

    /**
     * filter方法接收一个返回布尔值的lambda表达式。
     * 如果表达式结果为true，运行上下文中的那个元素就会被添加到结果集中;
     * 如果不是，就跳过它。最终返回的是一个Steam，它里面只包含那些表达式返回true的元素。
     * 最后我们用一个collect方法把这个集合转化成一个列表
     */
    private void test42() {
        final List<String> startsWith =
                friends.stream()
                        .filter(name -> name.startsWith("N"))
                        .collect(Collectors.toList());
        System.out.println(startsWith);
    }
```

运行结果：

![](/static/image/2020042214164436.png)

### 2.6.lambda表达式的重用



