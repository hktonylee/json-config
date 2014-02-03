json-config
===========

`JSONConfig` is a JSON config reader. You can write migration plan and read old config easily.

Example
-------

Config Version 1 (`user.json`)

```json
{
    "id": 5,
    "full_name": "Harmony Kim",
    "age": 26,
}
```

Config Version 2 (`user-v2.json`)

```json
{
    "id": "5",
    "version": 2,
    "first_name": "Harmony",
    "last_name": "Kim",
    "age": 26
}
```

Config Version 3 (`user-v3.json`)

```json
{
    "id": "5",
    "version": 3,
    "first_name": "Harmony",
    "last_name": "Kim",
    "email": "some@email.com"
}
```


Migration Plan `migration.js`

```javascript
[
    {
        from: null,
        to: {version: 2},
        mapping: {
            "id": T.Convert("id", "string"),
            "first_name": T.Rename("full_name"),
            "last_name": null
        }
    },
    {
        from: {version: 2},
        to: {version: 3},
        mapping: {
            "email": null,    // for old config, set to null as default value
            "age": T.Drop(),  // remove the key-value pair
        }
    }
]
```

Then you can load the config in latest format no matter what version the config is.

```objectivec
// Objective-C
NSDictionary *userConfig = [JCReader dictionaryWithJSONPath:[NSBundle pathForResource:"user" ofType:@"json"] 
                                          migrationPlanPath:[NSBundle pathForResource:"migration" ofType:@"js"]];
```

Features
========

* It supports default value for newly added key-value pair.

    ```js
    {
        from: {"version": 2},
        to: {"version": 3},
        mapping: {
            "new_key": "any default value you like"
        }
    }
    ```
   
* It supports inline transformation in Javascript.

	```js
	{
        from: {"version": 2},
        to: {"version": 3},
        mapping: {
            ""new_name": function (o) {
            	return "name_" + o.toUpperCase();
            }
        }
    }
    ```

* It supports array migration.

	```js
	{
        from: {"version": 2},
        to: {"version": 3},
        mapping: {
            "emails": T.Convert("email", "array")
        }
    }
    ```

* You can add `version` field in later version. So that you can use this library on older projects.

	```js
	{
        from: null,
        to: {"version": 3},
        mapping: {
            // ...
        }
    }
    ```

* You can use different method to determine the version.

	```js
	{
	        from: {"v": "2"},
        to: {"version": 3}
    }
    ```

	or even...
	
	```js
	{
		from: {"v1": function (json) { return "full_name" in json; },
		to: {"version": 2}
	}
	```

* Multilingual ;-)

	```js
	// Objective-C
	NSDictionary *newJSONConfig = [JCReader dictionaryWithJSONPath:[NSBundle pathForResource:"user" ofType:@"json"] 
	                                             migrationPlanPath:[NSBundle pathForResource:"migration" ofType:@"js"]];

	// Javascript
	var newJSON = JCReader.read(oldJSONContent, migrationPlan);
	```


* It supports identity transform (do nothing).

	```js
	{
        from: {"version": 2},
        to: {"version": 3},
    }
    ```

* It can prevent the new config from inheriting old value.

	```js
	{
		from: {"version": 2},
        to: {"version": 3},
        inheritItems: false,   // default: true
        mapping: {
            // ...
        }
    }
    ```

* It supports nested array migration.

	```json
	// old example.json
	[
		{
			"name": "Thomas",
			"borrowed_movie_list": [
				{
					"id": 5311,
					"name": "500 Days of Summer"
				},
				{
					"id": 8812,
					"name": "Inception"
				}
			]
		}
	]
	```
	
	```json
	// new example.json
	[
		{
			"name": "Thomas",
			"borrowed_items": [
				{
					"id": 5311,
					"title": "500 Days of Summer",
					"type": "movie"
				},
				{
					"id": 8812,
					"title": "Inception"
					"type": "movie"
				}
			]
		}
	]
	```
	
	```js
	// migration.js
	{
		from: {"version": 2},
        to: {"version": 3},
        mapping: {
            "*.borrowed_items": T.Nested("*.borrowed_movie_list", {
            	"*.type": "movie",
            	"*.title": T.Rename("*.name")
            })
        }
    }
    ```


TODO
====

- implement everything ;-)






