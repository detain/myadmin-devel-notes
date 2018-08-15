# Form Fields

Overview of built-in form fields.

## addText($name, $label=null)[#](https://doc.nette.org/en/2.4/form-fields#toc-addtext)

Adds single line text field (class [TextInput](https://api.nette.org/2.4/Nette.Forms.Controls.TextInput.html)). Automatically trims left and right side whitespace. Beside [preset validation rules](https://doc.nette.org/en/2.4/form-validation), the following ones are available:

`Form::MIN_LENGTH`minimal string length`int $minLength``Form::MAX_LENGTH`maximal string length`int $maxLength``Form::LENGTH`exact length`[int $min, int $max]` or `int $length``Form::EMAIL`is value a valid email address?  
`Form::URL`is value a valid URL?  
`Form::PATTERN`tests against regular expression entire value, somewhat as if it is inside ^ and a $`string $pattern``Form::INTEGER`is value integer?  
`Form::NUMERIC`alias of `Form::INTEGER`  
`Form::FLOAT`is value a floating point number?  
`Form::MIN`minimum of the integer value`int|float $minimum``Form::MAX`maximum of the integer value`int|float $maximum``Form::RANGE`is value in the range?`[int $min, int $max]`

    $form->addText('zip', 'Postcode:')    ->setRequired()    ->addRule(Form::PATTERN, 'Postcode must have exactly 5 numerals', '([0-9]\s*){5}');

The `Form::INTEGER`, `NUMERIC` a `FLOAT` rules automatically convert the value to integer (or float respectively). In addition, `Form::URL` automatically completes the URL, for example, `nette.org` will be completed to `https://nette.org`.

## addPassword(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addpassword)

Adds password field (class [TextInput](https://api.nette.org/2.4/Nette.Forms.Controls.TextInput.html)). Automatically trims left and right side whitespace. Redrawing the form renders the input empty. Supports the same set of validation rules as [addText](https://doc.nette.org/en/2.4/form-fields#toc-addtext).

    $form->addPassword('password', 'Password:')    ->setRequired()    ->addRule(Form::MIN_LENGTH, 'Password has to be at least %d characters long', 3)    ->addRule(Form::PATTERN, 'Password must contain a number', '.*[0-9].*');

## addTextArea(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addtextarea)

Adds a multiline text field (class [TextArea](https://api.nette.org/2.4/Nette.Forms.Controls.TextArea.html)). Supports the same set of validation rules as [addText](https://doc.nette.org/en/2.4/form-fields#toc-addtext). Unlike oneline inputs, it does not trim the input's whitespace on either edge.

    $form->addTextArea('note', 'Note:')    ->setRequired(false) // optional    ->addRule(Form::MAX_LENGTH, 'Your note is way too long', 10000);

## addEmail(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addemail)

Adds email address field with validity check (class [TextInput](https://api.nette.org/2.4/Nette.Forms.Controls.TextInput.html)). Supports the same set of validation rules as [addText](https://doc.nette.org/en/2.4/form-fields#toc-addtext).

    $form->addEmail('email', 'Email:');

## addInteger(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addinteger)

Adds input field for integer. Supports the same set of validation rules as [addText](https://doc.nette.org/en/2.4/form-fields#toc-addtext).

    $form->addInteger('level', 'Level:')    ->setDefaultValue(0)    ->addRule($form::RANGE, 'Level must be between %d and %d.', [0, 100]);

## addUpload(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addupload)

Adds file upload field (class [UploadControl](https://api.nette.org/2.4/Nette.Forms.Controls.UploadControl.html)). Beside [preset validation rules](https://doc.nette.org/en/2.4/form-validation), the following ones are available:

`Form::MAX_FILE_SIZE`verifies maximal file size`int $bytes``Form::MIME_TYPE`checks if MIME type is valid`string $mimeType` or `$mimeTypes[]`. Can contain wildcards, such as `video/*`.`Form::IMAGE`checks if uploaded file is JPEG, PNG or GIF

    $form->addUpload('thumbnail', 'Thumbnail:')    ->setRequired(false) // optional    ->addRule(Form::IMAGE, 'Thumbnail must be JPEG, PNG or GIF')    ->addRule(Form::MAX_FILE_SIZE, 'Maximum file size is 64 kB.', 64 * 1024 /* size in Bytes */);

## addMultiUpload(_string|int_ $name, $label=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addmultiupload)

Adds multiple file upload field. Validation rules are same as `addUpload()` and adds following:

`Form::MIN_LENGTH`minimal count files`int $minCount``Form::MAX_LENGTH`maximal count files`int $maxCount``Form::LENGTH`exact count of uploaded files`[int $minCount, int $maxCount]` or `int $count`

    $form->addMultiUpload('files', 'Files');

## addHidden(_string|int_ $name, _string_ $default=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addhidden)

Adds hidden field (class [HiddenField](https://api.nette.org/2.4/Nette.Forms.Controls.HiddenField.html)).

    $form->addHidden('userid');

## addCheckbox(_string|int_ $name, $caption=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addcheckbox)

Adds a checkbox (class [Checkbox](https://api.nette.org/2.4/Nette.Forms.Controls.Checkbox.html)). The return value is either Boolean `true` or `false`, as checked or not checked.

    $form->addCheckbox('agree', 'I agree with terms')    ->setRequired('You must agree with our terms');

## addRadioList(_string|int_ $name, $label=null, _array_ $items=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addradiolist)

Adds radio buttons (class [RadioList](https://api.nette.org/2.4/Nette.Forms.Controls.RadioList.html)). Array of offered values is passed as the third argument.

    $sex = [    'm' => 'male',    'f' => 'female',];$form->addRadioList('gender', 'Gender:', $sex);// to list options within 1 line$form->addRadioList('gender', 'Gender:', $sex)    ->getSeparatorPrototype()->setName(null);

## addCheckboxList(_string|int_ $name, $label=null, _array_ $items=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addcheckboxlist)

Adds list of checkboxes for selecting multiple elements. Array of values is passed as the third argument. The component checks if the submitted values are from the given range.

    $form = new Form;$form->addCheckboxList('colors', 'Colors:', [    'r' => 'red',    'g' => 'green',    'b' => 'blue',]);

## addSelect(_string|int_ $name, $label=null, _array_ $items=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addselect)

Adds select box (class [SelectBox](https://api.nette.org/2.4/Nette.Forms.Controls.SelectBox.html)). Array of offered values is passed as third argument. Might as well be two-dimensional. The first item is often used as a call-to-action message, but worthless when actually selected – that's what method `setPrompt()` is for.

    $countries = [    'Europe' => [        'CZ' => 'Czech republic',        'SK' => 'Slovakia',        'GB' => 'United Kingdom',    ],    'CA' => 'Canada',    'US' => 'USA',    '?'  => 'other',];$form->addSelect('country', 'Country:', $countries)    ->setPrompt('Pick a country');

You can also add elements by using the [setItems()](https://api.nette.org/2.4/Nette.Forms.Controls.SelectBox.html#_setItems) method. If we want to get their values directly in place of key items, we can do this with the second argument:

    $form->addSelect('country', 'Country:')    ->setItems($countries, false);

Individual items may be, in addition to strings, objects `Nette\Utils\Html::('option')`, which then can set additional HTML attributes. However, the `selected` and `disabled` attributes do not set this way, the form itself takes care of it. This also applies to `addMultiSelect()`.

## addMultiSelect(_string|int_ $name, $label=null, _array_ $items=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addmultiselect)

Adds multichoice select box (class [MultiSelectBox](https://api.nette.org/2.4/Nette.Forms.Controls.MultiSelectBox.html)).

    $form->addMultiSelect('options', 'Pick many:', $options);

## addSubmit(_string|int_ $name, $caption=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addsubmit)

Adds submit button (class [SubmitButton](https://api.nette.org/2.4/Nette.Forms.Controls.SubmitButton.html)).

    $form->addSubmit('submit', 'Register');

If you don't want to validate the form when a submit button is pressed (such as _Cancel_ or _Preview_buttons), you can turn it off with `setValidationScope([])`.

## addButton(_string|int_ $name, $caption): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addbutton)

Adds button (class [Button](https://api.nette.org/2.4/Nette.Forms.Controls.Button.html)) without submit function. It is useful for binding other functionality to id, for example a JavaScript action.

    $form->addButton('raise', 'Raise salary')    ->setHtmlAttribute('onclick', 'raiseSalary()');

## addImage(_string|int_ $name, $alt=null): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addimage)

Adds submit button in form of an image (class [ImageButton](https://api.nette.org/2.4/Nette.Forms.Controls.ImageButton.html)).

    $form->addImage('submit', '/path/to/image');

## addContainer(_string|int_ $name): _static_[#](https://doc.nette.org/en/2.4/form-fields#toc-addcontainer)

Adds a sub-form (class [Container](https://api.nette.org/2.4/Nette.Forms.Container.html)), or a container, which can be treated the same way as a form. That means you can use methods like `setDefaults()` or `getValues()`.

    $sub1 = $form->addContainer('first');$sub1->addText('name', 'Your name:');$sub1->addEmail('email', 'Email:');$sub2 = $form->addContainer('second');$sub2->addText('name', 'Your name:');$sub2->addEmail('email', 'Email:');	'

