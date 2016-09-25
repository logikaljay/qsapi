# QSAPI

Quasi-API - Hand sanitiser for your API

* <a href="#intro">Why?</a>
* <a href="#usage">Usage</a>
    * <a href="#fetch">Fetch</a>
    * <a href="#schema">Schema modelling</a>
* <a href="#examples">Examples</a>
    * <a href="#fetchExample">Fetch examples</a>
    * <a href="#schemaExample">Schema mapping example</a>
* <a href="#api">API</a>
* <a href="#todo">TODO</a>

<a name="intro"></a>
# Why?
Sometimes API's are bad. Sometimes they fail, Sometimes they don't.
Your application shouldn't have to deal with intermittent API issues, It shouldn't have to deal with mismatched property types, or properties missing altogether.

<a name="usage"></a>
# Usage

<a name="fetch"></a>
## Fetch

QSAPI presumes that the API being called is unstable and often unavailable. It will by default attempt to fetch the resource data 3 times before rejecting the original promise. This default can be configured during initialisation of the QSAPI call.

Using `fetch`, in its most basic form, all you need to supply is a `url`, everything else is handled by the default values.

<a name="schema"></a>
## Schema modelling

A schema can be provided to QSAPI to transform the result of the API call to the expected object.
This can be used to make sure the data coming back from the API is uniform and consistant to what the UI is expecting.


# Examples

<a name="fetchExample"></a>
## Fetch examples

*Basic example*

Make a GET request to google.com, timeout after 1 second, don't retry.

```js
var opts = {
    url: 'http://www.google.com',

    // timeout after 1 second
    timeout: 1000,

    // don't retry
    retry: false
}

var instance = Fetch.req(opts)
instance.then((res) => {

    console.log(res) 
})
```

*Advanced example*:


```js
var retryCount = 3
var opts = {
    url: 'http://httpstat.us/500',
    timeout: 2000,
    retry: (req) => {
        console.log(`retry attempt #${retryCount - req.retryCount + 1} ${req.url}`)
    },
    retryCount,
}

var instance = Fetch.req(opts)

// on successful response
instance.then((res) => {
    console.log('Success!', res)
})

// once retryCount reaches 0 and 
instance.catch((err) => {
    console.log(`${opts.url} could not be fetched: ${err.code}`)
})
```

<a name="schemaExample"></a>
## Schema mapping example

Think for a moment that you were dealing with an API that returned a list of products, and price:

<a name='dataSource1'></a>

```js
var data = {
    products: [
        {
            id: 'product1',
            name: 'product 1',
            description: 'the first product',
            price: 55
        }, 
        {
            id: 'product2',
            name: 'product 2',
            description: 'the second product',
            price: '66.50'
        },
        {
            id: 'product3',
            name: 'product 3',
            price: '$11.00'
        }
    ]
}
```

The API response above is not great, we have inconsitant fields which is common with NoSQL based data stores, we also have inconsistant typing of the `price` field across products.

If we were dealing with this API in the front end logic of our application, we would need to add a lot of bulk and complexity to be evaluated at runtime just to make sure the properties exist, and they are the type that we are expecting.
Not only does this bulk the application out, it makes it generally harder to read and scale for any developers being on-boarded.

Using QSAPI schema mapping, we can define a schema for how we want our dataretryCount -  to be structured, and typed:

<a name='schema1'></a>

```js
import SchemaMap from '../src/schema'
const {parse, type, _default, transform} = SchemaMap

var schema = {
    products: {
        id: {
            [type]: 'string'
        },

        name: {
            [type]: 'string'
        },

        description: {
            [_default]: 'N/a'
        },

        price: {
            [transform]: (price) => {
                return parseFloat(price.toString().replace('$', ''), 2).toFixed(2)
            }
        }
    }
}
```

The above schema defines a few things:

| Property | Description | Usage |
| -------- | ----------- | ----- |
| `type` | the `type` <a target="_blank" href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol">`Symbol`</a> | this is used to indicate to the schema mapping what the output type should be |
| `_default` | the `_default` <a target="_blank" href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol">`Symbol`</a> | the key's value is used if there is no data for this specific property |
| `transform` | the `transform` <a target="_blank" href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol">`Symbol`</a> | A function that gets evaluated, the first parameter is the data of the property being evaluated |

Using the <a href='#schema1'>schema</a> defined above, we can parse our <a href='#dataSource1'>data source</a>:

```js
// ...(continued from above)...

var mappedData = parse(data, schema)

/*
    mappedData.products:

    [
        { 
            id: 'product1',
            name: 'product 1',
            description: 'the first product',
            price: 55 
        },
        { 
            id: 'product2',
            name: 'product 2',
            description: 'the second product',
            price: 66.5 
        },
        { 
            id: 'product3',
            name: 'product 3',
            price: 11,
            description: 'N/a' 
        } 
    ]
*/
```

After the mapping has been applied, each field is consistant in type, and also has the same fields.
`description` was added to `product3`, `price` was transformed from being mixed type in the data to a `float` in the mapped data


<a name="api"></a>
# API
TODO

<a name="todo"></a>
# TODO
* [x] Schema mapping
* [ ] Schema type transformation
* [x] Fetch API
* [x] Fetch setup to allow for retries, timeouts, bailouts
* [x] Pre-fetch caching
* [ ] Post-fetch caching
