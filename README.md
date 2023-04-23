# OCA Data Set Validator
This is a Python script for validating [Overlays Capture Architecture (OCA)](https://oca.colossi.network/) data sets. It includes three classes: `OCADataSet`, `OCADataSetErr`, and `OCABundle`. For more information about OCA, please check [OCA Specification v1.0.0](https://oca.colossi.network/specification/).

- `OCADataSet` represents an OCA data set to be validated, and can be loaded from a pandas DataFrame, an OCA Excel Data Entry File, or a CSV file.

- `OCADataSetErr` represents the result set of an OCA data set validation. This class is generated by the data set validation, contains all the error information, and also provides three methods for a quick overview: `overview()`, `first_error_col()`, and `get_error_col(attr_name)`.

- `OCABundle` represents a loaded OCA bundle from OCA zip files, which are used to validate the data set.

## Dependencies
- pandas
- pathlib

## Usage

### Installation
Download and copy `oca_ds_validator.py` to your working folder. Then you could import the classes from any Python scripts in the same directory.

### Validation
1. Import the OCA Bundle using `OCABundle(path)`.
2. Import the OCA Data Set using `OCADataSet(pandas_dataframe)` or `OCADataSet.from_path(path)`.
3. Generate the validation result using `validate()` method for class `OCABundle`.

```python
from oca_ds_validator import OCADataSet, OCADataSetErr, OCABundle

test_bundle = OCABundle("/path/to/oca/bundle.zip")

test_data = OCADataSet(data_set_dataframe)
# test_data = OCADataSet.from_path("/path/to/oca/data_entry_file.xlsx")
# test_data = OCADataSet.from_path("/path/to/oca/data_set_file.csv")

test_rslt = test_bundle.validate(test_data)
#########################################################################################
# Example of a possible test_rslt:
#   attr_err:
#     [('missing_attribute', 'Missing attribute (attribute not found in the data set).'), 
#      ('unmatched_attribute', 'Unmatched attribute (attribute not found in the OCA Bundle).')]
#   format_err:
#     {'attribute_with_format_error_on_row_0': {0: 'Format mismatch.'}, 
#      'array_attribute_without_array_data_on_row_0': {0: 'Valid array required.'}, 
#      'attribute_with_format_error_on_row_42': {42: 'Format mismatch.'}, 
#      'attribute_with_errors_on_row_0_and_1': {0: 'Format mismatch.', 
#                                               1: 'Valid array required.'}, 
#      'mandatory_attribute_with_missing_data': {0: 'Missing mandatory attribute.'},
#      'attribute_without_error': {}}
#   ecode_err:  # Not matching any of the entry codes
#     {'attribute_with_entry_codes': {0: 'One of the entry codes required.'}}
#########################################################################################
```

### Optional Messages
There are three optional boolean arguments to control the message printed.
| argument | default value | usage |
| -------- | ------------- | ----- |
| `show_data_preview` | `False` | If enabled, prints a pandas preview of the data set before validation. |
| `enable_flagged_alarm` | `True` | If enabled, prints a warning message for the existence of flagged attributes. |  
| `enable_version_alarm` | `True` | If enabled, prints a warning message for each overlay that contains an OCA version number different from the development version of this script (1.0).


### Result Observation
The errors of the data set is stored in the generated `OCADataSetErr` class.

```Python
# Prints a brief summary of errors. 
test_rslt.overview()  
#########################################################################################
# Attribute error. {'missing_attribute'} found in the OCA Bundle but not in the data set; 
# {'unmatched_attribute'} found in the data set but not in the OCA Bundle.
# Found 3 problematic row(s) in the following attribute(s): 
# {'attribute_with_format_error_on_row_0', 
#  'array_attribute_without_array_data_on_row_0', 
#  'attribute_with_format_error_on_row_42', 
#  'attribute_with_errors_on_row_0_and_1', 
#  'mandatory_attribute_with_missing_data',
#  'attribute_with_entry_codes'}
#########################################################################################

# Prints the information of the first problematic column.
test_rslt.first_err_col()  
#########################################################################################
# The first problematic column is: attribute_with_format_error_on_row_0
# Format error(s) would occur in the following rows:
# row 0 : Format mismatch.
# No entry code error found in the column.
#########################################################################################

# Prints the information of some certain column.
test_rslt.get_err_col("attribute_with_format_error_on_row_42")  
#########################################################################################
# Format error(s) would occur in the following rows of column 
# attribute_with_format_error_on_row_42:
# row 42 : Format mismatch.
# No entry code error found in the column.
#########################################################################################
```

### Further Processing
```Python
# Get objects of full error details. 
# You may find it useful for data visualization or further analysis.
test_rslt.get_attr_err()
test_rslt.get_format_err()
test_rslt.get_ecode_err()
```

## Development Status

This script is created with support by [Agri-food Data Canada](https://agrifooddatacanada.ca/), funded by [CFREF](https://www.cfref-apogee.gc.ca/) through the [Food from Thought grant](https://foodfromthought.ca/) held at the [University of Guelph](https://www.uoguelph.ca/). Currently, we do not provide any warranty of any kind regarding the accuracy, security, completeness or reliability of this script or any of its parts.

At the moment, this script is developed for the validation of the following [OCA attribute types](https://oca.colossi.network/specification/#attribute-type): 
- Text (with regular expressions)
- DateTime (with ISO 8601 formats)
- Array[Type]; for any Types that are not mentioned above, only the validness of the array will be checked. 

Also, besides the format overlay, the data set will be validated with the following overlays:
- [Conformance Overlay](https://oca.colossi.network/specification/#conformance-overlay), for any missing mandatory data
- [Entry Code Overlay](https://oca.colossi.network/specification/#entry-code-overlay), for any data that mismatches entry codes

JSON data types are **NOT** validated due to the type coercion while importing Excel or CSV files. We also recommend that you import the data set as Pandas DataFrame to prevent unexpected DateTime formatting by software such as Microsoft Excel.

Any validation errors other than the above are **NOT** guaranteed to be filtered by this script. Please feel free to contact us with any suggestions for future development.

You could also find a well-developed [OCA Validator](https://github.com/THCLab/oca-conductor) by [The Human Colossus Lab](https://github.com/THCLab) (Rust required).


## License

EUPL (European Union Public License), version 1.2 

We have distilled the most crucial license specifics to make your adoption seamless: [see here for details](https://github.com/THCLab/licensing).
