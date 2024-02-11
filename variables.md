# Variables


## Simple variables
In this file we learn to work with variables. They are declared with a type and a default value:

```bash
variable "name" {
  type = string
  default = "My Name"
}
```
We can use an interactive console to interact with this variable. Remember that terraform console in principle doesn't allow things like overwriting values of variables.

To start the console, run `terraform console` in a terminal. In the following examples the `>` symbol is the prompt that there is in terraform console, and that precedes commands. After that, the output that we get is shown.

```bash
# Variables like the one declared above (with the keyword "variable"), are accessed as var.<variable name>
> var.greeting
"Hello, World!"
> var.name
"My Name"

# We get an error if we try to access non existing variables
> var.asdf
╷
│ Error: Reference to undeclared input variable
│ 
│   on <console-input> line 1:
│   (source code not available)
│ 
│ An input variable with the name "asdf" has not been declared. This variable
│ can be declared with a variable "asdf" {} block.
╵
```

The `locals` keyword allows to declare variable in a local scope of the file where they are declared, and cannot be accessed from objects declared outside that locals block. For example, if we declare in a terraform file:
```bash
locals {
    hey =  "asfdasdf"
}
```
We can access that value in the console as `local.hey`. Also, inside a locals block you cannot define variables with a `variables` block, as we showed above.

## Arrays
Arrays are declared like this:

```bash
 variable "test_array" {
   type = list(string) 
   default = ["foo", "bar", "baz"]
 } 
```
In the `type` field, we need to specify the type inside the keyword `list()`. We can in a terraform console print this list or slice its elements:

```bash
# print the array (that "tolist" is added by terraform).
> var.test_array
tolist([
  "foo",
  "bar",
  "baz",
])
```

We can create a for each structure that will iterate over elements of the list and perform some action (defined after the ":" ).

```bash
> [ for value in var.test_array : value ]
[
  "foo",
  "bar",
  "baz",
]
```

The element of a list can also be a complex object, with its internal structure (don't forget to represent this structure with the corresponding key-type pair in the type definition at the top of the variable declaration):
```bash
variable "test_array2" {
  type = list(object({
    region = string
  }))

  default = [
    {
      region = "us-east-1"
    },
    {
      region = "eu-west-1"
    }
  ]
}
```

Next are shown some ways to access the elements of such list of objects:

```bash
# Return the complete array
> var.test_array2
tolist([
  {
    "region" = "us-east-1"
  },
  {
    "region" = "eu-west-1"
  },
])

# Return the second element
> var.test_array2[1]
{
  "region" = "eu-west-1"
}
> var.test_array2[1].region
"eu-west-1"

# Iterate through the elements of the object and return the "region" values
> [ for val in var.test_array2 : val.region ]
[
  "us-east-1",
  "eu-west-1",
]

# Instead of just returning, perform some operation, like a boolean comparison, to get which elements return true and which return false after such comparison
> [ for val in var.test_array2 : val.region == "eu-west-1" ]
[
  false,
  true,
]
```

In the next case we use a more complicated variable, consisting of an array of objects, with nested sub-objects:

```bash
variable "test_array3" {
  type = list(object({
    area = object({
      region = string
    })
  }))

  default = [
    {
      area = {
        region = "us-east-1"
      }
    },
    {
      area = {
        region = "eu-west-1"
      }
    }
  ]
}
```

Next are some ways to interact with the elements at different nesting levels in terraform console:
```
# Return the first element
> var.test_array3[0]
{
  "area" = {
    "region" = "us-east-1"
  }
}

# Return the value of the "area" property in the first element
> var.test_array3[0].area
{
  "region" = "us-east-1"
}

# Return the nested value of region in the first element
> var.test_array3[0].area.region
"us-east-1"

# Return a list of regions as a result of iterating through the list of objects
> [ for i in var.test_array3 : i.area.region ]
[
  "us-east-1",
  "eu-west-1",
]

# Format the output text, variables inside strings are used with ${...}
> [ for i in var.test_array3 : "hola: ${i.area.region}" ]
[
  "hola: us-east-1",
  "hola: eu-west-1",
]

# Iterate over the list elements and return keys with a new overriden value (in this case, a constant "test")
> [ for v in var.test_array3 : { for k, i in v.area : k => "test" }]
[
  {
    "region" = "test"
  },
  {
    "region" = "test"
  },
]

# it's possible to unzip keys and values, and in the case of arrays keys are indices:
> [ for k, v in var.test_array3 : k ]
[
  0,
  1,
]

# While if we unzip, values are the same as before:
> [ for k, v in var.test_array3 : v ]
[
  {
    "area" = {
      "region" = "us-east-1"
    }
  },
  {
    "area" = {
      "region" = "eu-west-1"
    }
  },
]

# You can use wildcards to access sub-elements of all elements in the loop (the matched keys don't appear in the results)
> [ for v in var.test_array3.*.area : v ]
[
  {
    "region" = "us-east-1"
  },
  {
    "region" = "eu-west-1"
  },
]

# An array can also be formed in the output of every element in the output list
> [ for v in var.test_array3.*.area : ["region is: ",v.region] ]
[
  [
    "region is: ",
    "us-east-1",
  ],
  [
    "region is: ",
    "eu-west-1",
  ],
]
```

Now consider this object, with several attributes in each sub object

```bash
variable "test_array4" {
  type = list(object({
    area = object({
      code = string
      region = string
    })
  }))

  default = [
    {
      area = {
        code = "US"
        region = "us-east-1"
      }
    },
    {
      area = {
        code = "EU"
        region = "eu-west-1"
      }
    }
  ]
}
```

These are some ways to interact with this object in the terraform console:

```bash
# Output of an array consising of different elements
> [ for v in var.test_array4.*.area : [v.code, v.region] ]
[
  [
    "US",
    "us-east-1",
  ],
  [
    "EU",
    "eu-west-1",
  ],
]

# Return key value pairs:
> { for k, v in var.test_array4.*.area : v.code => v.region }
{
  "EU" = "eu-west-1"
  "US" = "us-east-1"
}
```
