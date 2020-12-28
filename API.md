# Translate text to query format

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


The object to send is

```javascript
{
    "text": string, // The search/query text
    "schemas": Schema[] // The schemas that describe your database
}
```
where 

*Schema*
```javascript
{
    "name": KeyWithDescriptions, // The description of the schema name
    "fields": Field[] 
}
```

*Field*
```javascript
{
    "key": string
    "description": SimpleDescription // The description of the field 
    "domain": StandardDomainType | EnumDomain // Defines possible values
}
```

*KeyWithDescriptions*
```javascript
{
    "key": string
    "description": SimpleDescription // The translation
}
```

*SimpleDescription*
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

*LanguageCode* 
```javascript
'EN' | 'SWE' | 'ANY' 
```


*StandardDomainType* 
```javascript
'DATE' | 'NUMBER' | 'STRING'
```

*EnumDomain*
```javascript
[enumValue: string]: SimpleDescription
```
An enum domain is a map where each key represents the enum value key, and the value is a *SimpleDescription* which provides the translation for the key. This lets you control the value of the field by providing a list of feasible string values for a *enumValue* key. 

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


**Body example** 

```javascript
{
    "text": "Train ticket from Stockholm to London", 
    "schemas" : [{
        "name": "Train ticket",
        "fields": [
            {
                "key": "from"
                "description" : "From",
                "domain": ["Stockholm", "London", "Paris"]
            },
            {
                "key": "to",
                "description" : "To",
                "domain": ["Stockholm", "London", "Paris"]
            }
        ]
    }]
}
```

## Success Response

**Condition** : Request processed without errors

**Code** : `200`

**Content**
```javascript
{   
    "queries": QueryWithSchema[],
    "unknown": Unknown[];
}
```
where

*QueryWithSchema*
```javascript
{
        
    "from": string[], // From what schema keys do this query originate from
    "query": QueryCompare | QueryAnd | QueryOr
}
```

*QueryCompare*
```javascript
{
    "compare": Compare
}
```

*Compare* 
```javascript 
{
    "key": string,
    "lt": number,
    "lte": number,
    "gt": number,
    "gte": number,
    "eq": string|number
} 
```
*key* will always exist. Only one of the other properties can exist (the other ones must be null)

*QueryAnd*
```javascript
{
    "and": (QueryCompare | QueryAnd | QueryOr)[]
}
```
*QueryOr*
```javascript
{
    "or": (QueryCompare | QueryAnd | QueryOr)[]
}
```

*Unknown*
```javascript
{
    "start": number, // Start index (including)
    "end": number // End index (excluding)
}
```
The *Unkown* object describes the start and the end indices of the parts of the query text that has not been understood. 
For example 
```javascript
{
    "start": 0
    "end": 1
}
```
means that the first character is unkown.


## Error Responses

**Condition** : Not authorized, bad or non existing API key

**Code** : `401`

**Content** : `{}`

### Or

**Condition** : Bad request

**Code** : `400 BAD REQUEST`


