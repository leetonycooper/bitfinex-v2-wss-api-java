# crypto-bot

<a href="https://travis-ci.org/jnidzwetzki/bitfinex-v2-wss-api-java">
  <img alt="Build Status" src="https://travis-ci.org/jnidzwetzki/bitfinex-v2-wss-api-java.svg?branch=master">
</a>
<a href="https://repo1.maven.org/maven2/com/github/jnidzwetzki/"><img alt="Maven Central Version" src="https://maven-badges.herokuapp.com/maven-central/com.github.jnidzwetzki/bitfinex-v2-wss-api/badge.svg" />
  </a><a href="https://codecov.io/gh/jnidzwetzki/bitfinex-v2-wss-api-java">
  <img src="https://codecov.io/gh/jnidzwetzki/bitfinex-v2-wss-api-java/branch/master/graph/badge.svg" />
</a>

This project contains a client for the [Bitfinex WebSocket API (v2)](https://docs.bitfinex.com/v2/reference). At the moment, historical candles, orders, ticks, and wallets are supported.

**Warning:** Trading carries significant financial risk; you could lose a lot of money. If you are planning to use this software to trade, you should perform many tests and simulations first. This software is provided 'as is' and released under the _Apache 2.0 license_. 

# Integrating the library into your project

Add this to your pom.xml 

```xml
<dependency>
	<groupId>com.github.jnidzwetzki</groupId>
	<artifactId>bitfinex-v2-wss-api</artifactId>
	<version>0.0.1</version>
</dependency>
```

# Examples

## Connecting and authorizing

```java 
final String apiKey = "....";
final String apiSecret = "....";

// For public operations (subscribe ticker, bars)
BitfinexApiBroker bitfinexApiBroker = BitfinexApiBroker();
bitfinexApiBroker.connect();

// For public and private operations (executing orders, read wallets)
BitfinexApiBroker bitfinexApiBroker = BitfinexApiBroker(apiKey, apiSecret);
bitfinexApiBroker.connect();
```

## Subscribe candles

```java
final BiConsumer<String, Tick> callback = (symbol, tick) -> {
	System.out.println("Got tick for symbol: " + symbol + " / " + tick;
};

bitfinexApiBroker.getTickerManager().registerTickCallback("tUSDBTC", callback);
bitfinexApiBroker.sendCommand(new SubscribeCandlesCommand("tUSDBTC", Timeframe.MINUTES_1));
```

## Market order

```java
final BitfinexOrder order = BitfinexOrderBuilder
		.create(currency, BitfinexOrderType.MARKET, 0.002)
		.build();
		
bitfinexApiBroker.placeOrder(order);
```

## Order group

```java
final CurrencyPair currencyPair = CurrencyPair.BTC_USD;
final Tick lastValue = bitfinexApiBroker.getLastTick(currencyPair);

final int orderGroup = 4711;

final BitfinexOrder bitfinexOrder1 = BitfinexOrderBuilder
		.create(currencyPair, BitfinexOrderType.EXCHANGE_LIMIT, 0.002, lastValue.getClosePrice().toDouble() / 100.0 * 100.1)
		.setPostOnly()
		.withGroupId(orderGroup)
		.build();

final BitfinexOrder bitfinexOrder2 = BitfinexOrderBuilder
		.create(currencyPair, BitfinexOrderType.EXCHANGE_LIMIT, -0.002, lastValue.getClosePrice().toDouble() / 100.0 * 101)
		.setPostOnly()
		.withGroupId(orderGroup)
		.build();

// Cancel sell order when buy order failes
final Consumer<ExchangeOrder> ordercallback = (e) -> {
		
	if(e.getCid() == bitfinexOrder1.getCid()) {
		if(e.getState().equals(ExchangeOrder.STATE_CANCELED) 
				|| e.getState().equals(ExchangeOrder.STATE_POSTONLY_CANCELED)) {
			bitfinexApiBroker.cancelOrderGroup(orderGroup);
		}
	}
};

bitfinexApiBroker.getOrderManager().addOrderCallback(ordercallback);

bitfinexApiBroker.placeOrder(bitfinexOrder1);
bitfinexApiBroker.placeOrder(bitfinexOrder2);
```
