# Annotation Cheat-Sheet for CSDL JSON

How to construct an annotation from a term definition, or re-engineer a term definition from an annotation example?

Here's how.

## Vocabularies

Terms are defined within a "vocabulary", which is just an EDMX document.

```json
{
  "$Version": "4.0",
  "Vocab": {
```

To use a term in an annotation, the vocabulary of the term needs to be referenced and included.

```json
{
  "$Version": "4.0",
  "$Reference": {
    "https://somewhere.org/vocab.xml": {
      "$Include": [
        {
            "$Namespace": "Vocab"
        }
      ]
    }
  }
```

## Primitive Terms

If the term has a primitive type

```json
"StringTerm": {
  "$Kind": "Term"
}
```

A _constant annotation value_ can be provided as a [corresponding JSON value](https://docs.oasis-open.org/odata/odata-csdl-json/v4.01/odata-csdl-json-v4.01.html#sec_ConstantExpression).

```json
"@Vocab.StringTerm": "annotation value"
```

See [Constant Annotation Values](#constant-annotation-values) for a list of all primitiv types and example values for them.

A _dynamic annotation value_ can be provided for the same term using a [value path expression](https://docs.oasis-open.org/odata/odata-csdl-json/v4.01/odata-csdl-json-v4.01.html#sec_ValuePath).

```json
"@Vocab.StringTerm": {
  "$Path": "SomeStringProperty"
}
```

The property referenced via the value path expression, here `SomeStringProperty` needs to have the same type as the term. A value path expression can always be used instead of a constant value, also in the more complicated cases below.

## Collections

If the term has a collection type

```json
"CollectionTerm": {
  "$Kind": "Term",
  "$Collection": true,
  "$Type": "Edm.Decimal"
}
```

the annotation value is provided as a [collection expression](https://docs.oasis-open.org/odata/odata-csdl-json/v4.01/odata-csdl-json-v4.01.html#sec_Collection)

```json
"@Vocab.CollectionTerm": [
  2.78,
  3.14
]
```

Note: decimal values can also be represented as strings to prevent loss of precision in JavaScript clients.

## Structures

If the term has a structured type

```json
"StructuredTerm": {
  "$Kind": "Term",
  "$Type": "Vocab.Complex"
},

"Complex": {
  "$Kind": "ComplexType",
  "IntegerField": {
    "$Type": "Edm.Int32"
  }
}
```

the annotation value is provided as a [record expression](https://docs.oasis-open.org/odata/odata-csdl-json/v4.01/odata-csdl-json-v4.01.html#sec_Record)

```json
"@Vocab.StructuredTerm": {
  "IntegerField": 42
}
```

[Constant annotation values](#constant-annotation-values) for primitive properties are provided in the same way as for primitive terms.

## Collection of Structures

Terms can also be typed as a collection of a structured type.

```json
"StructuredCollectionTerm": {
  "$Kind": "Term",
  "$Collection": true,
  "$Type": "Vocab.AnotherComplex"
},

"AnotherComplex": {
  "$Kind": "ComplexType",
  "DateField": {
    "$Type": "Edm.Date",
    "$Nullable": true
  },
  "TimeField": {
    "$Type": "Edm.TimeOfDay",
    "$DefaultValue": "00:00:00"
  }
}
```

The annotation value is provided as a collection of records.

```json
"@Vocab.StructuredCollectionTerm": [
  {
    "DateField": "2020-06-30",
    "TimeField": "16:55:03"
  },
  {
    "DateField": "2020-07-01"
  },
  {
    "TimeField": "23:59:59"
  }
]
```

Properties that are nullable or have a default value can be omitted.

## Nested Structures and Collections

Properties of a structured type can themselves be structured or collections.

```json
"NestedTerm": {
  "$Kind": "Term",
  "$Collection": true,
  "$Type": "Vocab.YetAnotherComplex",
},

"YetAnotherComplex": {
  "StructuredField": {
    "$Type": "Vocab.Complex"
  },
  "CollectionField": {
    "$Collection": true,
    "$Type": "Vocab.AnotherComplex"
  }
}
```

The property value is provided as a record or collection.

```json
"@Vocab.NestedTerm": {
  "StructuredField": {
    "IntegerField": 42
  },
  "CollectionField": [
    {
      "DateField": "2020-06-30",
      "TimeField": "16:55:03"
    }
  ]
}
```

## Constant Annotation Values

The name of the attribute to provide a constant annotation value depends on the type of the term or term property.

| Type of Term or Term Property | Example Value |
|:-|:-|
| `Edm.Binary` | `"T0RhdGE"` |
| `Edm.Boolean` | `true` |
| `Edm.Date` | `"2000-01-01"` |
| `Edm.DateTimeOffset` | `"2000-01-01T16:00:00.000Z"` |
| `Edm.Decimal` | `"3.14"` |
| `Edm.Duration` | `"P7D"` |
| Enumeration Type | `"Red"` |
| `Edm.Double` or `Edm.Single` | `3.14` |
| `Edm.Guid` | `"21EC2020-3AEA-1069-A2DD-08002B30309D"` |
| `Edm.Int16`, `Edm.Int32`, `Edm.64`, `Edm.Byte`, `Edm.SByte` | `42` |
| `Edm.Int64` | `"42"` |
| `Edm.String` | `"annotation value"` |
| `Edm.TimeOfDay` | `"21:45:00"` |
| `Edm.AnnotationPath` | `"Product/Supplier/@UI.LineItem"` |
| `Edm.NavigationPropertyPath` | `"Supplier"` |
| `Edm.PropertyPath` | `"Details/ChangedAt"` |
