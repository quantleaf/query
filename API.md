# Endpoint: Translate text to query format

In this endpoint you provide the described schema, and the query text, and the result will be the interpreted query (in the generalized format)

**URL** : `https://xxxxxxxxxxxxxxx`

**Method** : `POST`

**Auth required** : YES, provide `x-api-key=YOUR API KEY` in the header

**Permissions required** : None

**Body**

The schemas below are provided in Javascript format. 
- '|' means OR, 
- The value after the semicolon (:) denotes the type of the field.
- 'A[]' means an array of objects of type *A* 
- '{[key:A]:B}' is intepreted as a map, with a key of type A with the value of type B

>All **limitations** below are *soft limitations*. What this means is that they can and might be increased.

---



### Entity (the request object)
```javascript
{
    "text": string, 
    "schemas": Schema[] 
    "languageFilter": LanguageCode[]
    "fuzzy": boolean

}
```

*text* 

The query text.
**Limitation: max 1000 characters.**

*schemas*

The schema objects, defining you database structure.
**Limitation: Max 10 schemas.**

*languageFilter (Optional)*

Specify allowed languages. Default is all languages.

*fuzzy (Optional)*

If true, then we allow spelling erros of 25% amount. 
If false, no spelling errors allowed.

> Note: Latency is highly dependent of schemas and schema fields used. Performance might potentielly improve by introducing language filters and/or set the *fuzzy* parameter to *false* as the probability of finding a query decreases. However keep in mind that having a large language space (cover multiple languages and handle spelling errors) is something that can truly enhance the quality of the services using this endpoint.

---
### Entity *Schema*
```javascript
{
    "name": KeyWithDescriptions
    "fields": Field[] 
}
```
*name* 

The name of the schema. This lets the translation tool to understand if a user is mentioning a certain data type

*fields* 

The fields of the schema. 
**Limitation: Max 100 fields.** 
> Note: Latency is highly dependent of the number of fields used.

---
### Entity *Field*
```javascript
{
    "key": string
    "description": SimpleDescription 
    "domain": StandardDomainType | EnumDomain 
}
```
*key* 

The key, i.e. the symbol you denote the variable to in a programming context.

*description* 

The description of the field.

*domain*

The domain defines what values are allowed for this field.

---
### Entity *KeyWithDescriptions*
```javascript
{
    "key": string
    "description": SimpleDescription 
}
```

*key* 

The key, i.e. the symbol you denote the variable to in a programming context.

*description* 

The description.


---
### Entity *SimpleDescription*
```javascript
{ [key: LanguageCode]: string[] | string } | string[] | string
```
The *SimpleDescription* object is where the translation of your data is defined. It is important that you consider what languages and how many different types of words you want to support. 

*SimpleDescription* can take 3 forms: 

- The description is provied as a map, where the key is the language code, and the value is either an array string array of descriptions or a single string representing one description

- Descriptions are provided as a list of strings. ANY language is assumed for all the descriptions of the array. 

- If one description is provided as a string, then ANY language is assumed for that description

The Quantleaf Query API does not currently look at similiar word to word translations (but this something that is going to be implemented), which means you have to provide all the words that the user might write (and that you want to capture). This means that if you want to describe the color 'red' in english, you would want to write 'red', 'crimson', 'maroon' as descriptions.  
It is important that you spell right if you want an accurate translations. 

---
### Entity *LanguageCode* 
The supported language codes

```javascript
'EN' | 'SWE' | 'ANY' 
```

---
### Entity *StandardDomainType* 
The supported domains by type

```javascript
'DATE' | 'NUMBER' | 'TEXT'
```
---
### Entity *EnumDomain*

An enum domain is a map where each key represents the enum value key, and the value is a *SimpleDescription* which provides the translation for the key. This lets you control the value of the field by providing a list of feasible string values for a *enumValue* key. 

```javascript
[enumValue: string]: SimpleDescription
```

**Limitation: Max 100 enum value keys, 100 translations for each key for each language. One transation can max have 30 characters.** 


For example an *EnumDomain* could be 
```javascript
{
    'red' : {
        'SWE' : 'röd',
        'EN' : 'red'
    },
    'blue' : 
    {
        'SWE' : 'blå',
        'EN' : 'blue'
    }
}
```

or of you ignore providing language codes: 
```javascript
{
    'red' : ['red', 'röd']
    'blue' : ['blue', 'blå']
}
```


## Example request

```javascript
{
    "text": "I want to buy a train ticket from stockholm to london or paris 10 days from now for less than 100 bucks", 
    "schemas" : [{
        "name": {
            "key": "train-ticket",
            "description": "Train ticket"
        },
        "fields": [
            {
                "key": "from",
                "description" : "From",
                "domain": ["Stockholm", "London", "Paris"]
            },
            {
                "key": "currency",
                "description" : "Currency",
                "domain": {
                    "SEK": ["SEK","kronor"],
                    "USD": ["USD","dollar","bucks"]
                }
            },
            {
                "key": "to",
                "description" : "To",
                "domain": ["Stockholm", "London", "Paris"]
            },
            {
                "key": "from",
                "description" : ["From","When","Date of departure"],
                "domain": "DATE"
            },
            {
                "key": "price",
                "description" : ["Price", "pris"],
                "domain": "NUMBER"
            }
        ]
    }]
}
```
This example describes a schema with two fields, which is of enum type and have the same domain (city locations).
The key describes the value which the user use to denote the field.
In this case the API will understand that 
'from Stockholm' means thay the field we are interested in is with the key 'from', and that the value is Stockholm which is in the domain.

The fields *fuzzy* and *languageFilter* have been omitted hence assumed to be the default values.

## Success Response

**Condition** : Request processed without errors

**Code** : `200`

**Content**
### Entity (Request response)
```javascript
{   
    "queries": QueryWithSchema[],
    "unknown": Unknown[];
}
```

*queries*

All the translated queries

*unkown*

The not understood parts of the query text. This field lets you create fallback behaviour if you notice that most of the query is not understood/interpreted. Some parts that you might think is obvious could for the Quantleaf Query API be unkown, for example 
'I want the price to be less than 10 dollar' will have three unknown sections 'I want the' and 'to be' (because the words does not provide any value) and 'dollar' since the API currently does not support currencies associated directly with numeric values. 


---
### Entity *QueryWithSchema*
```javascript
{
        
    "from": string[], 
    "query": QueryCompare | QueryAnd | QueryOr
}
```
*from*

From what schema keys do this query originate from. This can be multiple keys since if your provide multiple schemas and if the fields *overlap* the same query could be feasible for different schemas at once.

*query*

The query object contains all information about the query

---

### Entity *QueryCompare*
```javascript
{
    "compare": Compare
}
```

---

### Entity *Compare* 
```javascript 
{
    "key": string,
    "lt": number,
    "lte": number,
    "gt": number,
    "gte": number,
    "eq": string | number
} 

```


*key*

Will always exist.

*lt*

Less than.

*lte*

Less than or equal to.


*gt*

Greater than.

*gte*

Greater than or equal to.

*eq*

Equal to.

 > Note: *key* will alway exist, but only one of the other properties can exist
    Dates are represented with a number which represents milliseconds since Unix epoch.




---

### Entity *QueryAnd*
```javascript
{
    "and": (QueryCompare | QueryAnd | QueryOr)[]
}
```


*and*

Each element of the array is 'and' conditional.


---

### Entity *QueryOr*
```javascript
{
    "or": (QueryCompare | QueryAnd | QueryOr)[]
}
```

*or*

Each element of the array is 'or' conditional.

---

### Entity *Unknown*

Unkown text location
```javascript
{
    "start": number, 
    "end": number 
}
```


*start*

Start index (including)

*end*

End index (excluding)


> Note: The *Unkown* object describes the start and the end indices of the parts of the query text that has not been understood. For example 
>```javascript
>{
>    "start": 0
>    "end": 1
>}
>```
>means that the first character is unknown.


## Example response
From the **Exampel Request** this was the response (the API was at used 2021-01-03. The request took 310 ms to perform).
*The query text was "I want to buy a train ticket from stockholm to london or paris 10 days from now for less than 100 bucks"*

```javascript
{
    "queries": [
        {
            "from": [
                "train-ticket"
            ],
            "query": {
                "and": [
                    {
                        "compare": {
                            "key": "from",
                            "eq": "Stockholm"
                        }
                    },
                    {
                        "or": [
                            {
                                "compare": {
                                    "key": "to",
                                    "eq": "London"
                                }
                            },
                            {
                                "compare": {
                                    "key": "to",
                                    "eq": "Paris"
                                }
                            }
                        ]
                    },
                    {
                        "compare": {
                            "key": "from",
                            "eq": 1610562689000
                        }
                    },
                    {
                        "compare": {
                            "key": "price",
                            "lt": 100
                        }
                    },
                    {
                        "compare": {
                            "key": "currency",
                            "eq": "USD"
                        }
                    }
                ]
            }
        }
    ],
    "unknown": [
        {
            "start": 0,
            "end": 15
        },
        {
            "start": 80,
            "end": 83
        }
    ]
}
```


---
## Error Responses

**Condition** : Not authorized, bad or non existing API key

**Code** : `401`

**Content** : `{}`

### Or

**Condition** : Bad request, too many schemas or fields per schema, or too long query text.

**Code** : `400 BAD REQUEST`


