# 10.3 자바로 DSL을 만드는 패턴과 기법

간단한 도메일 모델 정의 → DSL을 만드는 패턴 

```java
// 주식 가격을 모델링하는 순수 자바 빈즈
public class Stock {
    private String symbol;
    private String market;
    public String getSymbol(){
        return symbol;
    }
    public void setSymbol(String symbol) {
        this.symbol = symbol;
    }
    ...
}

// 주어진 가격에서 주어진 양의 주식을 사거나 파는 거래
public class Trade {
    public enum Type { BUY, SELL }
    
    private Type type;
    private Stock stock;
    private int quantity;
    private double price;
    ...    
}

// 고객이 요청한 한 개 이상의 거래 주문
public class Order {
    private String customer;
    private List<Trade> trades = new ArrayList<>();
    ...
}
```

하지만 이러한 코드는 장황하고 비개발자인 도메인 전문가가 이해하고 검증하기 어렵다.

직관적으로 도메인 모델을 반영할 수 있는 DSL 이 필요하다. 이 책에서는 다음과 같은 DSL 패턴들을 소개하고 있다.

- 메서드 체인
- 중첩된 함수
- 람다 표현식을 이용한 함수 시퀀싱

### **메서드 체인**

메서드 체인은 DSL 에서 가장 흔한 방식 중 하나이다.

- 장점
    - 사용자가 지정된 절차에 따라 플루언트 API의 메서드 호출을 강제
    - 파라미터가 빌더 내부로 국한 됨
    - 메서드 이름이 인수의 이름을 대신하여 가독성 개선
    - 문법적 잡음이 최소화
    - 선택형 파라미터와 잘 동작
    - 정적 메서드 사용을 최소화하거나 없앨 수 있음
- 단점
    - 빌더를 구현해야 함
    - 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요
    - 도멘인 객체의 중첩 구조와 일치하게 들여쓰기를 강제할 수 없음

```java
Order order = forCustomer("BigBank")
    .buy(80)
    .stock("IBM")
    .on("NYSE")
    ...
    .end();
```

이러한 코드를 만들기 위해서는 최상 수준 빌더를 만들고 주문을 감싸서 거래를 추가할 수 있도록 해야 한다.

```java
public class MethodChainingOrderBuilder {

    public final Order order = new Order();
    
    private MethodChainingOrderBuilder(String customer) {
        order.setCustomer(customer);
    }
    public static MethodChainingOrderBuilder forCustomer(String customer) {
        return new MethodChainingOrderBuilder(customer);
    } 
    public TradeBuilder buy(int quantity) {
        return new TradeBuilder(this, Trade.Type.SELL, quantity);
    }
    public MethodChainingOrderBuilder addTrade(Trade trade) {
        order.addTrade(trade);
        return this;
    }
    public Order end() {
        return order;
    }
}

public class TradeBuilder {
    private final MethodChainingOrderBuilder builder;
    public final Trade trade = new Trade();
    
    private TradeBuilder(MethodChainingOrderBuilder builder, Trade.Type type, int quantity) {
        this.builder = builder;
        trade.setType(type);
        trade.setQuantity(quantity);
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(builder, trade, symbol);
    }
}

public class StockBuilder {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    private StockBuilder(MethodChainingOrderBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    } 
    public TradeBuilderWithStock on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return new TradeBuilderWithStock(builder, trade);
    }    
}

public class TradeBuilderWithStock {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    
    public TradeBuilderWithStock(MethodChainingOrderBuilder builder, Trade trade) {
        this.builder = builder;
        this.trade = trade;
    }
    public MethodChainingOrderBuilder at(double price) {
        trade.setPrice(price);
        return builder.addTrade(trade);
    }
}
```

### **중첩된 함수 이용**

중첩된 함수 DSL 패턴은 다른 함수 안에 함수를 이용해 도메인 모델을 만든다.

- 장점
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영
    - 구현의 장황함을 줄일 수 있음
- 단점
    - 결과 DSL 에 더 많은 괄호를 사용
    - 정적 메서드 사용이 빈번
    - 인수 목록을 정적 메서드에 넘겨줘야 함
    - 인수의 의미가 이름이 아니라 위치에 의해 정의 됨
    - 도메인에 선택 사항 필드가 있으면 인수를 생략할 수 있으므로 메서드 오버로딩 필요

```java
Order order = order("BigBank",
                    buy(80, stock("IBM", on("NYSE")), at(125.00)),
                    sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
                    );
```

구현하는 코드는 다음과 같다.

```java
public class NestedFunctionOrderBuilder {

    public static Order order(String customer, Trade... trades) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }
    public static Trade buy(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.BUY);
    }    
    private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type type) {
        Trade trade = new Trade();
        trade.setQuantity(quantity);
        trade.setType(type);
        trade.setStock(stock);
        trade.setPrice(price);
        return trade;
    }
    public static double at(double price) {
        return price;
    }
    publid static Stock stock(String Symbol, String market) {
        Stock stock = new Stock();
        stock.setSymbol(symbol);
        stock.setMarket(market);
        return stock;
    }
}
```

### **람다 표현식을 이용한 함수 시퀀싱**

람다 표현식으로 정의한 함수 시퀀스를 사용한다.

- 장점
    - 플루언트 방식으로 도메인 객체 정의 가능
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영
    - 선택형 파라미터와 잘 동작
    - 정적 메서드를 최소화하거나 없앨 수 있음
    - 빌더의 접착 코드가 없음
- 단점
    - 설정 코드가 필요
    - 람다 표현식 문법에 의한 잡음의 영향을 받음

```java
Order order = order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        });
    });
})
```

이런 DSL을 만들기 위해서는 람다 표현식을 받아 실행해 도메인 모델을 만드는 빌더를 구현해야 한다.

```java
public class LambdaOrderBuilder {

    private Order order = new Order();
    
    public static Order order(Consumer<LambdaOrderBuilder> consumer) {
        LambdaOrderBuilder builder = new LambdaOrderBuilder();
        consumer.accept(builder);
        return builder.order;
    }
    public void forCustomer(String customer) {
        order.setCustomer(customer);
    }
    public void buy(Consumer<TradeBuilder> consumer) {
        trade(consumer, Trade.Type.BUY);
    }
    private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(type);
        consumer.accept(builder);
        order.addTrade(builder.trade);
    }
}

public class TradeBuilder {

    private Trade trade = new Trade();
    
    public void quantity(int quantity) {
        trade.setQuantity(quantity);
    }
    public void price(double price) {
        trade.setPrice(price);
    }
    public void stock(Consumer<StockBuilder> consumer) {
        StockBuilder builder = new StockBuilder();
        consumer.accept(builder);
        trade.setStock(builder.stock);
    }
}
...
```

### **조합하기**

중첩된 함수 패턴과 람다 기법을 혼용하면 다음과 같이 사용할 수 있다.

- 장점
    - 가독성 향상
- 단점
    - 여러 기법을 혼용했기 때문에 사용자가 DSL을 배우는 시간이 오래 걸림

```java
Order order = 
    forCustomer("BigBank", 
                buy(t -> t.quantity(80)
                          .stock("IBM")
                          .on("NYSE")
                          .at(125.00)),
                sell(t -> t.quantity(50)
                           .stock("GOOGLE")
                           .on("NASDAQ")
                           .at(125.00)));
```

이 코드를 사용하기 위한 빌더 코드는 다음과 같다.

```java
public class MixedBuilder {

    public static Order forCustomer(String customer, TradeBuilder... builders) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(builders).forEach(b -> order.addTrade(b.trade));
        return order;
    } 
    public static TradeBuilder buy(Consumer<TradeBuilder> consumer) {
        return builderTrade(consumer, Trade.Type.BUY);
    }
    private static TradeBuilder buildTrade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(buy);
        consumer.accept(builder);
        return builder;
    }
}

public class TradeBuilder {
    
    private Trade trade = new Trade();
    
    public TradeBuilder quantity(int quantity) {
        trade.setQuantity(quantity);
        return this;
    }
    public TradeBuilder at(double price) {
        trade.setPrice(price);
        return this;
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(this, trade, symbol);
    }
}

public class StockBuilder {
    
    private final TradeBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    public StockBuilder(TradeBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    }
    public TradeBuilder on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return builder;
    }
}
```

### **DSL에 메서드 참조 사용하기**

주문의 총 합에 세금을 추가해 최종값을 계산하는 기능을 추가한다.

```java
public class Tax {
    public static double regional(double value) {
        return value * 1.1;
    }
    public static double general(double value) {
        return value * 1.3;
    }
    public static double surcharge(double value) {
        return value * 1.05;
    }
}
```

함수형 기능을 이용하여 간결하고 유연한 방식으로 구현한다.

```java
public class TaxCalculator {

    public DoubleUnaryOperator taxFunction = d -> d;
    
    public TaxCalculator with(DoubleUnaryOperator f) {
        taxFunction = taxFunction.andThen(f);
        return this;
    }
    public double calculate(Order order) {
        return taxFunction.applyAsDouble(order.getValue());
    }
}

double value = new TaxCalculator().with(Tax::regional)
                                  .with(Tax::surcharge)
                                  .calculate(order);
```
