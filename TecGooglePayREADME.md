# TEConnect Google Pay
This document demonstrates the available options, using Google Pay via TEConnect.  TEConnect currently offers a Manual Entry form, 3DS Manual Entry, Apple Pay and Google Pay.  Your app may use one - or all of these platforms to collect a payment token.  The section below explains how opt-in works with TEConnect for Google Pay specifically.

# Payment Request Opt-In
```createTEConnect``` accepts a public key as the first argument. This public key is for [TEConnect Manual Entry](README.md#Getting-Started).
The second argument is the [TEConnect options](README.md#TEConnect-Options) object.  
Providing an ```appleMerchantId``` or a ```googleMerchantId``` to the ```tecPaymentRequest``` object is how to opt-in for each Payment Request Platform.  So providing a `googleMerchantId` for Google Pay.  Example below.

```javascript
const TE_CONNECT: TEConnect = createTEConnect("__publicKeyGoesHere__", {
    tecPaymentRequest: {
        googleMerchantId: "__googleMerchantId__"
    }
});
```

Opting in to one or more platform affects the way the Payment Request Object should be supplied to TEConnect - as well as the  [```CanMakePaymentsResult```](TecPaymentRequestREADME.md#CanMakePaymentsResult).

Be aware that if you only opt-in for one platform (i.e. only an ```googleMerchantId``` is provided to ```createTEConnect```) - you may provide the payment request object for that specific platform.  In this case - the [Google Pay](#Google-Pay-Payment-Request-Object) can be provided as-is.  

In the case of multiple payment request platforms (i.e. both an ```appleMerchantId``` and ```googleMerchantId``` are provided) - the payment request object must be structured to reflect each platform.  Example below.

```javascript
const paymentRequestObject = {
    applePay: applePayRequestObject,
    googlePay: googlePayRequestObject
}
```

## Important Google Pay Notes
The Google Pay workflow differs from Apple Pay in a few ways.  

The [`confirm-token` Event](./TecPaymentRequestREADME.md#confirm-token-Event) is only mandatory if you do not provide the optional `callbackIntents` in your [payment request object](#Google-Pay-Payment-Request-Object).  If you do choose to leverage an `onPaymentAuthorized` callback - the token will be presented in that callback function, prior to the [`confirm-token` Event](./TecPaymentRequestREADME.md#confirm-token-Event).  
  
By registering an `onPaymentAuthorized` callback - the Google Pay form will wait for a positive or negative response from your application before closing (waiting for your application's processing response). This callback is optional - and the default behavior is the form closes upon token generation ([`confirm-token` Event](./TecPaymentRequestREADME.md#confirm-token-Event)).  The [standard implementation code example](./TecPaymentRequestREADME.md#Example-Implementation) uses the [`confirm-token` Event](./TecPaymentRequestREADME.md#confirm-token-Event), while the [more complex implementation](#Google-Pay-Example-Implementation) uses the `onPaymentAuthorized` callback to capture the token.

Any optional callback or options will be provided directly in the [payment request object](#Google-Pay-Payment-Request-Object).
  
Google Pay requires three points of identification:  `merchantId`, `merchantName`, and `gatewayId`. The `merchantId` and `merchantName` will be negotiated directly between you and Google.  [Google's documentation on MerchantInfo and their Wallet Console](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo) is very detailed - just remember that the `merchantId` is fed to the `createTEConnect` function, while the other points of information are provided in the [payment request object](#Google-Pay-Payment-Request-Object).  The `gatewayId` is provided by Magensa™ after a successful account creation.  

<br/>


# Google Pay Payment Request Object
This section lists all available properties that are available for use with TEConnect.
Many of the properties correspond directly to Google's documentation on the payment request object ([found here](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataRequest)).  Some of the properties listed in Google's documentation (such as ```tokenizationSpecification```) are built by TEConnect, and as such is not read as an input.

An Example for a simple implementation can be [found here](https://github.com/Magensa/te-connect-js/blob/master/TecPaymentRequestREADME.md#Example-Implementation).
An example for the most complex object allowed can be [found below](#Google-Pay-Example-Implementation).

For more information on each property - please see [Google's Documentation](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataRequest).


| Property Name | Input Type | Required | Default Value (if optional) | Notes |
|:--:|:--:|:--:|:--:|:--:|
| apiVersion | ```number``` | :x: | 2 | Optional field to set the major version of Google Pay API |
| apiVersionMinor | ```number``` | :x: | 0 | Optional field to set the minor version of Google Pay API |
| allowedAuthMethods | ```string[]``` | :x: | ```["PAN_ONLY", "CRYPTOGRAM_3DS"]``` | [More details on ```allowedAuthMethods``` here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| allowedCardNetworks | ```string[]``` | :heavy_check_mark: | N/A  | [More details on ```allowedCardNetworks``` here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| merchantName | ```string``` | :heavy_check_mark: | N/A | ```merchantId``` is provided to the ```createTeConnect``` call. ```merchantName``` is provided in the payment request object. [More info here](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo) |
| gatewayId | ```string``` | :heavy_check_mark: | N/A  | Given by Magensa™ after a successful account creation |
| transactionInfo | [```TransactionInfo```](https://developers.google.com/pay/api/web/reference/request-objects#TransactionInfo) | :heavy_check_mark: |  | [More details on all available properties here](https://developers.google.com/pay/api/web/reference/request-objects#TransactionInfo) |
| callbackIntents | ```string[]``` | :x: | N/A | Declares intents for [paymentDataCallbacks](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataCallbacks) |
| paymentDataCallbacks | ```object``` | :x: | N/A | [Callback functions](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataCallbacks) for dynamic price updates, and user-facing messages via Google Pay form |
| offerInfo | [```OfferInfo```](https://developers.google.com/pay/api/web/reference/request-objects#OfferInfo) | :x: | N/A | [OfferInfo](https://developers.google.com/pay/api/web/reference/request-objects#OfferInfo) is displayed when the payment sheet loads |
| emailRequired | ```boolean``` | :x: | N/A | set ```true``` to request an email address  |
| shippingAddressRequired | ```boolean``` | :x: | N/A | set ```true``` to request a full shipping address  |
| shippingOptionRequired | ```boolean``` | :x: | N/A | Set to true when the ```SHIPPING_OPTION``` callback intent is used. This field is required if you implement support for Authorize Payments or Dynamic Price Updates. [More info on ShippingOptionParameters](https://developers.google.com/pay/api/web/reference/request-objects#ShippingOptionParameters) |
| shippingAddressParameters | ```object``` | :x: | N/A | If ```shippingAddressRequired``` is set to true, specify shipping address restrictions | 
| shippingOptionParameters | [```ShippingOptionParameters```](https://developers.google.com/pay/api/web/reference/request-objects#ShippingOptionParameters) | :x: | N/A | Set default options. [More details on ```ShippingOptionParameters``` here](https://developers.google.com/pay/api/web/reference/request-objects#ShippingOptionParameters) | 
| allowedPaymentMethods | [```PaymentMethod[]```](https://developers.google.com/pay/api/web/reference/request-objects#PaymentMethod) | :x: | N/A | TEConnect will build the default ["CARD" Payment Method](https://developers.google.com/pay/api/web/reference/request-objects#PaymentMethod). This property allows for optionally adding more. Be aware that every [```PaymentMethod```](https://developers.google.com/pay/api/web/reference/request-objects#PaymentMethod) supplied will receive Magensa's [```tokenizationSpecification```](https://developers.google.com/pay/api/web/reference/request-objects#PaymentMethodTokenizationSpecification). Overrides for [```tokenizationSpecification```](https://developers.google.com/pay/api/web/reference/request-objects#PaymentMethodTokenizationSpecification) is not supported at this time. It is unlikely this property will be utilizied, but is available for future iterations.| 
| environment | ```string``` | :x: | ```"TEST"``` | ```"TEST"``` or ```"PRODUCTION"```. [More details on `PaymentOptions.environment` here](https://developers.google.com/pay/api/web/reference/request-objects#PaymentOptions) |
| existingPaymentMethodRequired | `boolean` | :x: | N/A | Setting this to `true` modifies the `CanMakePaymentsResult` [More details here](https://developers.google.com/pay/api/web/reference/request-objects#IsReadyToPayRequest) |
| prefetchPaymentData |  ```boolean``` | :x: | N/A | If set to ```true```, [```prefetchPaymentData```](https://developers.google.com/pay/api/web/reference/client#prefetchPaymentData) will be called upon button mount. [More info on ```prefetchPaymentData``` here](https://developers.google.com/pay/api/web/reference/client#prefetchPaymentData) |  
| allowPrepaidCards | `boolean` | :x: | true | [More details here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| allowCreditCards | `boolean` | :x: | true | [More details here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| assuranceDetailsRequired | `boolean` | :x: | N/A | Set to `true` to request `assuranceDetails` [More details here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| billingAddressRequired | `boolean` | :x: | false | Set to `true` if you require a billing address [More details here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |
| billingAddressParameters | `object` | :x: | N/A | The expected fields returned if billingAddressRequired is set to `true`. [More details here](https://developers.google.com/pay/api/web/reference/request-objects#CardParameters) |



# Google Pay Button Options
Google Pay offers many button options to tailor the Google Pay Button to the applications theme. [Here are Google's Brand Guidelines](https://developers.google.com/pay/api/web/guides/brand-guidelines) that explain how the button may be styled appropriately.
[Here we have Google's list of ```ButtonOptions```](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions).  These button options are available with a few restrictions (two properties managed by TEConnect).

TEConnect is responsible for creating and mounting the button. To that end - TEConnect will handle the ```allowedPaymentMethods``` property, which is based upon the payment request object supplied to TEConnect, in addition to any additional payment methods supplied in the optional property ```allowedPaymentMethods```.  It's unlikely any additional ```allowedPaymentMethods``` need be supplied (Google Pay currently only supports the ```"CARD"``` payment method which is built based upon the payment request object supplied to TEConect), however adding additional payment methods is supported if desired.

TEConnect is also responsible for the button click handler (```onClick```).  If any action is needed before the payment form is rendered - make use of the ```preClick``` property in the TEConnect options to provide your own function.
The ```preClick``` function has its limitations, and is only intended for application triggers prior to the Google Pay button click:
- ```preClick``` functions will be called synchronously.  The function is called before returning the async ```onClick``` function.
- Any return value from the ```preClick``` function will not be returned to the caller.  

Below is a table of available properties, and after that are examples.  Feed the options object to the ```TecPaymentRequestButtons``` component.

| Property Name | Input Type | Notes |
|:--:|:--:|:--:|
| preClick | ```function``` | will be called syncronously. Return value will not be returned to caller. |
| buttonColor | ```string``` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |
| buttonType | ```string``` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |
| buttonLocale | ```string``` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |
| buttonSizeMode | ```string``` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |
| buttonRadius | `number` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |
| buttonRootNode | ```HTMLDocument or ShadowRoot``` | See Google's [ButtonOptions](https://developers.google.com/pay/api/web/reference/request-objects#ButtonOptions) |

```javascript
const exampleGoogleButtonOptions = {
    preClick: () => console.log('preClick executed'),
    buttonColor: 'black',
    buttonType: 'order',
    buttonLocale: 'en', //If not supplied - defaults to browser or OS language settings
    buttonSizeMode: 'static'
}

const buttonOptions = {
    googlePayOptions: exampleGoogleButtonOptions
}

import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';


function demoInit() {
    const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { googleMerchantId: "__tecGoogleMerchantId__" } 
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(googlePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div", buttonOptions);
    });

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //Unsuccessful - check message.
        }
        else {
            //Successful - pass `tokenDetails.token` to MPPG to process payment
        }
    });
}

demoInit();
```

<br />

# Google Pay Example Implementation
The below implementation utilizes most of the Google Pay features, for a more complex example. Your application's specific implementation will differ according to your needs.  

Google also provides [several implementation examples](https://developers.google.com/pay/api/web/guides/tutorial#full-example) as well - just be aware that TEConnect handles most of the interactions between the Google Pay API and your application - meaning that most all of your logic will be provided in the [payment request object](#Google-Pay-Payment-Request-Object) (as opposed to the various sections provided in Google's examples).

```index.html```
```html
<body>
    <h1>Token Exchange Connect Payment Request</h1>
    <div id="example-div"></div>
    <script src='main.bundle.js'></script>
</body>
```
```javascript
//======= Build Payment Request Object =======

function getGoogleTransactionInfo() {
	return {
        displayItems: [{
			label: "Subtotal",
			type: "SUBTOTAL",
			price: "11.00",
        },
		{
            label: "Tax",
            type: "TAX",
            price: "1.00",
        }],
        countryCode: 'US',
        currencyCode: "USD",
        totalPriceStatus: "FINAL",
        totalPrice: "12.00",
        totalPriceLabel: "Total"
	};
}

function processPayment(paymentData) {
	return new Promise(function(resolve, reject) {
        console.log(paymentData);
        // When using an 'onPaymentAuthorized' callback - pass the property below to MPPG for payment processing.
        console.log(paymentData.paymentMethodData.tokenizationData.token);
        //If processing is successful - resolve - otherwise reject
        resolve({});
    });
}

function onPaymentAuthorized(paymentData) {
    // When using an 'onPaymentAuthorized' callback - you don't need to 'listenFor' 'confirm-token'.
	return new Promise(function(resolve, reject) {
        processPayment(paymentData)
        .then(function() {
            resolve({transactionState: 'SUCCESS'});
        })
        .catch(function() {
            resolve({
                transactionState: 'ERROR',
                error: {
                    intent: 'PAYMENT_AUTHORIZATION',
                    message: 'Insufficient funds',
                    reason: 'PAYMENT_DATA_INVALID'
                }
            });
        });
    });
}

function getGoogleUnserviceableAddressError() {
	return {
        reason: "SHIPPING_ADDRESS_UNSERVICEABLE",
        message: "Cannot ship to the selected address",
        intent: "SHIPPING_ADDRESS"
	};
}

function getGoogleDefaultShippingOptions() {
	return {
        defaultSelectedOptionId: "shipping-001",
        shippingOptions: [{
            "id": "shipping-001",
            "label": "Free: Standard shipping",
            "description": "Free Shipping delivered in 5 business days."
        },
        {
            "id": "shipping-002",
            "label": "$1.99: Standard shipping",
            "description": "Standard shipping delivered in 3 business days."
        },
        {
            "id": "shipping-003",
            "label": "$10: Express shipping",
            "description": "Express shipping delivered in 1 business day."
        }]
    };
}

function getShippingCosts() {
	return {
        "shipping-001": "0.00",
        "shipping-002": "1.99",
        "shipping-003": "10.00"
    }
}

function calculateNewTransactionInfo(shippingOptionId) {
	let newTransactionInfo = getGoogleTransactionInfo();

    let shippingCost = getShippingCosts()[shippingOptionId];
    newTransactionInfo.displayItems.push({
        type: "LINE_ITEM",
        label: "Shipping cost",
        price: shippingCost,
        status: "FINAL"
    });

    let totalPrice = 0.00;
    newTransactionInfo.displayItems.forEach(displayItem => totalPrice += parseFloat(displayItem.price));
    newTransactionInfo.totalPrice = totalPrice.toString();

    return newTransactionInfo;
}

function onPaymentDataChanged(intermediatePaymentData) {
	return new Promise(function(resolve, reject) {
		let shippingAddress = intermediatePaymentData.shippingAddress;
		let shippingOptionData = intermediatePaymentData.shippingOptionData;
		let paymentDataRequestUpdate = {};

		if (intermediatePaymentData.callbackTrigger === "INITIALIZE" || intermediatePaymentData.callbackTrigger === "SHIPPING_ADDRESS") {
			if(shippingAddress.administrativeArea === "NJ")  {
				paymentDataRequestUpdate.error = getGoogleUnserviceableAddressError();
			}
			else {
				paymentDataRequestUpdate.newShippingOptionParameters = getGoogleDefaultShippingOptions();
				let selectedShippingOptionId = paymentDataRequestUpdate.newShippingOptionParameters.defaultSelectedOptionId;
				paymentDataRequestUpdate.newTransactionInfo = calculateNewTransactionInfo(selectedShippingOptionId);
			}
		}
		else if (intermediatePaymentData.callbackTrigger === "SHIPPING_OPTION") {
		    paymentDataRequestUpdate.newTransactionInfo = calculateNewTransactionInfo(shippingOptionData.id);
		}

		resolve(paymentDataRequestUpdate);
	});
}

const googlePaymentRequestObject = {
	apiVersion: 2,
	apiVersionMinor: 0,
	allowedCardNetworks: ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "MIR", "VISA"],
	merchantName: "TEConnect Test",
	gatewayId: "__magensa_gateway_id__",
	transactionInfo: {
        displayItems: [{
			label: "Subtotal",
			type: "SUBTOTAL",
			price: "11.00",
        },
      	{
			label: "Tax",
			type: "TAX",
			price: "1.00",
		}],	
		countryCode: 'US',
		currencyCode: "USD",
		totalPriceStatus: "FINAL",
		totalPrice: "12.00",
		totalPriceLabel: "Total"
  	},
	callbackIntents: ["SHIPPING_ADDRESS",  "SHIPPING_OPTION" , "PAYMENT_AUTHORIZATION"],
	emailRequired: true,
	shippingAddressRequired: true,
	shippingOptionRequired: true,
	shippingAddressParameters: {
		allowedCountryCodes: ['US'],
		phoneNumberRequired: true
	},
	shippingOptionParameters: getGoogleDefaultShippingOptions(),
	paymentDataCallbacks: {
		onPaymentAuthorized: onPaymentAuthorized,
		onPaymentDataChanged: onPaymentDataChanged
	},
	environment: "TEST",
    allowCreditCards: true,
    assuranceDetailsRequired: false,
    billingAddressRequired: true,
    billingAddressParameters: {
        format: "FULL",
        phoneNumberRequired: false
    }
};

const customGoogleButtonOptions = {
    googlePayOptions: {
        preClick: () => console.log('preClick executed just before payment sheet loads'),
        buttonColor: 'white',
        buttonType: 'order',
        buttonLocale: 'ja',
        buttonSizeMode: 'fill'
    }
}

//======= JS Implementation =======

import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';


function demoInit() {
    const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { googleMerchantId: "__tecGoogleMerchantId__" } 
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(googlePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div", customGoogleButtonOptions);
    });

    /*
    Note that because we have registered an `onPaymentAuthorized` callback in the payment request object, the listener below becomes optional for Google Pay:

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //Unsuccessful - check message.
        }
        else {
            //Successful - pass `tokenDetails.token` to MPPG to process payment
        }
    });
    */
}

demoInit();
```


# Google Pay Error Handling
Google offers a few methods to display errors to users via the Google Pay form.  Both those options exist within the optional callbacks supplied to the ```paymentDataCallbacks``` property of the [paymentRequestObject](#Google-Pay-Payment-Request-Object).
- [Here is Google's documentation about the ```onPaymentAuthorized``` callback](https://developers.google.com/pay/api/web/reference/client#onPaymentAuthorized)
- [Here is Google's documentation about the ```onPaymentDataChanged``` callback](https://developers.google.com/pay/api/web/reference/client#onPaymentDataChanged)
- [Here is the section on how to build Google Pay Error objects](https://developers.google.com/pay/api/web/reference/response-objects#PaymentDataError)
- [Here is the section on Errors in general - including those you may encounter while building your Google Pay integration](https://developers.google.com/pay/api/web/reference/error-objects)

