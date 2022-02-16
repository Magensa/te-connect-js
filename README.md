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
    <script src="https://cdn.magensa.net/te-connect/1.1.1/te-connect.js"></script>
    <script src="https://cdn.magensa.net/te-connect-js/1.1.1/te-connect-js.js"></script>
```

If you would prefer to let the code speak, below we have two [example implementations](#Example-Implementation). 
One is using npmjs - while [the other](#Example-Implementation-CDN) uses our CDN  


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
| tecPaymentRequest | ```TecPaymentRequestOptions``` | See the [Payment Request README for more info](TecPaymentRequestREADME.md) |
| appleMerchantId | string | This property is part of a child object supplied to ```tecPaymentRequest```. Provide a valid ```appleMerchantId``` to [opt in for Apple Pay](TecPaymentRequestREADME.md) |

```typescript
type TecPaymentRequestOptions = {
    appleMerchantId?: string
}

type CreateTEConnectOptions = {
    hideZip?: boolean,
    tecPaymentRequest?: TecPaymentRequestOptions
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
    }
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

# Styles API
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
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes |  
| autoMinHeight | variants | ```boolean``` | ```boolean``` | ```false``` | ```true``` will maintain a static margin on each input box that will not grow with validation errors | 
  
<br />

### Default Base Object:
```javascript
{
    base: {
        wrapper: {
            margin: '1rem',
            padding: '1rem'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'normal',
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

    <script src="https://cdn.magensa.net/te-connect/1.1.1/te-connect.js"></script>
    <script src="https://cdn.magensa.net/te-connect-js/1.1.1/te-connect-js.js"></script>

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
