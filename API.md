# Endpoint: Translate text to query format

With this endpoint you provide a query text, "described" schema and you will retrieve the interpreted query, in a generalized format.

**URL** : `https://api.query.quantleaf.com/translate`

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
    "query": {}
    "suggest": { "limit": number }
    "languageFilter": LanguageCode[]
    "fuzzy": boolean
    "concurrencySize": number,
    
}
```

*text* 

The query text.
**Limitation: max 200 characters.**

*schemas*

The schema objects, defining you database structure.


*query (Optional conditional)*

Query can either be an empty object '{}' or omitted. 
If omitted, then no translated query will be created.
If omitted *suggest* has to exist.

*suggest (Optional conditional)*

The suggest object lets you enable suggestions. If you provide a *limit* field, then the number suggestions will be limited to this limit. This option is preferred to use if your want to optimize for lowest possible latency, if you know in advance many suggestions you want.
If the *suggest* field is omitted, then no suggestions will be created.
If the *suggest* field is omitted then *query* field has to exist.



*languageFilter (Optional)*

Specify allowed languages. Default is all languages.

*fuzzy (Optional)*

If true, then we allow spelling erros of 25% amount. The spelling error can only occur on the end of the words. 

If false, no spelling errors allowed.

*concurrencySize (Optional)* 

This value indicates how many schemas can be searched at once. What this means is that if this value is *2* then all possible pairs of schemas are evaluated where common fields exist. These pairs are then treated as a new schema, which can be queried upon. If this value is *3* then all pairs and triplets are evaluated and so on.

Default value is 1 (no concurrency). A value of -1 indicates that the concurrency value is set to be equal to the amount of schemas (all possible schema combinations are found).

With a value of 1 you can still make multiple queries per query text, but one query can only refer to one schema.


**Overall limitations: Total number of fields has to be less than 250. If the *concurrencySize* is greater than 1 then you can not calcuate the total number of fields by summing the fields of each schema, instead write and perform a test request and see whether you schemas are compatible with this limit.**

> Note: Latency is highly dependent of the total amount of schema fields. Performance might potentielly improve by introducing language filters and/or set the *fuzzy* parameter to *false* as the probability of finding a query decreases. However keep in mind that having a large language space (cover multiple languages and handle spelling errors) is something that can truly enhance the quality of the services using this endpoint.

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
The supported language codes (ISO 639-1 standard).
 'ANY' language code is special and means any or all languages.

```javascript
'EN' | 'SV' | 'ANY' 
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
        'SV' : 'röd',
        'EN' : 'red'
    },
    'blue' : 
    {
        'SV' : 'blå',
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
    "text": "I want to buy a ticket for less than 100 bucks in 10 days from stockholm or gothenburg to", 
    "schemas" : [{
        "name": {
            "key": "airplane-ticket",
            "description": "Airplane ticket"
        },
        "fields": [
            {
                "key": "departureLocation",
                "description" : "From",
                "domain": {
                    "STOCKHOLM": "Stockholm",
                    "GOTHENBURG": 
                    {
                        "SV": "Göteborg",
                        "EN": "Gothenburg"
                    }
                }
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
                "domain": {
                    "LONDON": "London",
                    "PARIS" : "Paris"
                }
            },
            {
                "key": "departureDate",
                "description" : ["From","When","Date of departure"],
                "domain": "DATE"
            },
            {
                "key": "price",
                "description" : ["Price", "Pris"],
                "domain": "NUMBER"
            }
        ]
    }],
    "query": {},
    "suggest": { "limit": 10 }
}

```
This example describes a translation of a non-complete query text (the last word in the text is 'to').
The example has a schema with 5 fields, and as we include both *query* and *suggest* we will obtain the query translation as well as suggestions when performing this request. 

The fields *fuzzy*, *languageFilter* and *concurrencySize* have been omitted hence assumed to be the default values.

*See Example Response* for the result of this request.

## Success Response

**Condition** : Request processed without errors

**Code** : `200`

**Content**
### Entity (Request response)
```javascript
{   
    "query": QueryWithSchema[],
    "suggest": Suggestion:[],
    "unknown": Unknown[];
}
```

*query*

The translated queries

*suggest* 

The suggestions. Each element represent one suggestion. Suggestions are ordered in relevance (most relavent first). If two suggestions are equally relavent, they are ordered alphabetically.

*unknown*

The not understood parts of the query text. This field lets you create fallback behaviour if you notice that most of the query is not understood/interpreted. Some parts that you might think is obvious could for the Quantleaf Query API be unknown, for example 
'I want the price to be less than 10' will have two unknown sections 'I want the' and 'to be' (because the words does not provide any value).


---
### Entity *QueryWithSchema*
```javascript
{
        
    "from": string[], 
    "condition": ConditionCompare | ConditionAnd | ConditionOr
}
```
*from*

From what schema keys do this query originate from. This can be at most *concurrencySize* (property of the request) amount of keys. 

*condition*

The condition object contains all information about the conditions of the query

---

### Entity *ConditionCompare*
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

### Entity *ConditionAnd*
```javascript
{
    "and": (ConditionCompare | ConditionAnd | ConditionOr)[]
}
```


*and*

Each element of the array is 'and' conditional.


---

### Entity *ConditionOr*
```javascript
{
    "or": (ConditionCompare | ConditionAnd | ConditionOr)[]
}
```

*or*

Each element of the array is 'or' conditional.

---
### Entity *Suggestion*
Suggestion object defines a suggestion

```javascript
{
    "offset": number, 
    "text": string 
}
```
*offset*

Start index. This index is most likely to be at the end of your text. This index tells you were to place the suggestion to build the complete suggestion text

*text*

The suggestion. 

> Note: In order to build a complete suggestion you have to add the suggestion to your existing query text. For example if your query text is 'price' and one suggestion is
>```javascript
>{
>    "offset": 5
>    "text": ' less'
>}
>```
>Then you need to insert the text ' less' at index 5 in 'price', so that the complete (readable) suggestion becomes 'price less'

---


### Entity *Unknown*

Unkown text location
```javascript
{
    "offset": number, 
    "length": number 
}
```


*offset*

Start index.

*length*

The length of the unknown text.


> Note: The *Unkown* object describes the start and the end indices of the parts of the query text that has not been understood. For example 
>```javascript
>{
>    "offset": 0
>    "length": 1
>}
>```
>means that the first character is unknown.


## Example response
From the **Exampel request** this was the response (the API was at used 2021-01-08).
*The query text was "I want to buy a ticket for less than 100 bucks in 10 days from stockholm or gothenburg to"*

```javascript
{
    "query": [
        {
            "from": [
                "airplane-ticket"
            ],
            "condition": {
                "and": [
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
                    },
                    {
                        "compare": {
                            "key": "departureDate",
                            "eq": 1610991460000
                        }
                    },
                    {
                        "or": [
                            {
                                "compare": {
                                    "key": "departureLocation",
                                    "eq": "STOCKHOLM"
                                }
                            },
                            {
                                "compare": {
                                    "key": "departureLocation",
                                    "eq": "GOTHENBURG"
                                }
                            }
                        ]
                    }
                ]
            }
        }
    ],
    "unknown": [
        {
            "offset": 0,
            "length": 26
        },
        {
            "offset": 87,
            "length": 2
        }
    ],
    "suggest": [
        {
            "offset": 89,
            "text": " London"
        },
        {
            "offset": 89,
            "text": " Paris"
        },
        {
            "offset": 89,
            "text": " 123"
        },
        {
            "offset": 89,
            "text": " 2021-01-08"
        },
        {
            "offset": 89,
            "text": " equal"
        },
        {
            "offset": 89,
            "text": " Gothenburg"
        },
        {
            "offset": 89,
            "text": " SEK"
        },
        {
            "offset": 89,
            "text": " Stockholm"
        },
        {
            "offset": 89,
            "text": " USD"
        },
        {
            "offset": 89,
            "text": " Currency"
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


