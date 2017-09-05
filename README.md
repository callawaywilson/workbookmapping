# Workbook Mapping
This document describes a data structure for defining a two-way mapping from a relational database to a standard Excel workbook file.  It is intended to allow arbitrary data in a database to be automatically populated into an Excel file and read back into the database.  

---
## Data Set to Workbook Data
It is expected that the data set be segmented into workbook data for each workbook.  This is a logical “WHERE” clause on the data set for the data that appears in the workbook.  This is manifest in the identifiers for the tables:

``` json
{
  "countries": {
    "id": 45
  },
  "country_data": {
    "country_id": 45,
    "year": 2016
  },
  "country_population": {
    "country_id": 45,
    "year": 2016
  }
}
```

These mapping values are automatically included in the identifier sections of mapping types.  For example, if countries for the workbook data is set to and id of 45, then id 45 will always be a part of the identifier for a table even if it’s not inlined in the particular mapping.

## Mapping Types
Mappings are how the data gets from the data set into a workbook template and back into the data set.

### Cell to Field
One cell in a sheet maps onto a field in the data set.
``` json
{
  "type": "cell_field",
  "cell": "Sheet 1'!A3",    // Standard Excel reference
  "behavior": "update",   // Only update existing value for existing row
  "identifier": {
    "table": "countries",
    "values": {
      "id": 45
    }
  },
  "column": "population"
}
```

### Rows to Rows
One row in a sheet maps onto a row in the data set.
``` json
{
  "type": "rows_rows",
  "sheet": "Sheet 2",
  "range": "4:",            // Closed or open row range 
  "behavior": ["update","new"]  // Update and insert based on identifier
  "identifier": {
    "table": "country_data",
    "values": [
      {"country_id": 45}    // id is equal to 45 in table
      {"year": 2016}      // year attribute is 2016 in table
      "district",       // district matches sheet to data set row
    ]
  },
  "mapping": {
    "id": {
      "type": "integer",
      "column": "B"
    },
    "district": {
      "type": "string",
      "column": "C"
    },
    "population": {
      "type": "integer",
      "column": "D"
    },
    "treated": {
      "type": "integer",
      "column": "E"
    },
    "prevalence": {
      "type": "float",
      "column": "F"
    }
  }
}
```

### Range to Row
A range of cells maps onto a row in the data set.
``` json
{
  "type": "range_rows",
  "sheet": "Sheet 1",
  "range": "A3:A4,A6",          // Range length must be same as mapping
  "behavior": ["update","new"]    // Update and insert based on identifier
  "identifier": {
    "table": "country_population",
    "values": [
      {"id": 5},          // id is equal to 5 in the table
      {"year": 2016}
    ]
  },
  "mapping": [            // Array, in order through range
    {
      "name": "male_population",
      "type": "integer"
    },
    {
      "name": "female_population",
      "type": "integer"
    },
    {
      "name": "growth_rate",
      "type": "float",
      "validators": [
        {"less_than": 1},
        {"greater_than": 0}
      ]
    }
  ]
}
```