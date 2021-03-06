# The JSON-ND Specification

## Abstract
JSON-ND or _**JSON with Named Datatypes**_ is a very simple way to include data-types in [JSON](https://json.org) data. The JSON Syntax is described in The JSON Data Interchange Syntax standard [ECMA-ST-ECMA-404](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf) (_the JSON standard_).

The **JSON-ND Format Specification** (_the specification_) uses the "name" element of JSON value pairs to convey the data-type.

_The JSON standard_ ECMA-ST-ECMA-404, paragraph 6, in conjunction with [RFC8259](https://tools.ietf.org/html/rfc8259) state that: 
```
A name is _string._ The JSON syntax does not impose any restrictions on the strings used as names, does not require that name strings be unique, and does not assign any significance to the ordering of name/value pairs...
```
This removes the previously ambigous term "name" which, as a result of an omission of a definition of the term in 2 of 3 parts of [JSON Syntax](https://json.org) (_the JSON Post_), left the intent un-clear.  The term "name" was described only in the text of _the JSON Post_, but not included in the accompanying McKeeman Form or Workflow diagrams. 

This means, by agreed convention, it is possible to convey an unambiguous data-type in JSON syntax by **appending the data-type to the element name**.

This fact allows a very simple and efficient means to fully qualify the data type in JSON syntax.  It also provides, as a consequence of this simple qualification, to make it possible to define complex types and to define remote methods (aka remote procedure calls).


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


While using characters such as colons, braces and brackets in the name is valid JSON, the practice makes the object name inaccessible after a JSON.parse() call in Javascript.  This is fortunate and convenient as it forces the script to validate the received object before any qualified values are (easily) accessible to javascript code.  

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
 
 `application/json-nd`

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

## Examples

### Element Example
When using JSON-ND then following Standard JSON
```
{"id: 23, "name":"Alice", "results": [1,6,5]}
```
becomes:
```
{"id": 23, "name":"Alice", "results:integer[]":[1,6,5]}
OR
{"id:integer": 23, "name":"Alice", "results:integer[0,3]":[1,6,5]}
OR
{"id:integer": 23, "name":"Alice", "results:integer[]":[1,6,5]}

```

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

### Remote Method Examples
The following represents a JSON-ND method defintion -in this case Pascal Style.  The may be different specification for different consumers of the data.

```
{
  "SquareRoot:function(value:integer):integer" : "https://some.server.com:999/sqrt" 
}
```

Any language style is permitted and is in no way restricted or defined in this specification. The method is unabiguous as you know not to call this method with floating point values.  Again depending on the consumer, you may need to qualify the datatype allowed using a language specific construct (eg with RegularExpression for Javascript) to enforce non-floating point restriction.


