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


