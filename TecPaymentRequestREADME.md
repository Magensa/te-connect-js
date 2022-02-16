# TEConnect Payment Request (React)
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-js.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-js "@magensa/te-connect-js npm.js")  
JavaScript module for use with Apple Pay via Token Exchange Connect

# Getting Started
```
npm install @magensa/te-connect @magensa/te-connect-js
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-js
```
or via CDN:
```html
    <script src="https://cdn.magensa.net/te-connect/1.1.1/te-connect.js"></script>
    <script src="https://cdn.magensa.net/te-connect-js/1.1.1/te-connect-js.js"></script>
```
If you would prefer to let the code speak, below we have three [example implementations](#Example-Implementation). One is minimally viable, while the other utilizes all available features.

# Step-by-step
1. The first step is to create a TEConnect instance, and feed that instance to the TeConnectJs (from ```@magensa/te-connect-js```) constructor.
    - The first parameter is your public key for [TEConnect manual entry](https://github.com/Magensa/te-connect-js) (```CardEntry``` component).
    - The second parameter is the [```options``` object](https://github.com/Magensa/te-connect-js#teconnect-options)
        - For the ```tecPaymentRequest``` options - supply your ```appleMerchantId``` to enable Apple Pay.
            - ```appleMerchantId``` is obtained after the on-boarding process with Magensa™ is completed.  
            
```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);
}

demoInit();
```  

2. Build a [```paymentRequestObject```](#Payment-Request-Object) to define the Payment Request Form.
    - Once you have built your [payment request object](#Payment-Request-Object). Supply that object to the  [```createPaymentRequestInterface```](#Create-Payment-Request-Interface) function.
        - This function will return the [TecPaymentRequestsInterface](#Token-Exchange-Connect-Payment-Request-Interface) (referred to as ```tecPrInterface```, in this document).
    - Call the [```canMakePayments```](#canMakePayments) function, and await the result to determine if the user is capable of using Payment Requests (Apple Pay).  

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            //Positive CanMakePayments result
    });
}

demoInit();
```  

3. Next, design your page according to your specifications - and create a ```<div>``` with a unique id. Once the interface has been instantiated, and the [```CanMakePaymentsResult```](#CanMakePaymentsResult) yeilds a positive result - use the ```tecPrInterface``` to mount the Payment Request Button on that div.  

```index.html```
```html
<body>
    <div id="example-div"></div>
    <script src='main.bundle.js'></script>
</body>
```

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div");
    });
}

demoInit();
```  

3. Finally - when a user submits the payment request form - an event is emitted. Listen to the [```confirm-token```](#confirm-token-Event) event to inspect the result (as demonstrated below).
    - The ```completePayment``` function is a property of the ```confirm-token``` event. This function must be called within 30 seconds of the event firing - otherwise the form will timeout.

```index.html```
```html
<body>
    <div id="example-div"></div>
    <script src='main.bundle.js'></script>
</body>
```

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div");
    });

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //Unsuccessful - check message.
            completePayment('failure');
        }
        else {
            //Successful - close user's payment request form with payment success notification.
            completePayment('success');
        }
    });
}

demoInit();
```  

The above is a minimally viable solution to get you started. There are [two more code examples](#example-implementation) - a minimally viable example, as well as a more complex example to leverage more of the features available. Read on for further features and details availble to the Payment Request Utility.  
<br />

# Payment Request Object
The Payment Request Object is the only parameter for the [```createPaymentRequestInterface```](#Create-Payment-Request-Interface) function. This object describes the form that the end-user will interact with.
While all payment requests will require a payment request object to define your form - currently TEConnect only supports Apple Pay as a Payment Request Provider.  To that end - this section will specifically address the [ApplePayPaymentRequest object](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest).
- [Here is Apple's Documentation about the payment request object](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest)
- It is encouraged to read through Apple's documentation for all the options available to you to define your form.
- Be aware that any inaccuracy in the payment request object will most likely throw a ```TypeError``` without a message. Any errors that come from Apple's javascript rarely have any messages attached to errors.

Below is an example of a Payment Request object.  This is where your form is defined - so each customer's object will appear differently, depending on the customer's circumstances.
```javascript
const examplePaymentRequestObject = {
    storeDisplayName: "TEConnect Example Store",
    currencyCode: "USD",
    countryCode: "US",
    supportedNetworks: ['visa', 'masterCard', 'amex', 'discover', 'jcb'],
    merchantCapabilities: ['supports3DS'],
    total: {
        label: "Test Transaction",
        amount: "1.00",
        type: "final"
    },
    shippingType: ['shipping'],
    shippingMethods: [
        {    
            label: 'Free Shipping',
            detail: "Arrives in a week",
            amount: '0.00',
            identifier: "FreeShipping"
        },
        {    
            label: 'Not Free Shipping',
            detail: "Arrives in less than week",
            amount: '10.00',
            identifier: "ChargeShipping"
        }
    ],
    requiredShippingContactFields: [
        'name',
        'email',
        'phone',
        'postalAddress'
    ],
    requiredBillingContactFields: [
        'name',
        'postalAddress'
    ],
    shippingContact: {
        phoneNumber: '888-8888',
        emailAddress: 'something@example.net',
        givenName: 'Bob',
        familyName: 'Kahuna',
        addressLines: ['123 Main St.', 'Suite 101'],
        locality: 'Los Angeles',
        administrativeArea: 'CA',
        postalCode: '90010',
        country: 'United States',
        countryCode: 'US',
    },
    billingContact: {
        emailAddress: 'something@example.net',
        givenName: 'Bob',
        familyName: 'Kahuna',
        addressLines: ['123 Main St.', 'Suite 101'],
        subLocality: 'Los Angeles',
        locality: 'CA',
        postalCode: '90010',
        country: 'United States',
        countryCode: 'US',
    }
};
```

# Token Exchange Connect Payment Request Interface
The Token Exchange Connect Payment Request Interface (referred to as the ```tecPrInterface```) is the interface needed to interact, and respond to user's interactions on the Payment Request Form.

## Create Payment Request Interface
The ```createTecPaymentRequest``` function is property of the ```TeConnectJs``` class, after it's been instanted. Feed this function a [payment request object](#Payment-Request-Object) and it returns the ```tecPrInterface```.

## Interface Methods
### ```canMakePayments```
This function returns a Promise, that resolves to a [```CanMakePaymentsResult```](#CanMakePaymentsResult)

```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult) {
            //User has ability to pay using a Payment Request
        }
    });
}

demoInit();
```  

#### ```CanMakePaymentsResult```
```typescript
type CanMakePaymentsResult = {
    applePay: boolean
} | null;
```

### ```updatePaymentRequest```
This function is only useful _before_ the user has hit the Apple Pay button.  If you have [created a payment request interface](#Create-Payment-Request-Interface), using a [payment request object](#Payment-Request-Object) - but wish to update that object before the user hits the button and renders the payment request form - you may call this function to do so.  Be aware that this function is a __full update__, so the object will completely replace the previous payment request object supplied.
- It's not necessary to update your object inside of [payment request listeners](#Payment-Request-Event-Handlers). Any response in your completion functions will update the form dynamically.


### ```listenFor```
This function accepts an event name (```string```) to listen for, as well as a listener function. These events are specific to payment request interface instance created. Please [see the section below](#Payment-Request-Event-Handlers) for more details.


# Payment Request Event Handlers
When users interact with the payment request form - there are several events that are fired. Listening to the ```confirm-token``` event is required to complete the workflow, but there are more events that may be subscribed to - optionally.  At this time, Apple Pay is the only payment request platform available via TEConnect. To that end, we will focus on Apple specific events in this document.  
  
## Apple Pay Listeners
When [qualified Apple Pay users](#User-Requirements) interact with the Apple Pay form, there are several events that may be subscribed to. Each event will fire with an object (describing the event) and a function (to respond to the event). Be aware that if an event is subscribed to - the response function _must_ be called with a response within 30 seconds - otherwise the form will timeout. The only exception to this is the ```cancel-transaction``` event - which only contains a message.  
Examples of all the listeners can be found in the more complex of the [Example Implementations](#Example-Implementation)

### ```confirm-token``` Event
This is the only event listener that is required to complete the Apple Pay workflow. This is the only event that can optionally contain an ```error``` property (in the case the payment was submitted, but was unsuccessful). Be sure to check for that property first, if it exists.  
When listening to the ```confirm-token``` event - there will be two special properties to the event:
- ```tokenDetails```
    - ```type``` specifies the payment request platform in which the token was created.
        - Currently any successful token creation will be of type ```applePay```.
    - ```token``` will be the object needed to process transactions, using the payment token created during the current session.
    - ```error``` is an optional property in the case the token creation was unsuccessful.
- ```completePayment```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the payment form.
    - There are two possible value types to call this function with - either a string value, or an object:
        - This function expects either ```"sucess"``` or ```"failure"``` as string options - and will display the chosen completion status on the form.
        - Optionally, if you've confirmed the user is a [qualified Apple Pay user](#User-Requirements) (```canMakePayments``` result has returned ```{ applePay: true }```) you can provide an [Apple Pay Authorization Result](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentauthorizationresult) for more descriptive error messages.
            - In this case - there will be a ```status``` and an ```errors``` property. The status must be provided in a recognized format ([as demonstrated in the Apple docs](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentauthorizationresult)).
                - Since this is an error scenario it's likely that the value needed would be: ```ApplePaySession.STATUS_FAILURE```.
                - It's important to confirm that the user is a [qualified Apple Pay user](#User-Requirements) before providing this object - as ```ApplePaySession``` is a global variable that will not exist (throw an error) in all other browsing contexts.
            - The ```errors``` can be supplied as explained in the [Payment Request Error Handling](#Payment-Request-Error-Handling) section.  
            
```confirm-token``` example: 
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var isAppleUser = false;

    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( availablePrMethods => {
        if (availablePrMethods && availablePrMethods.applePay === true) {
            isAppleUser = true;
            tecPrInterface.mountTecPrButton("example-div");
        }
    });

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //You can complete with string value, as below.
            //completePayment("failure");

            //Optionally - you can provide more concise error messages
            if (isAppleUser === true) {
                completePayment({
                    status: ApplePaySession.STATUS_FAILURE,
                    errors: [
                        new ApplePayError("shippingContactInvalid", "postalCode", "ZIP Code is invalid"),
                        new ApplePayError("addressUnserviceable", "addressLines", "Cannot deliver to P.O. Box")
                    ]
                })
            }
        }
        else {
            completePayment('success')
        }
    });
}

demoInit();
```  

### ```payment-method-selection``` Event
When listening to the ```payment-method-selection``` event - there will be two special properties to the event:
- ```paymentMethod```
    - This object will provide details about the currently selected payment method. If you subscribe to this event - it will fire at least once (when the payment form loads), and will fire every time the user changes the payment method (during the current session). You must call ```completePaymentMethodSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethod).
- ```completePaymentMethodSelection```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the payment form.
    - A [Payment Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethodupdate) object is required to invoke this function. ```'newTotal'``` [line item](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaylineitem) will reflect the total cost of all the purchased items.
        - More information on the [Payment Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentmethodupdate) object.
        - More information on [line items](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaylineitem)
    - Only provide ```'newLineItems'``` if there are new or updated costs or discounts (depending on the payment method the user has selected) - Otherwise pass an empty array or ```null```.  


### ```shipping-contact-update``` Event
When listening to the ```shipping-contact-update``` event - there will be two special properties to the event:
- ```shippingContact```
    - This object will provide details about the currently selected shipping contact. If you subscribe to this event - it will fire every time the user changes their shipping contact (during the current session) - or if they edit their selected shipping contact mid-session. You must call ```completeShippingContactSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentcontact).
- ```completeShippingContactSelection```
    - Call this function within 30 seconds of receiving it - otherwise a timeout error will occur and close the payment form.
    - A [Shipping Contact Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate) object is required to be supplied to this function.
        - More information about the [update object is here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate) 

### ```shipping-method-update``` Event
When listening to the ```shipping-method-update``` event - there will be two special properties to the event:
- ```shippingMethod```
    - This object will provide details about the currently selected shipping method. If you subscribe to this event - it will fire every time the user changes their shipping method (during the current session). You must call ```completeShippingMethodSelection``` within 30 seconds of receiving the event - or a timeout will occur and close the payment form.  
    - The object's structure is defined in [Apple's documentation here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingmethod).
- ```completeShippingMethodSelection```
    - A [Shipping Method Update](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingmethodupdate) object is required to be supplied to this function.
        - More information about the [update object is here](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingcontactupdate)

### ```cancel-transaction``` Event
When listening to the ```cancel-transaction``` event - there will only be a ```reason``` notifying you that the customer's payment request form has been dismissed.

# Payment Request Button Options
The ```TecPaymentRequestButtons``` component accepts an optional prop named ```tecPrOptions```.  Here you can define your custom options for the payment request buttons.  At this time - Apple Pay is the only supported platform. You may define custom values for the Apple Pay Button, using the property ```applePayOptions``` (You can see the [default values](#Default-Payment-Request-Button-Options-Values), for an example on how to structure the object, [below]).

## Apple Pay Button Options
There are many options and styles available to assist with tailoring the Apple Pay Button to your specifications. There are a few things to be aware of:
- Apple has defined [guidelines for displaying the Apple Pay button](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks)
    - The "Button Types" and "Button Styles" mentioned in the [guidelines](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks) can be supplied in the Apple Pay Button Options.
- ```buttonLanguage``` and ```buttonType``` values are not validated, and will be applied directly to the button's style properties. Please check the supplied value for accuracy. 
    - ```buttonStyle```, however, is validated, and will display an accurate value supplied - or default to ```'black'```, if the value is not recognized.
- In addition to the Apple Pay Button options available - it is also possible to apply your own custom styles to the button using CSS. Target the button's ```id```:
    - ```"te-connect-apple-pay-btn"```
    - When applying custom CSS - please be sure to adhere to [Apple's Guidelines](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/buttons-and-marks)

| Property Name | Type | Possible Values |
|:--:|:--:|:--:|
| ```buttonLanguage``` | ```string``` | [available button languages](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttonlocale) | 
```buttonType``` | ```string``` | [available button types](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttontype) |
| ```buttonStyles``` | ```string``` | ```'black'``` ```'white'``` ```'white-outline'``` [available button styles](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaybuttonstyle)  |

#### Default Payment Request Button Options Values:  
  
```javascript
const defaultButtonOptions = {
    applePayOptions: {
        buttonLanguage: 'en',
        buttonType: 'plain',
        buttonStyle: 'black'
    }
};
```

# Example Implementation
There are two example implementations below - a minimally viable, as well as a more complex implementation. Be aware that any implementation is also available via CDN. There is [a manual entry example using our CDN](README.md#Example-Implementation-CDN) for reference on implementation.  

Below you will find a minimally viable implementation.  

```index.html```
```html
<body>
    <h1>Token Exchange Connect Apple Pay</h1>
    <div id="example-div"></div>
    <script src='main.bundle.js'></script>
</body>
```

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div");
    });

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //Unsuccessful - check message.
            completePayment('failure');
        }
        else {
            //Successful - close user's payment request form with payment success notification.
            completePayment('success');
        }
    });
}

demoInit();
```  


Below you will find a more complex implementation:  

```index.html```
```html
<body>
    <h1>Token Exchange Connect Apple Pay</h1>
    <div id="example-div"></div>
    <script src='main.bundle.js'></script>
</body>
```

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

const customButtonOptions= {
    applePayOptions: {
        buttonLanguage: 'ja',
        buttonType: 'book',
        buttonStyle: "white-outline"
    }
};

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult)
            tecPrInterface.mountTecPrButton("example-div");
    });

    tecPrInterface.listenFor('confirm-token', tokenResp => {
        const { tokenDetails, completePayment, error } = tokenResp;

        if (error) {
            //Unsuccessful - check message.
            completePayment('failure');
        }
        else {
            //Successful - close user's payment request form with payment success notification.
            completePayment('success');
        }
    });

    tecPrInterface.listenFor('cancel-transaction', (cancelEvent) => {
        console.log(cancelEvent);
    });

    tecPrInterface.listenFor('payment-method-selection', (paymentMethodEvent) => {
        const { paymentMethod, completePaymentMethodSelection } = paymentMethodEvent;

        const examplePaymentUpdateObj = {
                newTotal: {
                    label: "Updated Payment Method",
                    amount: "0.00",
                    type: "final"
                }
            }

        if (paymentMethod.type === "credit") {
            //Credit logic - if needed
            const creditUpdateObj = { 
                newTotal: {
                    ...examplePaymentUpdateObj['newTotal'],
                    amount: "1.10"
                }
            }

            completePaymentMethodSelection(creditUpdateObj);
        }
        else if (paymentMethod.type === "debit") {
            //Debit logic - if needed
                const debitUpdateObj = { 
                newTotal: {
                    ...examplePaymentUpdateObj['newTotal'],
                    amount: "0.10"
                }
            }

            completePaymentMethodSelection(debitUpdateObj);
        }

        completePaymentMethodSelection(examplePaymentUpdateObj);
    });

    tecPrInterface.listenFor('shipping-contact-update', (shippingContactEvent) => {
        const { shippingContact, completeShippingContactSelection } = shippingContactEvent;

        const exampleShippingContactUpdateObj = {
            newTotal: {
                label: "Test Positive Transaction for Shipping Update",
                amount: "1.00",
                type: "final"
            },
            newShippingMethods: [
                {    
                    label: 'Outside Shipping Network',
                    detail: "Arrives in 7-12 weeks",
                    amount: '100.00',
                    identifier: "OutOfCountryShip"
                },
                {    
                    label: 'Free Shipping',
                    detail: "Arrives in less than week",
                    amount: '0.00',
                    identifier: "FreeShipping"
                }
            ]
        }

        const errObj = {
            newTotal: {
                label: "Test Negative Transaction for Shipping Update",
                amount: "1.00",
                type: "final"
            },
            errors: [
                {
                    errorType: "addressInvalid",
                    message: "Cannot Ship Outside the US"
                }
            ]
        }

        completeShippingContactSelection(
            (shippingContact.countryCode === "US") ? exampleShippingContactUpdateObj : errObj
        );
    });

    tecPrInterface.listenFor('shipping-method-update', (shippingMethodUpdateEvent) => {
        const { shippingMethod, completeShippingMethodSelection } = shippingMethodUpdateEvent;

        const freeShipping = {
            newTotal: {
                label: "Item with Free Shipping",
                amount: "1.00",
                type: "final"
            }
        }

        const outOfCountryShipping = {
            newTotal: {
                label: "Item with Shipping Out of Network",
                amount: "101.00",
                type: "final"
            }
        }

        if (shippingMethod.identifier === "OutOfCountryShip")
            completeShippingMethodSelection(outOfCountryShipping);
        else
            completeShippingMethodSelection(freeShipping);
    });
}

demoInit();
```  

# Payment Request Error Handling
This package is designed to be expanded upon in the future to include more Payment Request Providers. To that end, there are two [generic errors](#Generic-Errors) that will work regardless of payment request type/platform.  However, if you know which payment request type the user is currently interacting with - you may also provide [platform specific errors](#Platform-Specific-Errors) that can provide a more detailed error experience for your users. The ```errors``` array will accept either [generic errors](#Generic-Errors) or [platform specific errors](#Platform-Specific-Errors), or a mixture of the two.
## Generic Errors
Generic errors are an object that accept a ```"type"``` and ```"message"```.  There are currently two types:
- ```"addressInvalid"```
    - This error will notify user that the address they supplied (normally related to shipping address) is invalid for the transaction they chose. The message you supply can provide more details, if needed.
- ```"failure"```
    - This error will notify user with an "unknown" error and a message. Be aware that there are some instances (especially related to address info) in which the custom error message will not display correctly, and will only display an "unknown" error to the user. Since this option is difficult to direct the user how to correct the error - it's recommended to use this to catch errors which are actually unknown.
- Example:

```javascript
{
    type: "addressInvalid",
    message: "Shipping not available in selected country"
}
```
## Platform Specific Errors
Platform specific errors rely on proprietary JavaScript, available only on the target platform. Beware that attempting to build a platform specific error on the incorrect platform (i.e. building a ```new ApplePayError``` on a Chrome browser) will result in errors.  

To that end - if you plan to provide platform specific errors - ensure the code only executes if the target platform is available. Here is an ApplePay example error below:

```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

import { examplePaymentRequestObject } from '../consts';

function demoInit() {
    var teInstance = createTEConnect("__publicKeyGoesHere__", { 
        tecPaymentRequest: { appleMerchantId: "__tecAppleMerchantId__" }
    });

    var teConnect = new TeConnectJs(teInstance);

    var tecPrInterface = teConnect.createTecPaymentRequest(examplePaymentRequestObject);

    tecPrInterface.canMakePayments().then( canMakePaymentsResult => {
        if (canMakePaymentsResult && canMakePaymentsResult.applePay) {
            //User is using Safari and has ApplePay capabilities (Wallet), with an active card loaded on their device.
            var exampleError = new ApplePayError("shippingContactInvalid", "postalCode", "ZIP Code is invalid");
        }
    });
}

demoInit();
```  

### Apple Pay
Apple Pay accepts errors that are built using the ```ApplePayError``` class.  
- Please ensure the class is available by either leveraging the [```canMakePayments```](#canMakePayments) function - or checking the ```window``` object for the ```ApplePayError``` class.
- [Here is Apple's documentation for ```ApplePayError```](https://developer.apple.com/documentation/apple_pay_on_the_web/applepayerror)


# Apple Pay Button Requirements
Once the steps above are followed, and an Apple Pay capable web app is being served (for both development and production) the Apple Button will only display when __all__ pre-requisites are fulfilled:
- Web Application pre-requisites:
    - Must be registered as a Magensa™ Token Exchange customer with Apple Pay capabilities.
        - This will grant you the ```appleMerchantId``` needed to supply to the ```teConnect``` instance.
        - Here is a link for [Magensa™ Support](https://magensa.net/support.html) - if needed.
    - Must register and verify public domain with Magensa™. Public domain must be served via ```https://``` only.
- End-User's (Browsing) Pre-requisites:
    - Must be using a compatible device:
        - Compatible iOS or macOS device with Apple Pay capabilities:
        -   [Apple's list of compatible device](https://support.apple.com/en-us/HT208531)
    - Must have an active card loaded into the "Wallet" of the compatible device.
    - Must be using Safari as the browsing agent.
