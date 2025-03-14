# TEConnect JS
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-js.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-js "@magensa/te-connect-js npm.js")  

JavaScript module used to interact with TEConnect.  

# Getting Started
Via [npmjs](https://www.npmjs.com/package/@magensa/te-connect-js):  

```
npm install @magensa/te-connect @magensa/te-connect-js
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-js
```  
  
or via CDN:
```html
    <script src="https://cdn.magensa.net/te-connect/1.3.2/te-connect.js"></script>
    <script src="https://cdn.magensa.net/te-connect-js/1.3.0/te-connect-js.js"></script>
```

# Manual Card Entry
This document will cover the card manual entry utility that TEConnect offers. 3DS Manual Entry is also available, if a valid 3DS API Key is supplied to [the `options` parameter](#TEConnect-Options).  
TEConnect also offers a [Payment Request](https://github.com/Magensa/te-connect-js/blob/master/TecPaymentRequestREADME.md) utility as well, with both Apple Pay and Google Pay supported. [Payment Request Documentation can be found here](https://github.com/Magensa/te-connect-js/blob/master/TecPaymentRequestREADME.md)  
Below we will begin with a step-by-step integration of the card manual entry utility.  
If you would prefer to let the code speak, below we have [example implementations](#Example-Implementation). 
One is using npmjs, [another](#Example-Implementation-CDN) uses our CDN, and the final uses [3DS Manual Entry](#TecThreeDs-Example).

## Magensa™
TEConnect Manual Entry, 3DS Manual Entry and [TEConnect Payment Request](https://github.com/Magensa/te-connect-js/blob/master/TecPaymentRequestREADME.md) components all require a valid [Magensa™](https://magensa.net/) account. If you need assistance creating or configuring an account, please reach out to the [Magensa Support Team](https://magensa.net/support.html).  


# Step-by-step
1. The first step is to create a TEConnect instance (from ```te-connect```), and feed that instance to the ```TeConnectJs``` (from ```te-connect-js```) constructor.

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__");
    var teConnect = new TeConnectJs(teInstance);
}

demoInit();
```  

2. Next, design your page according to your specifications - and create a ```<div>``` with a unique id. Mount the ```TEConnect``` Card Entry on that div.  

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

function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__");
    var teConnect = new TeConnectJs(teInstance);

    teConnect.mountCardEntry('example-div');
}

demoInit();
```

3. Finally, to submit the card values, attach the ```createPayment``` method to a click handler of your choosing.  

```index.html```
```html
<body>
    <div id="example-div"></div>
    <button type="button" id="pay-button">Create Payment</button>
    <script src='main.bundle.js'></script>
</body>
```  

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__");
    var teConnect = new TeConnectJs(teInstance);

    teConnect.mountCardEntry('example-div');

    function createPaymentClick() {
        return teConnect.createPayment()
            .then(resp => {
                if (resp.error) {
                    //Not successful - see error
                }
                else {
                    //Successful - see response
                }
            })
            .catch(err => console.error(err));
    }

    document.getElementById('pay-button').addEventListener('click', createPaymentClick);
}

demoInit();
```  

4. There are two ways to inject your own styles into the Card Entry fields. One method is static (define styles when mounting), while the other is dynamic (utilize the ```setStyles``` method). The [example implementation](#Example-Implementation) uses both of these methods to demonstrate various ways you can control your custom styles. See the [complete styles API](#Styles-API) for more details.  

5. Optionally, you may hide the ZIP input box, as demonstrated below. Once the input is hidden, the ```billingZip``` parameter becomes optional. If you still wish to provide a ```billingZip``` without rendering the input - you may do so, as demonstrated below (this would hide the ZIP input field, but would still create a payment token with the ```billingZip``` provided in the call).   
However, if you wish to omit the ```billingZip``` completely, you may do so by hiding the zip input and passing no parameters to the ```createPayment``` function.

```index.html```
```html
<body>
    <div id="example-div"></div>
    <button type="button" id="pay-button">Create Payment</button>
    <script src='main.bundle.js'></script>
</body>
```  

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__", { hideZip: true });
    var teConnect = new TeConnectJs(teInstance);

    teConnect.mountCardEntry('example-div');

    var exampleZipCode = "90025";

    function createPaymentClick() {
        return teConnect.createPayment(exampleZipCode)
            .then(resp => {
                if (resp.error) {
                    //Not successful - see error
                }
                else {
                    //Successful - see response
                }
            })
            .catch(err => console.error(err));
    }

    document.getElementById('pay-button').addEventListener('click', createPaymentClick);
}

demoInit();
``` 

# TEConnect Options
The second parameter of the ```createTEConnect``` method is an options object. This object is optional.
| Property Name  | Input Type | Notes |
|:--:|:--:|:--:|
| billingZip | ```boolean``` | See implementation above |
| tecPaymentRequest | ```TecPaymentRequestOptions``` | See the [Payment Request README for more info](https://github.com/Magensa/te-connect-js/blob/master/TecPaymentRequestREADME.md) |
| threeds | `TecThreeDsOptions` | See [TecThreeDs](#TecThreeDs-3DS) for more info |

```typescript
type TecPaymentRequestOptions = {
    appleMerchantId?: string,
    googleMerchantId?: string
}

type TecThreeDsOptions = {
    threedsApiKey: string,
    threedsEnvironment: "sandbox" | "production"
}

type CreateTEConnectOptions = {
    hideZip?: boolean,
    tecPaymentRequest?: TecPaymentRequestOptions,
    threeds?: TecThreeDsOptions
}
```

<br />

# TecThreeDs (3DS) Optional Feature
TEConnect Manual Entry offers 3DS Manual Entry. To opt-in, add a `threeds` parameter to the [the ```options``` object](#TEConnect-Options) for the `createTEConnect` method.  
Add a 3DS API key (`threedsApiKey`) to the `threeds` object, and (optionally) a `threedsEnvironment` string as well (defaults to `sandbox` for testing, when not specified). Only flip your project to `production` when you have tested with `sandbox`, and are ready to deploy to production.  

[3DS `sandbox` Test Cards can be found here](https://docs.3dsintegrator.com/docs/test-cards#emv-3ds-test-cards).

### _Opt In_
```javascript
    const teInstance = createTEConnect("__publicKeyGoesHere__", { 
        threeds: {
            threedsApiKey: "__3dsApiKeyGoesHere__",
            threedsEnvironment: "sandbox"
        }
    });
```  

3DS workflow is used in a similar manner as manual entry. The only difference: A required [`threeDsConfigObject`](#ThreeDsConfigObject) must be fed to the `configureThreeds` method.  For a minimally viable 3DS solution, that is the only additional requirement needed complete the manual entry workflow. However, there are more optional methods that can be leveraged to communicate with your 3DS instance.  

See the [TecThreeDs Example](#TecThreeDsExample) for an example implementation.

### `threeDsInterface`
| Method Name  | Parameters | Notes |
|:--:|:--:|:--:|
| configureThreeds | `ThreeDsConfigObject` | This is the only method required to begin a 3DS Manual Entry instance. More info on required properties and options [can be found here](#ThreeDsConfigObject) |
| listenFor | eventType (as `string`),  listener (as `function`) | Currently, `"threeds-status"` is the only `eventType` available to listen for [3DS status updates](#ThreeDs-Status-Listeners) |
| updateThreedsConfig | `ThreeDsConfigObject` | Full update (complete object). Updates are only available prior to 3DS auth. Once cardholder has entered card info, object is not able to be updated |

See the [TecThreeDs Example](#TecThreeDsExample) for example uses for these methods.

## `ThreeDsConfigObject`
TEConnect forwards the `threeDsConfigObject` to 3DS API call 'authenticate browser'. This method begins the 3DS process; TEConnect handles the interactions on your application's behalf while the cardholder is entering the card info. TEConnect only requires two properties to get started: 
- `challengeNodeId: string` - the id of a node that must be mounted on the DOM. This node is used to mount a "challenge" iframe. 
    - The challenge iframe is defined by the issuer of the card being authenticated (in the event that a "challenge" is required to complete the 3DS authentication). This challenge varies based upon the issuer of the card. [See ThreeDsChallenge for more info](#ThreeDs-Challenge)
- `amount: number` - the final amount for the transaction. Required for 3DS Auth. 

Please note: There are four properties, documented in the [3DS API documentation](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), that TEConnect handles and will be ignored if supplied in the ThreeDsConfigObject: `browser` object, and card info (`pan`, `year`, and `month`).

There are many optionally properties available, in addition to the required `challengeNodeId` and `amount` properties. 3DS has extensive documentation on the properties available in this call, which [can be found here](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser).  With the exception of the `browser` object, `pan`, `year`, and `month` - all other options supplied to the `configureThreeds` method will be forwarded to the 3DS API call.


## ThreeDs Status Listeners
### `threeds-status`
Subscribe to this event to listen for status updates with the 3DS workflow.  
See the [TecThreeDs Example](#TecThreeDs-Example) for an example how to subscribe.

```typescript
type ThreeDsMethods = "GENERATE_JWT" | "AUTHENTICATE_BROWSER" | "FINGERPRINT_DEVICE" | "CHALLENGE";
type ThreeDsStatus = "REQUESTED" | "SUCCESS" | "FAIL" | "AWAIT_RESULTS";

type ThreeDsStatusEvent = {
    threedsMethod: ThreeDsMethods,
    status: ThreeDsStatus,
    message: string
}
```

## ThreeDs Challenge
There are many possible 3DS responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)).  When a status of type `"C"` is returned - this means that the card issuer has requested the cardholder to complete a challenge, in order to proceed. In this case - TEConnect will build the challenge iframe, and mount it on the node with the id provided (via `challengeNodeId`).  

The particular challenge rendered varies greatly depending on card issuer. The cardholder must complete the challenge in order to proceed with 3DS Authentication.

There are a few options available in the [ThreeDsConfigObject](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), in regards to a challenge.  Such as `challengeIndicator` to specify if a challenge is desired.  Other options include `challengeWindowSize` and `transactionForcedTimeout` (if an early timeout is desired).  

TEConnect will attempt to collect the challenge results 10 seconds after a challenge is rendered. If the results are not yet available, it will poll until results are available or timeout. It will timeout after 60 seconds.

Since it's unknown what the challenge will look like, as well as the operations needed to complete the challenge - it's recommended to mount the challenge on a node that is styled with `absolute` positioning.  If you wish to place the challenge in a modal - make use of the [`threeds-status`](#ThreeDs-Status-Listeners) and listen for the status: `{ threedsMethod: "CHALLENGE", status: "REQUESTED" }` to trigger the modal.  

## Additional ThreeDs Considerations
3DS can have many potential responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)). While 3DS auth may or may not be successful, or approved - please note that a token is always returned with a successful response, and can be used to call MPPG's `ProcessToken` for processing.  It's up to the merchant whether or not to assume the risk in the transaction. You may want to be familiar with responses, as they relate to various card networks, to determine if a liability shift has occured.  For example: [Visa](https://docs.3dsintegrator.com/docs/visa), [MasterCard](https://docs.3dsintegrator.com/docs/mastercard#faud-liability-shift-conditions), etc.  

When `"sandbox"` environment is in use - the `threeDSRequestorURL` will use a placeholder value of `"https://your.domainname.com"`. When flipped to `"production"` - the origin of your web application will be used.  This is because `http:` and `localhost` domains fail validation for 3DS calls, but are useful for early development.  Be aware that your `"production"` web application must be deployed using a valid `https://` domain, for 3DS to be successful.

<br />  
 

# Styles API
This Styles API is available to customize the form rendered for manual entry.  You may also style the container itself, using CSS, by targeting the id: `"__te-connect-secure-window"`.  

For 3DS Challenges: the `"#_threeds-challenge-frame-request"` is the iframe element in which the card issuer may render their challenge to the card holder. You may target this element to apply styles to the challenge container.  

The styles object injected is comprised of two main properties:
- ```base```  
    - General styles applied to the container.
- ```boxes```
    - Styles applied to the input elements.

Below we have the complete API with examples of default values for each.

### Base
| Property Name | Parent Property | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|:---:|
| backgroundColor | base | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | container background color |
| margin | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container margin |
| padding | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container padding |
| direction | wrapper | ```string``` | ```'row', 'row-reverse', 'column', 'column-reverse'``` | ```'row'``` | ```'flex-direction'``` style property |
| flexWrap | wrapper | ```string``` | ```'wrap', 'wrap', 'wrap-reverse'``` | ```'wrap'``` | ```'flex-wrap'``` style property |
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes |  
| inputSize | variants | ```string``` | ```"small", "medium"``` | ```"medium"``` | input component size |  
| autoMinHeight | variants | ```boolean``` | ```boolean``` | ```false``` | ```true``` will maintain a static margin on each input box that will not grow with validation errors | 
  
<br />

### Default Base Object:
```javascript
{
    base: {
        wrapper: {
            margin: '1rem',
            padding: '1rem',
            direction: 'row',
            flexWrap: 'wrap'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'normal',
            inputSize: 'medium',
            autoMinHeight: false
        },
        backgroundColor: '#fff'
    }
}
```
<br />

### Boxes
| Property Name | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|
| labelColor | ```string``` | jss color (rgb, #, or color name) | ```"#3f51b5"``` | label text and input outline (or underline) color |
| textColor | ```string``` | jss color (rgb, #, or color name) | ```"rgba(0, 0, 0, 0.87)"``` | color of text for input value *Also applies :onHover color to outline/underline* |
| borderRadius | ```number``` | numerical unit for css ```border-radius``` property | 4 | border radius for input boxes |
| inputColor | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | input box background color |  
| errorColor | ```string``` | jss color (rgb, #, or color name) | ```"#f44336"``` | Error text and box outline (or underline) color |  
  
<br />

### Default Boxes Object:
```javascript
{
    boxes: {
        labelColor: "#3f51b5",
        textColor: "rgba(0, 0, 0, 0.87)",
        borderRadius: 4,
        errorColor: "#f44336",
        inputColor: "#fff"
    }
}
```  


# createPayment Return Objects
These are the possible objects that will be returned *successfully* from the ```createPayment``` function. Thrown errors will be thrown as any other async method.  
  
  1. ### Success:
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: String,
    code: String,
    message: String
    status: Number,
    cardMetaData: null | {
        maskedPAN: String,
        expirationDate: String,
        billingZip: null | String
    },
    threedsResults: ThreeDsResults
}
```  
  
  2. ### Bad Request
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: null,
    code: String,
    message: String,
    error: String,
    cardMetaData: null
}
```

3. ### Error (Failed Validation, Timeout, Mixed Protocol, etc)
```typescript
{ error: String }
```

4. ### ThreeDsResults (property of Success response, if 3DS opt-in)  
    All responses documented in the [3DS Documentation](https://docs.3dsintegrator.com/reference/subscribetoupdates)  

```typescript
type ThreeDsResults = {
    scaRequired?: bool,
    creq?: string,
    status?: string,
    authenticationValue?: string,
    authenticationType?: string,
    eci?: string,
    acsUrl?: string,
    dsTransId?: string,
    acsTransId?: string,
    sdkTransId?: string,
    error?: string,
    errorDetail?: string,
    errorCode?: string,
    cardholderInfo?: string,
    transactionId?: string,
    correlationId?: string,
    code?: number
}
```  


# Example Implementation
```index.html```
```html
<body>
    <h1>TEConnectJS Example</h1>
    <div id="example-div"></div>
    <button type="button" id="pay-button">Create Payment</button>
    <button type="button" id="change-styles">Change Styles</button>
    <script src='main.bundle.js'></script>
</body>
```  

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';

function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__");
    var teConnect = new TeConnectJs(teInstance);

    var exampleStyles = {
        base: {
            wrapper: { 
                margin: 0, 
                padding: 0
            },
            variants: {
                inputType: 'outlined',
                inputMargin: 'dense'
            },
           backgroundColor: 'rgb(66, 66, 66)'
        },
        boxes: {
          labelColor: "#BB86FC",
          textColor: "#BB86FC",
          borderRadius: 10,
          errorColor: "#CF6679",
          inputColor: '#121212'
        }
    };

    teConnect.mountCardEntry('example-div', exampleStyles);

    function createPaymentClick() {
        return teConnect.createPayment()
            .then(resp => {
                if (resp.error) {
                    //Not successful - see error
                }
                else {
                    //Successful - see response
                }
            })
            .catch(err => console.error("[Catch]:", err));
    }

    function changeStyles() {
        var colorful = {
            base: {
                wrapper: { 
                    margin: 0, 
                    padding: 0
                },
                variants: {
                    inputType: 'filled',//'outlined', 'filled', 'standard'
                    inputMargin: 'dense'  //'dense', 'none', 'normal'
                },
               backgroundColor: '#f44336'
            },
            boxes: {
                textColor: "#90ee02",
                borderRadius: 2,
                errorColor: "#2196f3",
                inputColor: '#ff7961'
            }
        };

        teConnect.setStyles(colorful);
    }

    document.getElementById('pay-button').addEventListener('click', createPaymentClick);
    document.getElementById('change-styles').addEventListener('click', changeStyles);
}

demoInit();
```
  
# Example Implementation CDN
### An important note about CDN versioning: 
All ```te-connect-js``` versions will be available via CDN. You can see the version in the path below.  
If you are starting a new project - it is recommended to start with the most recent version. Therefore, even if you are not using npmjs, it is still recommended to view the most recent published version at [npmjs](https://www.npmjs.com/package/@magensa/te-connect-js), so that you may target the most recent version. CDN and npm packages are versioned simultaneously.  
Alternatively - if your project requires a specific version - you may target that version in the CDN path.  

```index.html```
```html
<body>
    <h1>TEConnectJS Example</h1>

    <div id="example-div"></div>
    <button type="button" id="pay-button">Create Payment</button>
    <button type="button" id="change-styles">Change Styles</button>

    <script src="https://cdn.magensa.net/te-connect/1.3.2/te-connect.js"></script>
    <script src="https://cdn.magensa.net/te-connect-js/1.3.0/te-connect-js.js"></script>

    <script>
        function demoInit() {
            var teInstance = window["te-connect"].createTEConnect("__your_public_key_here__");
            var teConnect = new window["te-connect-js"].TeConnectJs(teInstance);

            var exampleStyles = {
                base: {
                    wrapper: { 
                        margin: 0, 
                        padding: 0
                    },
                    variants: {
                        inputType: 'outlined',
                        inputMargin: 'dense'
                    },
                    backgroundColor: 'rgb(66, 66, 66)'
                },
                boxes: {
                    labelColor: "#BB86FC",
                    textColor: "#BB86FC",
                    borderRadius: 10,
                    errorColor: "#CF6679",
                    inputColor: '#121212'
                }
            };

            teConnect.mountCardEntry('example-div', exampleStyles);

            function createPaymentClick() {
                return teConnect.createPayment()
                    .then(resp => {
                        console.log(resp);

                        if (resp.error) {
                            //Not successful - see error
                        }
                        else {
                            //Successful - see response
                        }
                    })
                    .catch(err => console.error("[Catch]:", err));
            }

            function changeStyles() {
                var colorful = {
                    base: {
                        wrapper: { 
                            margin: 0, 
                            padding: 0
                        },
                        variants: {
                            inputType: 'filled',
                            inputMargin: 'dense'
                        },
                    backgroundColor: '#f44336'
                    },
                    boxes: {
                        textColor: "#90ee02",
                        borderRadius: 2,
                        errorColor: "#2196f3",
                        inputColor: '#ff7961'
                    }
                };

                teConnect.setStyles(colorful);
            }

            document.getElementById('pay-button').addEventListener('click', createPaymentClick);
            document.getElementById('change-styles').addEventListener('click', changeStyles);
        }

        demoInit();
    </script>
</body>
```  
  

# TecThreeDs Example
This example uses the `threeDsInterface` methods. Note that this is optional; only the `configureThreeds` method is required to create a token with 3DS response.   

```index.html```
```html
<body>
    <h1>TEConnectJS ThreeDs Example</h1>
    <div id="example-div"></div>
    <button type="button" id="pay-button">Create Payment</button>
    <button type="button" id="change-styles">Change Styles</button>
    <button type="button" id="update-threeds">Update ThreeDs Config</button>
    <div id="example-challenge-node"></div>
    <script src='main.bundle.js'></script>
</body>
```  

```main.js```
```javascript
import { createTEConnect } from '@magensa/te-connect';
import { TeConnectJs } from '@magensa/te-connect-js';


const exampleThreedsConfig = {
    amount: 100,
    challengeNodeId: "example-challenge-node"
};

const exampleStyles = {
    base: {
        wrapper: { 
            margin: 0, 
            padding: 0
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'dense'
        },
        backgroundColor: 'rgb(66, 66, 66)'
    },
    boxes: {
        labelColor: "#BB86FC",
        textColor: "#BB86FC",
        borderRadius: 10,
        errorColor: "#CF6679",
        inputColor: '#121212'
    }
};


function demoInit() {
    var teInstance = createTEConnect("__your_public_key_here__", { 
        threeds: {
            threedsApiKey: "__3dsApiKeyGoesHere__",
            threedsEnvironment: "sandbox"
        }
    });
    
    var teConnect = new TeConnectJs(teInstance);

    teConnect.configureThreeds(exampleThreedsConfig);

    teConnect.listenFor("threeds-status", function(threedsStatus) {
        console.log('[3DS Status Listener]:', threedsStatus);
    }

    teConnect.mountCardEntry('example-div', exampleStyles);

    function createPaymentClick() {
        return teConnect.createPayment()
            .then(resp => {
                if (resp.error) {
                    //Not successful - see error
                }
                else {
                    //Successful - see response

                    const { threedsResults } = resp;

                    if (threedsResults) {
                        const threeDsStatus = threedsResults?.status?.toLowerCase();

                        if (threeDsStatus) {
                            switch(threeDsStatus) {
                                case 'y':
                                    //Authentication Successful
                                    break;
                                case 'a':
                                    //Auth not available, but liability shift still occurs
                                    break;
                                case 'n':
                                    //Not Authenticated. Transaction denied.
                                    break;
                                case 'u':
                                    //Technical or other problem has occured while contacting the issuer.
                                    break;
                                case 'r':
                                    //Issuer has rejected Authentication, and has requested the transaction not to be processed.
                                    break;
                                default:
                                    break;
                            }
                        }
                    }
                }
            })
            .catch(err => console.error("[Catch]:", err));
    }

    function changeStyles() {
        var colorful = {
            base: {
                wrapper: { 
                    margin: 0, 
                    padding: 0
                },
                variants: {
                    inputType: 'filled',//'outlined', 'filled', 'standard'
                    inputMargin: 'dense'  //'dense', 'none', 'normal'
                },
               backgroundColor: '#f44336'
            },
            boxes: {
                textColor: "#90ee02",
                borderRadius: 2,
                errorColor: "#2196f3",
                inputColor: '#ff7961'
            }
        };

        teConnect.setStyles(colorful);
    }

    function updateThreeDsConfig() {
        teConnect.updateThreedsConfig({
            amount: 110,
            challengeNodeId: "example-challenge-node",
            challengeWindowSize: "05",
            transactionForcedTimeout: "14"
        });
    }

    document.getElementById('pay-button').addEventListener('click', createPaymentClick);
    document.getElementById('change-styles').addEventListener('click', changeStyles);
    document.getElementById('update-threeds').addEventListener('click', updateThreeDsConfig);
}

demoInit();
```
