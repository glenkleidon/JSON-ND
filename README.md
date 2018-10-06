# JSON-ND Format Specification V1.0

JSON-ND or _**JSON with Named Datatypes**_ is a very simple extension to the [JSON format](https://json.org).  
The specification describes a way to define a _specific data-type_ for JSON elements and also allows for _complex-data types be be defined_ as interfaces.

JSON-ND encoding is always valid JSON and all JSON is valid JSON-ND.    

Inspired by [TypeScript](https://www.typescriptlang.org) notation, all that is required is that _**the data-type is appended to the end of a JSON string**_.

## Using JSON-ND

To encode data using JSON ND when using Name/Value pairs, the **Element Name** CAN HAVE the data type appended after the name, with
a preceding colon character (":").

**"Name:_datatype_"**

For example:

	{ "factor" : 1 }
	
Becomes:

	{ "factor:boolean" : 1 }

For **Value elements** contained within a JSON Array, a JSON string is used.  The data element is encoded as a string with any literal colons escaped as
per the JSON specification (UNICODE character \u003A), followed by the Colon Character (":") and the data type. eg

For Example:
```
	[
	"T520",
	"H555", 
	"To be: or not to be",
	1,
	true
	]
```
	
Becomes:
```
	[
	"T520:char[4]",
	"T520:char[4]",
	"To be\u003A or not to be:string",
	"1:single",
	"true:boolean"
	]
	
```

## Use of Colons in Data-types
Implementations of JSON-ND should only consider the first occurrence of the Colon (":") character. Subsequent colons MAY or MAY NOT be escaped as desired. 

This is to facilitate data-types that may include the colon character for its own purpose.  For example C++ uses the colon character as a namespace qualifier.

This approach also allows the data-type to describe a method, its parameters and return type for the purposes of RPC calls.


## Data-type structure and style
This specification **DOES NOT** define any specific data types except for the **Interface** data-type.  Any programming language style can be used
to specify the data type.

In order to ensure the parsing system knows how to deal with the data-types, the following qualifying element **CAN BE** prefixed to the JSON message:

"JSON-ND" :{"version": 1.0, "style": "_**language**_"}

For example, using C++ style:
```
	{ 
	  ["JSON-ND": {"version": 1.0, "style": "C++"}},
	  "count:short int" : 1,
	  "age:float" : 27.3,
	  "arrivalTime:std::tm" : "15:23:02",
	  "dollarAmount:long double" : 200.33,
	  "arrayOfInt:int[]" : [1,2,3,4,5,6]
	  "twoDArray:string[2,2]" : [ ["0,0","0,1"], ["1,0","1,1"] ]
	}
```	
Using Pascal style:
```
	{ 
	  ["JSON-ND": {"version": 1.0, "style": "Pascal"}},
	  "count:integer" : 1,
	  "age:single" : 27.3,
	  "arrivalTime:datetime" : "15:23:02",
	  "dollarAmount:currency" : 200.33,
	  "arrayOfInt:array of integer" : [1,2,3,4,5,6]
	  "twoDArray:array[0..1][0..1] of string" : [ ["0,0","0,1"], ["1,0","1,1"] ]
	}
```


## Defining Complex Types - the _Interface_ data-type

A complex type can be defined using the "Interface" data-type in the following way:

"name**:Interface**" : [
  "elementName1:data-type",
  ...
  "elementName'n':data-type"
  ]
  

### Object Class Data Types

If follows that an object or class type can be defined using the interface above.

A class might include:
	+ Private, Public, Protected and Published Members
	+ Methods (and Class Methods)

The approach for defining data classes is not in the scope of this document, however the recommended approach for defining 
classes using JSON-ND may be described elsewhere.











