# Translate text to query format

In this endpoint you provide the described schema, and the query text, and the result will be the interpreted query (in the generalized format)

**URL** : `https://api.query.quantleaf.com/query`

**Method** : `POST`

**Auth required** : YES, provide `x-api-key=YOUR API KEY` in the header

**Permissions required** : None

**Body**

The schemas below are provided in Javascript format. 
- '|' means OR, 
- The value after the semicolon (:) denotes the type of the field.
- 'A[]' means an array of objects of type *A* 
- [key:A]:B is intepreted as a map, with a key of type A with the value of type B


```javascript
{
    "text": string, 
    "schemas" : Schema[] 
}
```
where 

*Schema*
```javascript
{
    "name": KeyWithDescriptions,
    "fields": Field[]
}
```

*Field*
```javascript
{
    "key": string
    "description": SimpleDescription // The translation
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

*SimpleDescription* can take 3 forms: 

- The description is provied as a map, where the key is the language code, and the value is either an array string array of descriptions or a single string representing one description

- Descriptions are provided as a list of strings. ANY language is assumed for all the descriptions of the array. 

- If one description is provided as a string, then ANY language is assumed for that description

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


**Body example** All fields must be sent.

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

**Condition** : If everything is OK and an Account didn't exist for this User.

**Code** : `200`

**Content example**

```json
{
    ""
}
```

## Error Responses

**Condition** : Not authorized, bad or non existing API key

**Code** : `401`

**Content** : `{}`

### Or

**Condition** : Bad request

**Code** : `400 BAD REQUEST`


