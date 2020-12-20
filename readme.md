Made by [al6x](http://al6x.com).

I do **Statstics**, **Analytics**, **Visualization**, $110/hr.

# What it is

**Simple and clean HTTP API** and Kotlin/Java API to Interactive Brokers.

It will automatically connect and disconnect to running TWS UI and exposes its messy Evented API
as **simple HTTP API**.

For API docs see [IB.kt](https://github.com/alexeypetrushin/ib_api/blob/master/src/ib/ib.kt)

# Notes

It's better to specify `exhange` and `currency` explicitly.

It's better to first get stock contract ID and then use that ID to get stock price. Because for some
stocks like "VAR1 IBIS:EUR" price is not available if asked on "IBIS", but available on "SMART".

Exchange for options could be different from the exchange for stock. The stock "VAR1 EUR" traded
on "IBIS" but its option traded on "DTB".

# Examples

Use implementation of `/api/v1/inspect_stock?symbol=VAR1` as example, it uses many API methods
to display information about the stock.

Example, **get stock price**:

```
http://localhost:8001/api/v1/stock_price?symbol=MCD&exchange=NYSE&currency=USD

{
  "price": 215.37,
  "datatype": "realtime",
  "price_type": "lastPrice"
}
```

Example, **get option chain**:

```
http://localhost:8001/api/v1/stock_option_chains?symbol=SIVR&exchange=ARCA&currency=USD

[
  {
    "option_exchange": "PSE",
    "expirations_asc": [
      "2020-10-16",
      ...,
      "2021-03-19"
    ],
    "strikes_asc": [
      5.0,
      ...,
      41.0
    ]
  },
  {
    "option_exchange": "BATS",
    "expirations_asc": [
      "2020-10-16",
      ...,
      "2021-03-19"
    ],
    "strikes_asc": [
      5.0,
      ...,
      41.0
    ]
  },
  ...
]
```

Example, **get option contracts**:

```
http://localhost:8001/api/v1/stock_option_chain_contracts?symbol=SIVR&exchange=AMEX&currency=USD

[
  {
    "option_exchange": "PSE",
    "expirations_asc": [
      "2020-10-16",
      ...,
      "2021-03-19"
    ],
    "strikes_asc": [
      5.0,
      ...,
      41.0
    ]
  },
  {
    "option_exchange": "BATS",
    "expirations_asc": [
      "2020-10-16",
      ...,
      "2021-03-19"
    ],
    "strikes_asc": [
      5.0,
      ...,
      41.0
    ]
  },
  ...
]
```

Example, **get option contract price**:

```
http://localhost:8001/api/v1/stock_option_price?symbol=SIVR&right=call&expiration=2021-03-19&strike=20.0
&exchange=AMEX&currency=USD

{
  "price": 5.0,
  "datatype": "realtime",
  "price_type": "close"
}
```

Example, **batch requests**:

```
http post http://localhost:8001/call [
  ["/api/v1/stock_price", { "symbol"="MCD", "exchange"="NYSE", "currency"="USD" }],
  ["/api/v1/stock_price", { "symbol"="WRONG", "exchange"="NYSE", "currency"="USD" }],
]

[
  {
    "price": 200.0,
    "datatype": "realtime",
    "price_type": "close"
  },
  {
    "is_error": true,
    "message": "Error code 200 No security definition has been found for the request"
  }
]
```

# How to use

1 Download this IB API repository.

2 Download the [Official TWS API Java Driver](https://interactivebrokers.github.io/tws-api). I believe
its license is not allowing to include it in other products, so you have to download it separately.

3 Copy Java Sources from Official TWS API, the `com/ib` folder to the `/src` folder in this repository.
So you should see the `/src/com/ib` folder in this repository to be present and contain
Java Sources from Official TWS API.

4 Compile and run it with Maven (you need to have Java and Maven installed) with `bin/crun`.

5 Ensure your TWS (Trader Workstation) is running, and the API access setting enabled and use same port as the port printed
by this driver when it's started (it could be changed in config if you want different port).

6 Open Browser and check price of MSFT to see if it's working
`http://localhost:8001/api/v1/stock_price?exchange=SMART&currency=USD&symbol=MSFT`

7 Look at [IB.kt](https://github.com/alexeypetrushin/ib_api/blob/master/src/ib/ib.kt) file to see the API docs.

8 Change config if needed. Any default config variable could be overiden by shell variable with
the same name, for list of all the configuration options
see [Config.kt](https://github.com/alexeypetrushin/ib_api/blob/master/src/ib/config.kt) file.

# Client Libraries

- Nim - [nim/ib.nim](/nim/ib.nim)
- Other languages - look at code in [nim/ib.nim](/nim/ib.nim) and translate it
  to language you need, it's short and simple.

# Features

- Simplicity vs universality and completeness.
- Strict and correct, any unexpected thing causes explicit error.
- No distraction, run it in background once and forget about it.
- Robust, requests retried in case of error and terminated by timeout if take too long.

# Contributing

Packages - `ib` - public stuff, `ib.lib` - internal stuff related to IB, `ib.lib.support` - internal stuff
  not related to IB.

`IB` - the API, that's the entry point, check this as the first step.

`IBImpl` - API imlementation.

`Worker` - pool of workers, handles requests. Each worker has `EWrapperImpl` instance to communicate with TWS.
`IB`, `Worker` and `EWrapperImpl` communicate through loosely coupled message queues. There's one message queue
for each worker `Worker <-> EWrapperImpl` and `IbQueue` to communicate `Worker <-> IbImpl`.

`Converter` - validates and parses TWS data and converts it to sane format.

`Config` - configuration options, some could be set from the shell.

# Notes

- I use unusual code conventions for Kotlin, just for fun, wanted to try it and see if it works or not.
- Internal IB Asset types
  https://www.interactivebrokers.com/en/software/et_dataguide/topics/asset_types.htm
- IB contracts
  https://interactivebrokers.github.io/tws-api/classIBApi_1_1Contract.html
- Some docs https://dimon.ca/dmitrys-tws-api-faq

# License

Free for personal usage, universities and non-profit organizations.

The software is provided "as is", without warranty of any kind, express or
implied, including but not limited to the warranties of merchantability,
fitness for a particular purpose and noninfringement. In no event shall the
authors or copyright holders be liable for any claim, damages or other
liability, whether in an action of contract, tort or otherwise, arising from,
out of or in connection with the software or the use or other dealings in the
software.