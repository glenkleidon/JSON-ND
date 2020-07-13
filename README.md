# JSON-ND Format Specification V1.0

__** -- DRAFT -- **__

JSON-ND or _**JSON with Named Datatypes**_ is a very simple way to include a Data-Type in [JSON](https://json.org) (_the JSON Post_) data as described in The JSON Data Interchange Syntax standard [ECMA-ST-ECMA-404](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf) (_the JSON standard_).

The technique uses the "name" element of JSON value pairs in the specification to convey the data-type.

_The JSON standard_ ECMA-ST-ECMA-404, paragraph 6, states that: 
```
A name is _string._ The JSON syntax does not impose any restrictions on the strings used as names, does not require that name strings be unique, and does not assign any significance to the ordering of name/value pairs...
```
This removes the previously ambigous term "name" which, as a result of an omission of a definition of the term name in 2 of 3 parts of _the JSON Post_.  The term "name" was described only in the text of _the JSON Post_, but not included in the McKeeman Form or Workflow diagrams of the original post. 

This means, by agreed convention, it is possible to remove the ambiguity of the intended data type by **appending the data-type to the element name**.

For example the potentially ambigous JSON object below: 
```
{ "name:string": "Alice", "isActive": 0 }
```
can be unambigously qualified as:

```
{ "name:string": "Alice", "isActive:boolean": 0  }
```
Both forms of this JSON object is valid JSON in most, if not all, modern browser Javascript implementations.

This fact allows a very simple and efficient means to fully qualify the data type in JSON.  

## Introduction

This specification describes the semantics required to efficiently _**qualify data types for JSON elements**_, to _**define complex-data types**_ and provides a means of _**defining Remote Method Calls**_ using JSON. 

The primary purpose is to **reduce or eleminate ambiguity** of JSON values by **including type information** in the JSON string and/or values.  This is most useful when using JSON to communicate between strongly typed and untyped language implementations of a service; or with a strongly typed language using a dynamic data structure where the exact data type may not be fully known at compile time. 

While this can be done easily using multiple JSON elements for each piece of data, 
```
{"name":{ "type": "string", "data": "Alice"}}
```
this can be moderately inefficient often more than doubling the message length; or by having a pre-processing section in the message header.
```
{ 
  "types": {
    "name": "string"
  },
  "name": "Alice"
}
```
which is generally more efficient than the first form, especially for repeating elements.

The JSON-ND form represents the same element as 
```
{ "name:string" : "Alice" }
```
which is smaller, and may have lower memory overhead and parsing advantages over the previous two implementations.

The JSON-ND specification also allows for unambiguous elements to be un-qualified, so the form
```
{ "name":"Alice", "isActive": false, "amount:currency": 32 } 
```
is valid JSON-ND.  The JSON-ND specification states that 

**_All JSON is valid JSON-ND, and all JSON-ND_** **MUST BE** **_valid JSON._**

In order to handle JSON Arrays of mixed type, the JSON-ND specification also specifies a convention for encoding mixed type JSon Array values by appending the data type to value and specifying the type of **_ndMixedType[]_** data-type. For example: 
```
{
  "stuff:ndMixedType[]" : [
    "Alice:string",
    true,
    "1:currency",
    ""To be\u003A Or not to be:string"
  ]
}
```
The JSON-ND specification states that the data-type qualifier is **_always_** optional, even for mixed type Json Arrays. This means that two services need to know if the messages are JSON-ND or simply JSON to ensure the destinction can be made. 

the HTTP Header "Content-Type" is generally sufficient for this in most cases.  The specification defines the Content Type as  Where the transport layer between two services is NOT HTTP, the services consuming messages 


## Purpose

When working with dynamic languages (or with dynamic structures in typed languages), it is often necessary to check what data type a incoming element is in order to work with it correctly. 

For example, the content of the following C# class is likely to loose precision when sent to or received from, a remote service. It may give the wrong result or cause a data type incompatibilty exception.

This is particularly the case with number types where the precision may be lost during the serialization of the element. Currency is generally ambigous as there is no indicator of the precision of a JSON number. There are also some implementation (particularly legacy systems) where a dynamically typed custom structure may not have a modern equivalent (eg DCOM implementations which ) 

```
public class MyClass 
{
  public bool isAllowed;
  public decimal cost = 0m;
  value float value = 0;
}
```

The following JSON could be data in a transaction with this class:
```
{
    "isAllowed": 1,
    "cost": 54,
    value: 28.0000002
}
```

The JSON notation does not provide sufficient information for (say) a Python application to correctly infer the required type and therfore the correct overload of a method to use with the received data.

In strongly typed languages, the violation of the float range in this example may cause an incompatible data type exception in a .Net consumer.

A dynamically typed languge like Javascript may also create objects with properties of the wrong type and cause exceptions when methods are executed.

## General Principles of the JSON-ND defintion 

To translate JSON to JSON-ND, (optionally) **append the data-type to the end of the element name or value**.

 The term **"data-type"** is completely open and it should be interpreted to mean: 

_**Text that describes how the associated value should be interpreted by the intended consumer**_. 

In order to qualify what type is being used, an optional language style can be specified in the form of a JSON-ND object:

"Json-ND" :{"version": 1.0,"style": "_**language**_"[, data:{}]}

JSON-ND can be passed without the style element where entities within a JSON transaction assume a pre-arranged language style.
```
{
    "id:UInt32": 345,
    "name:string": "Bob",
    "cost:currency": 1400
}
```

In cases where there is no pre-arranged style, the style qualifier can be added as an additional element.

```
{
    "Json-ND": {"version": 1.0,"style": "pascal"}
    "id:UInt32": 345,
    "name:string": "Bob",
    "cost:currency": 1400
}
```

A fully qualified JSON-ND Object can also be used:

```
{
    "Json-ND": {
        "version": 1.0,
        "style": "pascal",
        "data": {
            "id:UInt32": 345,
            "name:string": "Bob",
            "cost:currency": 1400
        }
    }
}
```

In the case where the consumer may not be know (public APIs), multiple comsumer language versions of the definition can be generated depending on the intended consumer of the definition (eg C++, C#, Pascal, GO).

The alternative is to use a "default" language specification such as [XSD Built-in Types](https://www.w3.org/TR/2004/REC-xmlschema-2-20041028/datatypes.html#built-in-datatypes) 
```
"Json-ND" :{"version": 1.0,"style": "http://www.w3.org/2001/XMLSchema-datatypes"}
``` 


or [Typescript](https://www.typescriptlang.org/) 
```
"Json-ND" :{"version": 1.0,"style": "typescript"}
``` 

Unfortunately, there are limitations to both of these language styles - XSD provides very good reference for data types, but overly verbose methods for defining remote procedure calls, and Typescript (as it current stands) still has a limited list of primitive data types.

As there is likely to be no easy consensus on what the default language should be, this specification does not define a default language style.

### Element Example
When using JSON-ND then following Standard JSON
```
{"id: 23, "name":"Alice", "results": [1,6,5]}
```
becomes:
```
{"id:integer": 23, "name:string":"Alice", "results:integer[]":[1,6,5]}
```
_Note: JSON-ND encoding is always valid JSON and all JSON is valid JSON-ND.  The name portion of JSON Element (as defined by json.org)
does not exclude any specific characters: it is simply defined as a "string".  Browser based interpretters consider JSON-ND element names valid although there are implications for referencing them without pre-processing._

For Typescript, any ambiguous data-types can be qualified with a regular expression. For the 


{}

###  Mixed Array type Example
Although mostly avoidable for standard array types, the follow format for mixed type JSON arrays can be used.

The Standard JSON array
```
[23, 1, "$22.04"]
```
becomes
```
["23:decimal", "1:boolean", "22.04:currency"]
```

## More Complex data types

With this simple convention, there is sufficient syntax to define **custom data types** and define **remote methods**.

This approach is far simpler, (but also less complete) than the [JSON Schema](https://json-schema.org/) approach because

+ the data and definition are not separated
+ and no prior knowledge is required by a service to use the definition. 

### Enumerated type Example
Here is an example enumerated type
```
{ 
  "RoleType:enum" :[
    "admin:1",
    "accounts",
    "sales",
    "service"
  ]
}
```
### Complex Object type Example using defined type
Yiou can create complex types utilizing pre-defined types
```
{
  "User:Interface": [
        "name:string",
        "id:int",
        "roles:RoleTypes[0,]"
    ]
}
```



### Remote Method Examples
The following represents a JSON-ND method defintion -in this case Pascal Style.  The may be different specification for different consumers of the data.

```
{
  "SquareRoot:function(value:integer):integer" : "https://some.server.com:999/sqrt" 
}
```

Any language style is permitted and is in no way restricted or defined in this specification. The method is unabiguous as you know not to call this method with floating point values.  Again depending on the consumer, you may need to qualify the datatype allowed using a language specific construct (eg with RegularExpression for Javascript) to enforce non-floating point restriction.


## The JSON-ND Specification

### 1. Qualifying The Data-type of a Name Value Pair
To encode data using JSON ND when using Name/Value pairs, append a colon character (":") and the data-type after **Element Name** within the JSON string.

**{"Name:_data-type_**[**_[\<lowerlimit\>_**[**_,\<upperlimit\>_**]**_]_**]**" : value}**

 It is OPTIONAL to use a qualifier. Defining an array lower limit and upper limit are optional.  

### 2. Qualifying The Data-type of Value Elements for Mixed Array Types
For **Value elements** contained within a mixed type JSON Array, optionally convert the data element to a JSON string, then     append a colon character (":") and the data-type. 
**["Value:_data-type_**[**_[\<lowerlimit\>_**[**_,\<upperlimit\>_**]**_]_**]**"]**

 + This method MUST NOT be used for Object type values.
 + The string value SHOULD escape any  (as per the JSON specification using UNICODE character \u003A), followed by the colon character (":") then the data-type.
 + see notes on use of colons in value data type definitions below  

### 3. Defining Complex Types as Interfaces
Complex data types may be defined by appending the **_Reserved KeyWord_** `Interface` to the name of a JSON Element.
There are two forms:

#### Complex Type: Array Form
      {
        "<Type name>:Interface" : [
          "<Property Name>:<data-type>",
          ...
        ]
      }

#### Complex Type: Object form

      {
        "<Type name>:Interface" {
          "<Property Name>": "<data-type>,
          ...
        }
      }

Implementations MUST support complex types of any nested depth and or complexity utilising any type **already encountered by the parser** or any implied type for the consumer's language.  Implementations MUST support data-types **yet to be encountered by the parser** if the consumer language supports that feature.



### 4. Defining Type Aliases and Constants
eg `["pi:3.14159"]`

### 4. Defining Remote Methods Calls

### 5. Defining Constants

A data-type MAY BE a literal value - essentially to define a constant.

When intent is not clear use data types, otherwise data type is optional 

```
[
    "id" : 0,
    "Name" : "Alice";
    "isLoggedIn:boolean" : 0
    "amount:currency": 0.01,
    "results:integer": [23,42,22]
    "numbers": [
       "1:boolean",
       "234:decimal"
       "54:currency"
       ]
]
``` 

It is expected that _more specific_ definitions for specific purposes (eg "JSON-ND for C Languages") will be defined in the future as extensions of this specification.

## Use of Colons in Data-types
Implementations of JSON-ND should only consider the first occurrence of the colon (":") character. Subsequent colons MAY or MAY NOT be escaped as desired and MUST BE considered as data, and NOT a delimiter.

The purpose of interpreting subsequent colons as data is to facilitate data-types that may include the colon character for its own purpose.  For example C++ uses as double colon ("::") as a namespace qualifier. This approach also allows the data-type to describe a method, its parameters and return type for the purposes of RPC calls.

In the case where multiple colons exist in the string and NO colons have been escaped (to improve human readability), the FINAL colon is TO BE interpretted as the datatype (ie simple datatype name).  This is a particular problem for URI's so you MUST include the data-type othewise the interpreter will be confused. eg

**["https://wwww.some.site.com:99:URI"]** is to be interpretted as  "https://www.some.site.com:99" as type "URI"
**["https://wwww.some.site.com:99"]** however will be interpretted as  "https://www.some.site.com" as type "99"


While using characters such as colons, braces and brackets in the name is valid JSON, the practice makes the object name inaccessible after a JSON.parse() call in Javascript.  This is fortunate and convenient as it forces the script to validate the received object before any qualified values are (easily) accessible to javascript code.  A very basic validator implementation is included as an example below.
[demo](https://www.galkam.com.au/public/testjson-nd.html)
 

## Data-type structure and style
This specification **DOES NOT** define any specific data types except for the **Interface** data-type.  Any programming language style can be used to specify the data type.  

In order to ensure the parsing system knows how to deal with the data-types, the following qualifying element **CAN BE** prefixed to the JSON message:

"Json-ND" :{"version": 1.0,"style": "_**language**_"}

where **language** refers to a specific language or specification used within the JSON document.


The _**default language style**_ is defined as "_**Typescript**_", and when no style is specified, TypeScript MUST BE assumed.  Again the exact specification for "JSON-ND for Typescript" is out of scope and may be defined elsewhere.

There are strong arguments for selecting **TypeScript** to be the default language.  However the limitation of this is that **Number** data type is often the reason the type qualification is required.  

## Defining Complex Types - the _Interface_ data-type
A complex type can be defined using an array of values or objects.  The **Interface** data-type is reserved for this purpose.  It is defined in the following way:

{
"data-type name: **interface**" : "Name:data-type"}

**OR** 

{"data-type name: **interface**" : [

  "elementName1:data-type" **OR** {"elementName1:data-type":"value" **OR** [] },
  
  "elementName2:data-type" **OR** {"elementName2:data-type":"value" **OR** [] },

  ...

  "elementName'n':data-type" **OR** {"elementName'n':data-type":"value" **OR** [] }
  
  ] 
} 

The Element Name MAY BE empty.

## JSON-ND Mime Type
The following MIME Type (to be used in html headers) is defined to be _**application/json+nd**_ and the preferred file extension is _**.jsonnd**_. 

The use of this mime type is preferred however, as JSON-ND is always valid JSON, the standard JSON mime type (_application/json_) and the _.json_ file extension are acceptable.

### The HTTP _**content-type**_ and _**accept**_ Headers
For HTTP protocol responses, as per 
 [rfc1341](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html), the mime type to use as the JSON-ND mimetype as defined above 
 
 `application/json+nd`

Consideration should also be given to using the _**Parameter**_ option with the "style" attribute to define the language style.

 Request Headers:
 `accept: application/json+nd[; style=`_**language**_`]`

 Response Header:
 `content-type: application/json+nd[; style=`_**language**_`]`
 

## JSON-ND Examples:

### Element Names
```
{ "useFactor" : 1 }
```

Becomes:
```
{ "useFactor:boolean" : 1 }
```


### Value Elements in JSON Array
```
[
"T520",
"H555", 
"To be: Or not to be",
1,
true,
0
]
```

Becomes:
```
[
"T520:char[4]",
"T520:char[4]",
"To be\u003A Or not to be:string",
"1:single",
"true:boolean",
"0:currency"
]

```

### Programming Language Styles (suggestion only)
#### Typescript style:
```
  "count:BigInteger" : 1,
  "age:number" : 27.3,
  "cost:
  "arrivalTime:Date" : "15:23:02",
  "dollarAmount:number" : 200.33,
  "arrayOfInt:number[]" : [1,2,3,4,5,6]
  "twoDArray:any[2][2]" : [ ["one","two"], ["three","four"] ]
}
```

#### C++ style:
```
{ 
  ["JsonND" :{"version": 1.0,"style": "C++"},
  "count:short int" : 1,
  "age:float" : 27.3,
  "arrivalTime:std::tm" : "15:23:02",
  "dollarAmount:long double" : 200.33,
  "arrayOfInt:int[]" : [1,2,3,4,5,6]
  "twoDArray:string[2][2]" : [ ["0,0","0,1"], ["1,0","1,1"] ]
}
```
#### C# style:
```
{ 
  ["JsonND" :{"version": 1.0,"style": "C#"},
  "count:int" : 1,
  "age:Single;f" : 27.3,
  "arrivalTime:DateTime" : "15:23:02",
  "dollarAmount:Decimal;m" : 200.33,
  "arrayOfInt:int[]" : [1,2,3,4,5,6]
  "twoDArray:string[2,2]" : [ ["one","two"], ["three","four"] ]
}
```
#### Pascal style:
```
{ 
  ["JsonND" :{"version": 1.0,"style": "Pascal"},
  "count:integer" : 1,
  "age:single" : 27.3,
  "arrivalTime:datetime" : "15:23:02",
  "dollarAmount:currency" : 200.33,
  "arrayOfInt:array of integer" : [1,2,3,4,5,6]
  "twoDArray:array[0..1][0..1] of string" : [ ["one","two"], ["three","four"] ]
}
```

### Methods 

#### Method Types
Given a method **GetUser** with parameters "id" (integer) and "options" (UserEnum) which returns a "User" object, 
the data COULD BE encoded as follows:
```
  { "GetUser:function(int id, UserEnum options):User" : "https://userserver/api/getuser" }
```
Remember that this specification is not specific about the programming language style. 
The example above is in a C style, but could also be encoded in a pascal style
```
  { "GetUser:function(Id:integer;Options:UserEnum):User" : "https://userserver/api/getuser" }
```
or REST API call
```
  { "GetUser:GET(Id:integer;Options:UserEnum):User" : "https://userserver/api/getuser" }
```
#### Return Types

Return types are implied as n-dimensional arrays of the expected return type, so if no array length is defined, n-length with
zero base is implied. Eg the following are equivalent
```
  { "GetUser:GET(Id:integer;Options:UserEnum):User" : "https://userserver/api/getuser" }
```
```
  { "GetUser:GET(Id:integer;Options:UserEnum):User[0,]" : "https://userserver/api/getuser" }
```

### Interface Examples

#### Single Value (type aliasing)
C Style
```
{"PInteger:Interface": ":*int"}
```
Pascal Style
```
{"PInteger:Interface": ":^integer"}

{"HResult:Interface": ":Int32"}
```
#### Enumerated types.
```
{ 
  "RoleType:enum" :[
    "admin:1",
    "accounts",
    "sales",
    "service"
  ]
}
```
#### Simple Objects
There are two acceptable forms for defining Objects:

The slightly more standard and intuitive, but more verbose version:

```
{"HResult:Interface": ":Int32"}
{
  "User:Interface": [
      {"name":"string"},
      {"id":"int"},
      {"roles" :"RoleTypes[0,]"}
  ]
}
```

OR, as it has been defined here, the following which form


The latter may actually more easily interpretted in the wild as it is already int the form used for processing.



#### Class Data Types 
If follows that a class type COULD BE defined using the interface above.

The approach for defining data classes is not in the scope of this document.
The recommended approach for defining classes using JSON-ND MAY BE described elsewhere. The example below gives the concept some consideration, but should not be taken as a part of the specification.

A class might include:
+ Private, Public, Protected and Published Members
+ Methods (and Class Methods)

First define a Class interface:
```
{ "Class:Interface": 
  [
  "private:any[0,]",
  "protected:any[0,]",
  "public:any[0,]",
  "published:any[0,]"
]
}
```

Now define the User Class (using the User type and GetUser method type examples)
```
{
  "UserClass:Class" : [
    "private" : [
             "_user:User",
             "getUser:GetUser"
    ],
    "public" : [
             {"user:User as Property": ["get:getUser","set:null"],
              "HasRole:function(role:RoleType):boolean"
    ]
  ]
}
```
#### Javascript JSON-ND Validation
Below is a very simple JSON-ND parser/validator. [raw source](https://raw.githubusercontent.com/glenkleidon/JSON-ND/master/testJson-nd.html)
```
<html>
<head>
<script>
  function JSON_ND_Parse(ndObj)
  {
    var obj = JSON.parse(ndObj);     
    for (var propertyName in obj) {
        var p = propertyName.indexOf(":");
        var v = obj[propertyName];
        if (p > 0) {
            if (propertyName.substr(p+1,3)==='int' && !parseInt(v)) 
            {
                v=NaN;
            }; // add more validation.
            obj[propertyName.substr(0, p)] = v;
            delete(obj[propertyName]);
        }
    };
    return obj;
  }
</script>
</head>
<body>
  <script>
    var nd = '{"abc:string":"abc contains a string", "def:integer" : "not a number"}'; 
    document.write("<h3>Original 'nd' object:<pre>",nd,"<pre></h3>");  
    document.write("See that 'nd.abc' has no value: abc=<b>" ,nd.abc,"</b>");
    document.write("<h3>Then validate with JSON_ND_Parse:<pre>",JSON.stringify(JSON_ND_Parse(nd)),"</pre></h3>");
    document.write("<p style='color: red'>Note that 'def' failed validation</p>");
  </script>
</body>    
</html>
```

