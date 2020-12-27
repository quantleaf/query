# Translate text to query format

In this endpoint you provide the described schema, and the query text, and the result will be the interpreted query (in the generalized format)

**URL** : `https://api.query.quantleaf.com/query`

**Method** : `POST`

**Auth required** : YES, provide `x-api-key=YOUR API KEY` in the header

**Permissions required** : None

**Data constraints**

Provied the query and your schemas.

```json
{
    "text": string, // This is the natural language search text
    "schemas" : Schema[] // Array of Schema objects
}
```

**Data example** All fields must be sent.

```json
{
    "text": 'Train ticket from Stockholm to London, // This is the natural language search text
    "schemas" : [{
        name: 'Train ticket',
        fields: [
            {
                key: 'from'
                description : 'From',
                type: ['Stockholm', 'London', 'Paris']
            },
            {
                key: 'to'
                description : 'To',
                type: ['Stockholm', 'London', 'Paris']
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


