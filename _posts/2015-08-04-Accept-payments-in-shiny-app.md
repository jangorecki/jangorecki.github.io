---
layout: post
title: Accept payments in shiny app
tags: R shiny BTC
---



Have you ever think about accepting payments in your shiny app?  
Probably not, but now you can start ;)  
Shiny apps are usually single task, not very heavy websites. It may be not so easy to turn them into online shop/service provider.  
Anyway you can find this post interesting as it presents a paperwork-less implementation to accept payments.  

Some notable features of presented setup:  

- self-hosted
- license-free
- not relying on ANY third party services (banks, brokers, payment processors, exchange markets)
- no new accounts, registrations, verifications
- no new accounts, registrations, verifications also for your customer
- low fee

# Example use case on selling content

You are publishing article, an expertise on particular topic. The part of your article is publicly available and acts as introduction and general overview on the subject. The other part of article is detailed analysis, that part you want to sell to a viewer for a single dollar per view (1 USD). So what you need is to conditionally display the second part of article based on the received payment condition.  

Obviously such payments can be employed to sell goods or services, but then it usually better to add user account functionality to track delivery process.  

# Infrastructure

Shiny app connects to bitcoin node directly using [httr](https://github.com/hadley/httr) and [jsonlite](https://github.com/jeroenooms/jsonlite) packages. In the process two JSON-RPC methods are employed:

- `getnewaddress` - generate new payment address for the current session
- `getreceivedbyaddress` - refresh the balance to check if payment received

The workhorse function to handle JSON-RPC looks like:

```r
bitcoind.rpc <- function(host = getOption("rpchost","127.0.0.1"),
                         user = getOption("rpcuser"),
                         password = getOption("rpcpassword"),
                         port = getOption("rpcport","8332"),
                         id = NA_integer_, method, params = list()){
    stopifnot(is.character(user), is.character(password), is.character(method))
    rpcurl <- paste0("http://",user,":",password,"@",host,":",port)
    req <- httr::POST(rpcurl, body = jsonlite::toJSON(list(jsonrpc = "1.0", id = id, method = method, params = params), auto_unbox=TRUE))
    if(httr::http_status(req)$category != "success"){
        message(jsonlite::fromJSON(httr::content(req, "text"))$error$message)
        httr::stop_for_status(req)
    }
    jsonlite::fromJSON(httr::content(req, "text"))
}
```

Shiny app will additionally:

- calculate current fiat currencies value based on rates from [kraken exchange](https://www.kraken.com/) ticker as `(ask+bid)/2`
- produce clickable payment link
- if [hrbrmstr/qrencoder](https://github.com/hrbrmstr/qrencoder) package is installed, it will also encode payment link into QR code so payment can be easily done from smartphone wallet

# Setup

- install bitcoin node, [Download Bitcoin Core](https://bitcoin.org/en/download), see also [Running A Full Node](https://bitcoin.org/en/full-node)
- allow RPC against your bitcoin daemon
- install [R](https://www.r-project.org/) and [shiny](https://github.com/rstudio/shiny) package, for QR code also install [hrbrmstr/qrencoder](https://github.com/hrbrmstr/qrencoder) package

To run presented shinyPay app you can fork [jangorecki/shinyPay](https://github.com/jangorecki/shinyPay) and replace below bitcoin node details in `app.R` file:

```r
options(rpchost = "192.168.56.103",
        rpcuser = "bitcoinduser",
        rpcpassword = "userpassbitcoind")
```

If you would setup bitcoin daemon on the same ip/credentials you can run the app by simply:  

```r
shiny::runGitHub("jangorecki/shinyPay")
```

# Cold demo

The post does not contain reproducible code as it requires bitcoin node. Below a screen from the shinyPay app and short description.

![shinyapp](https://cloud.githubusercontent.com/assets/3627377/9061873/97e64a1e-3ab6-11e5-8699-22af044ad3fd.png)

To request payment I simply click on button to generate an address for my payment.  
I choose the amount of 0.0035 BTC (~1 USD), the QR code gets refreshed.  
I've scanned the QR code with on Android (2.3.5) phone using [this wallet app](https://play.google.com/store/apps/details?id=de.schildbach.wallet&hl=en_GB).  
My android wallet made payment over HSDPA network on different ISP than the ISP which my bitcoin node uses (30 Mbps).  
The payment arrived after 2-3 seconds. It gets first confirmation after 2 minutes (on avarage first *conf* takes around 10 minutes).  
Total transaction fee was 0.0001 BTC (~0.028 USD). In this particular case it was 2.8% of the payment, but the transaction fee is constant no matter how big transaction is. For a 100 dollar payment the 2 cents fee would be unnoticeable (0.028%).  
Mentioned transaction can be [found in blockchain](https://blockchain.info/tx/49fe8b430bb467ac7bdfb72c42598bde4b71cf1cd54b93957abefab7fdb99f66).  

# External bitcoin node service

If you are not comfortable with hosting own bitcoin node you can use external bitcoin node provider.  

The most popular would be [blockchain.info](https://blockchain.info), a less known alternative [blockcypher.com](http://blockcypher.com). There may be some others I'm not aware of.  

For non-merchant users who are already familiar with blockchain.info I will add an important info: unlike when using blockchain.info over www where you are not sharing priv key to server, when using blockchain.info over rpc you have to share priv key to server in order to generate new addresses.  

> Unlike when using the javascript wallet transaction signing is conducted server side which means your private keys are shared with the server. However many operations such as fetching a balance and viewing transactions are possible without needing to decrypt the private keys. The is possible through the use of double encryption only when the second password is provided does the server have ability to access the funds in a wallet. For merchants who need to only receive transactions it maybe possible to never provide a second password.

source: [blockchain.info/api/json_rpc_api](https://blockchain.info/api/json_rpc_api)

# Receive payments in fiat currencies

If you would like to automatically convert collected funds into fiat currencies you can use `sendtoaddress` rpc method to transfer your funds to your deposit address on a bitcoin exchange market, and there automatically buy fiat currency using market api (see [Rbitcoin](https://github.com/jangorecki/Rbitcoin)). This obviously requires to create an account on the market and usually go through formal KYC/AML verification process.  

# Security

There are numerous ways to increase the security of your wallet, listing just some of them:  

- validate sold content/service/product value against received funds
- increase confirmations required for payment to be accepted
- increase confirmations for low mining-priority transactions
- run multiple nodes to verify against double-spend
- automatically send confirmed payments to offline wallet
- restrict rpc methods
- restrict access to particular ip
- use json-rpc over ssl
- isolation of bitcoin node machine
- ssh tunnel

You should not try to implement all possible ways but rather weight the risk and scale your security to the volume/value of received transactions.  

Anyway I'm not an expert in security, for more info on that you can look at [Securing your wallet](https://bitcoin.org/en/secure-your-wallet), [bitcoin wiki](https://en.bitcoin.it/wiki/Main_Page) or [bitcoin stackexchange](http://bitcoin.stackexchange.com).  

If you are interested in getting comprehensive knowledge about the *money of the internet* you should look at the free online full semester course (10 ECTS) at University of Nicosia: [Introduction to Digital Currencies](http://digitalcurrency.unic.ac.cy/free-introductory-mooc/course-overview).
