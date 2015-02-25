Validation Module
====================================
### Module Author

* [İskender TOTOĞLU](http://www.altivebir.com.tr)

Validation Module for ProcessWire, Validation module using [GUMP](https://github.com/Wixel/GUMP) standalone PHP data validation and filtering class. That makes validating any data easy and painless without the reliance on a framework. Usage almost same with original [GUMP](https://github.com/Wixel/GUMP) validation class. Extended original validation class for make it multi-language and added 2 new function 1 for field labels, 1 for set and get fields.


#### Available Methods

```php
// Set & Get Fields (Extended Method)
setFields(array $fields);

// Set Field Labels (Extended Method)
set_field_labels(array $labels);

// Shorthand validation
is_valid(array $data, array $rules) 

// Get or set the validation rules
validation_rules(array $rules); 

// Get or set the filtering rules
filter_rules(array $rules); 

// Runs the filter and validation routines
run(array $data); 

// Strips and encodes unwanted characters
xss_clean(array $data); 

// Sanitizes data and converts strings to UTF-8 (if available), 
// optionally according to the provided field whitelist
sanitize(array $input, $whitelist = NULL); 

// Validates input data according to the provided ruleset (see example)
validate(array $input, array $ruleset); 

// Filters input data according to the provided filterset (see example)
filter(array $input, array $filterset); 

// Returns human readable error text in an array or string
get_readable_errors($convert_to_string = false); 

// Fetch an array of validation errors indexed by the field names
get_errors_array();

// Override field names with readable ones for errors
set_field_name($field, $readable_name);
```

# Example (Long format)

The following example is part of a registration form, the flow should be pretty standard

```php
// Set Fields With Options
$Fields = [
    'firstname' => [
        'label'     => __('First Name'),
        'rule'      => 'required|max_len,100|min_len,3',
        'filter'    => 'sanitize_string',
        'type'      => 'text'
    ],
    'lastname' => [
        'label'     => __('Last Name'),
        'rule'      => 'required|max_len,100|min_len,3',
        'filter'    => 'sanitize_string',
        'type'      => 'text'
    ],
    'email'	=> [
        'label'     => __('E-mail'),
        'rule'      => 'required|valid_email',
        'filter'    => 'sanitize_string',
        'type'      => 'text'
    ],
    'subject' => [
        'label'     => __('Subject'),
        'rule'      => 'required',
        'filter'    => 'sanitize_string',
        'type'      => 'text'
    ],
    'message' => [
        'label'     => __('Message'),
        'rule'      => 'required',
        'filter'    => 'sanitize_string',
        'type'      => 'textarea'
    ]
];

// Call Validation Module
$Validation = new Validation();

// Set Form Fields Validations
$Fields = $Validation->setFields($Fields);

// You don't have to sanitize, but it's safest to do so.
$_POST = $Validation->sanitize($_POST);

// Set rules and run ! Cheate a variable for check form post validated?
$isValid = $Validation->is_valid($_POST, $Fields);

// Check Post Data
if($_POST) {
    // valid
    if($isValid === TRUE) {
        echo "<h4>Your form validation is ok !</h4>";
        echo "<p>Your form validation is ok, now you can do what you want !</p>";
    }
    // notValid
    else {
        echo "<h4>Your form validations isn't ok, check following errors !</h4>";
            foreach($isValid as $key => $errors)
                echo $errors.'<br />';
    }
}
// Check if form validation not ok, show our form
if($isValid !== TRUE)
{
    // Create form !
    echo "<form method='post'>";
    foreach($Fields['Fields'] as $key => $field)
    {
        switch($field['type'])
        {
            case 'textarea':
                echo "<label for='{$key}'>{$field['label']}</label> <textarea id='{$key}' name='{$key}'>{$field['value']}</textarea><br />";
                break;
            default:
                echo "<label for='{$key}'>{$field['label']}</label> <input type='text' id='{$key}' name='{$key}' value='{$field['value']}' /><br />";
                break;
        }
    }
    echo "<input type='submit' value='Submit' />";
    echo "</form>";
}
```
```php
$validator = new Validation();

$_POST = $validator->sanitize($_POST);  //  You don't have to sanitize, but it's safest to do so.

$validator->validation_rules(array(
	'username'    => 'required|alpha_numeric|max_len,100|min_len,6',
	'password'    => 'required|max_len,100|min_len,6',
	'email'       => 'required|valid_email',
	'gender'      => 'required|exact_len,1|contains,m f',
	'credit_card' => 'required|valid_cc'
));

$validator->filter_rules(array(
	'username' => 'trim|sanitize_string',
	'password' => 'trim',
	'email'    => 'trim|sanitize_email',
	'gender'   => 'trim',
	'bio'	   => 'noise_words'
));

$validated_data = $validator->run($_POST);

if($validated_data === false) {
	echo $validator->get_readable_errors(true);
} else {
	print_r($validated_data); // validation successful
}
```

# Example (Short format)

The short format is an alternative way to run the validation.

```php
$data = array(
	'street' => '6 Avondans Road'
);

$validated = Validation::is_valid($data, array(
	'street' => 'required|street_address'
));

if($validated === true) {
	echo "Valid Street Address!";
} else {
	print_r($validated);
}
```


Match data-keys against rules-keys
-------------
We can check if there is a rule specified for every data-key, by adding an extra parameter to the run method.

```
Validation::run($_POST, true);
```

If it doesn't match the output will be:
```
There is no validation rule for <span class=\"$field_class\">$field</span>
```

Return Values
-------------
`run()` returns one of two types:

*ARRAY* containing the successfully validated and filtered data when the validation is successful

*BOOLEAN* False when the validation has failed

`validate()` returns one of two types:

*ARRAY* containing key names and validator names when data does not pass the validation.

You can use this array along with your language helpers to determine what error message to show.

*BOOLEAN* value of TRUE if the validation was successful.

`filter()` returns the exact array structure that was parsed as the `$input` parameter, the only difference would be the filtered data.


Available Validators
--------------------
* required `Ensures the specified key value exists and is not empty`
* valid_email `Checks for a valid email address`
* max_len,n `Checks key value length, makes sure it's not longer than the specified length. n = length parameter.`
* min_len,n `Checks key value length, makes sure it's not shorter than the specified length. n = length parameter.`
* exact_len,n `Ensures that the key value length precisely matches the specified length. n = length parameter.`
* alpha `Ensure only alpha characters are present in the key value (a-z, A-Z)`
* alpha_numeric `Ensure only alpha-numeric characters are present in the key value (a-z, A-Z, 0-9)`
* alpha_dash `Ensure only alpha-numeric characters + dashes and underscores are present in the key value (a-z, A-Z, 0-9, _-)`
* alpha_space `Ensure only alpha-numeric characters + spaces are present in the key value (a-z, A-Z, 0-9, \s)`
* numeric `Ensure only numeric key values`
* integer `Ensure only integer key values`
* boolean `Checks for PHP accepted boolean values, returns TRUE for "1", "true", "on" and "yes"`
* float `Checks for float values`
* valid_url `Check for valid URL or subdomain`
* url_exists `Check to see if the url exists and is accessible`
* valid_ip `Check for valid generic IP address`
* valid_ipv4 `Check for valid IPv4 address`
* valid_ipv6 `Check for valid IPv6 address`
* valid_cc `Check for a valid credit card number (Uses the MOD10 Checksum Algorithm)`
* valid_name `Check for a valid format human name`
* contains,n `Verify that a value is contained within the pre-defined value set`
* containsList,n `Verify that a value is contained within the pre-defined value set. Comma separated, list not outputted.`
* doesNotcontainList,n `Verify that a value is not contained within the pre-defined value set. Comma separated, list not outputted.`
* street_address `Checks that the provided string is a likely street address. 1 number, 1 or more space, 1 or more letters`
* iban `Check for a valid IBAN`
* min_numeric `Determine if the provided numeric value is higher or equal to a specific value`
* max_numeric `Determine if the provided numeric value is lower or equal to a specific value`
* date `Determine if the provided input is a valid date (ISO 8601)`
* starts `Ensures the value starts with a certain character / set of character`

Available Filters
-----------------
Filters can be any PHP function that returns a string. You don't need to create your own if a PHP function exists that does what you want the filter to do.

* sanitize_string `Remove script tags and encode HTML entities, similar to GUMP::xss_clean();`
* urlencode `Encode url entities`
* htmlencode `Encode HTML entities`
* sanitize_email `Remove illegal characters from email addresses`
* sanitize_numbers `Remove any non-numeric characters`
* trim `Remove spaces from the beginning or end of strings`
* base64_encode `Base64 encode the input`
* base64_decode `Base64 decode the input`
* sha1 `Encrypt the input with the secure sha1 algorithm`
* md5 `MD5 encode the input`
* noise_words `Remove noise words from string`
* json_encode `Create a json representation of the input`
* json_decode `Decode a json string`
* rmpunctuation `Remove all known punctuation characters from a string`
* basic_tags `Remove all layout orientated HTML tags from text. Leaving only basic tags`
* whole_number `Ensure that the provided numeric value is represented as a whole number`

#  Creating your own validators and filters

Adding custom validators and filters is made easy by using callback functions.

```php
/* 
   Create a custom validation rule named "is_object".   
   The callback receives 3 arguments:
   The field to validate, the values being validated, and any parameters used in the validation rule.
   It should return a boolean value indicating whether the value is valid.
*/
Validation::add_validator("is_object", function($field, $input, $param = NULL) {
    return is_object($input[$field]);
});

/* 
   Create a custom filter named "upper".
   The callback function receives two arguments:
   The value to filter, and any parameters used in the filter rule. It should returned the filtered value.
*/
Validation::add_filter("upper", function($value, $params = NULL) {
    return strtoupper($value);
});

```
# Set Custom Field Names

You can easily override your form field names for improved readability in errors using the `Validation::set_field_name($field, $readable_name)` method as follows:

```php
$data = array(
	'str' => null
);

$rules = array(
	'str' => 'required'
);

Validation::set_field_name("str", "Street");

$validated = Validation::is_valid($data, $rules);

if($validated === true) {
	echo "Valid Street Address\n";
} else {
	print_r($validated);
}
```
