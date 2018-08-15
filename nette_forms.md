# Nette Forms

Some notes/code:
```php
$form->addText('name', 'Your name')->setRequired('Enter your name');
$form->addRadioList('gender', 'Your gender', ['male', 'female']);
$form->addCheckboxList('colors', 'Favorite colors', ['red', 'green', 'blue']);
$form->addSelect('country', 'Country', ['Buranda', 'Qumran', 'Saint Georges Island']);
$form->addCheckbox('send', 'Ship to address');
$form->addGroup('Your account');
$form->addPassword('password', 'Choose password');
$form->addUpload('avatar', 'Picture');
$form->addTextArea('note', 'Comment');
```

### https://my.interserver.net/nette/bootstrap3-rendering.php

### https://github.com/nette/forms

https://dev.nette.org/en/ajax

https://dev.nette.org/en/quickstart/comments

https://api.nette.org/2.4/namespace-Nette.Forms.html

https://doc.nette.org/en/2.4/forms

https://editor.nette.org/

https://doc.nette.org/en/2.4/form-fields

https://doc.nette.org/en/2.4/form-validation

https://doc.nette.org/en/2.4/form-rendering

# Forms

Nette Forms greatly facilitates creating and processing web forms. What it can really do?

*   validate sent data both client-side (JavaScript) and server-side
*   provide high level of security
*   multiple render modes
*   translations, i18n

Nette Framework puts a great effort to be safe and since forms are the most common user input, Nette forms are as good as impenetrable. All is maintained dynamically and transparently, nothing has to be set manually. Well-known vulnerabilities such as [Cross Site Scripting (XSS)](https://doc.nette.org/en/2.4/vulnerability-protection#toc-cross-site-scripting-xss) and [Cross-Site Request Forgery (CSRF)](https://doc.nette.org/en/2.4/vulnerability-protection#toc-cross-site-request-forgery-csrf) are filtered, as well as special control characters. All inputs are checked for UTF-8 validity. Every multiple-choice, select boxes and similar are checked for forged values upon validating. Sounds good? Let's try it out.

With Nette Forms, you can diminish routine tasks like double validation on both server and client side. You can also avoid common mistakes and security issues.

## First Form[#](https://doc.nette.org/en/2.4/forms#toc-first-form)

Let's create a simple registration form for our app. Forms are added into presenters using component [factories](https://doc.nette.org/en/2.4/presenters#toc-component-factories):

    use Nette\Application\UI;class HomepagePresenter extends UI\Presenter{    // ...    protected function createComponentRegistrationForm()    {        $form = new UI\Form;        $form->addText('name', 'Name:');        $form->addPassword('password', 'Password:');        $form->addSubmit('login', 'Sign up');        $form->onSuccess[] = [$this, 'registrationFormSucceeded'];        return $form;    }    // called after form is successfully submitted    public function registrationFormSucceeded(UI\Form $form, $values)    {        // ...        $this->flashMessage('You have successfully signed up.');        $this->redirect('Homepage:');    }}

[render in template](https://doc.nette.org/en/2.4/form-rendering) is done using control macro:

    {control registrationForm}

and the result should look like this:

We have created a simple form with name and password [form fields](https://doc.nette.org/en/2.4/form-fields), which will call `registrationFormSucceeded()` after being submitted and successfully validated. The form itself is passed as the first parameter. The submitted values are passed as the second parameter contained in a [ArrayHash](https://doc.nette.org/en/2.4/arrays#toc-arrayhash) object. If you prefer a simple array, you can typehint the second parameter `array $values`. You can also use `$values = $form->getValues()` to retrieve the submitted values.

In the context of [presenter lifecycle](https://doc.nette.org/en/2.4/presenters#toc-life-cycle-of-presenter) the form is processed on the same level as the processing of signals (`handle*` methods), that is right after the `action*` method and before the `render*` method.

The rendered form follows basic web accessibility guidelines. All labels are generated as `<label>`elements and are associated with their inputs, clicking on the label moves cursor on the input.

Data stored in `$values` do not contain values of form buttons, so they're ready for more operations (such as inserting into database). It is noteworthy that whitespace at both left and right side of the input is removed. Just try it out: put your name into the form field, add few spaces and submit it: the name will be _trimmed_.

Although we mentioned [validation](https://doc.nette.org/en/2.4/form-validation), our form has none. Let's fix it. In order to require user's name, call `setRequired()` method on the form item. You can pass an error message as optional argument and it will be displayed if user does not fill his name in:

    $form->addText('name', 'Name:')    ->setRequired('Please fill your name.');

Try submitting a form without the name – the message is displayed unless you meet the validation rules. The form is validated on both the client and server side.

If you don't use nette/sandbox, you need to link _netteForms.js_ in order to enable JavaScript validation. You can find the file in _src/assets_ folder.

    <script src="/path/to/netteForms.js"></script>

Nette Framework adds `required` class to all mandatory elements. Adding the following style will turn label of _name_ input to red.

    <style>.required label { color: maroon }</style>

Even though setting classes might be handy, we need solid data, don't we? We will add yet another validation rule, this time with `addRule()` method. First argument sets what should the value look like, second one is the error message again, shown if the validation is not passed. Preset validation rules will do this time, but you will learn how to create your own in no time.

Form will get another input _age_ with condition, that it is optional, it has to be a number (`Form::INTEGER`) and in certain boundaries (`Form::RANGE`). This time we will utilize a third argument of `addRule()`, the range itself:

    $form->addText('age', 'Age:')    ->setRequired(false)    ->addRule(Form::INTEGER, 'Your age must be an integer.')    ->addRule(Form::RANGE, 'You must be older 18 years and be under 120.', [18, 120]);

Obviously, room for a small refactoring is available. In the error message and in the third parameter, the numbers are listed in duplicate, which is not ideal. If we were creating a [multilingual forms](https://doc.nette.org/en/2.4/localization) and the message containing numbers would have to be translated into multiple languages, it would make it more difficult to change values. For this reason, substitute characters can be used in [this format](http://php.net/sprintf):

    ->addRule(Form::RANGE, 'You must be older %d years and be under %d.', [18, 120]);

Nette Framework supports HTML5, including new form elements. Thanks to that we can set the age input as numeric:

    $form->addText('age', 'Age:')    ->setHtmlType('number')    ...

In the most advance browsers, namely Google Chrome, Safari and Opera, tiny arrows are rendered next to the input. Safari for iPhone shows an optimized keyboard with numbers.

Let's return to the _password_ field, make it _required_, and verify the minimum password length (`Form::MIN_LENGTH`), again using the substitute characters in the message:

    $form->addPassword('password', 'Password:')    ->setRequired('Pick a password')    ->addRule(Form::MIN_LENGTH, 'Your password has to be at least %d long', 3);

We will add one more input, `passwordVerify`, where user will be prompted to enter his password once more, to check for typo. Using validation rules, we will check if both fields contain the same value (`Form::EQUAL`). Notice the dynamic third argument, which is in fact the `password` control itself:

    $form->addPassword('passwordVerify', 'Password again:')    ->setRequired('Fill your password again to check for typo')    ->addRule(Form::EQUAL, 'Password mismatch', $form['password']);

If the form would not be used for registration, but rather for editing records, it would be handy to set [default values](https://doc.nette.org/en/2.4/forms#toc-default-values).

That's a complete, fully working registration form with both client-side and server-side validation. Automatically treats _magic quotes_, checks for invalid UTF-8 strings etc. Everything is ready and without a slightest effort on our side – Nette has taken care of it.

Examples are available to [download](https://files.nette.org/git/doc-2.4/form-example-en.php). Try adding some more [form fields](https://doc.nette.org/en/2.4/form-fields). Many inspiring forms are also in `/examples/Forms` of the distribution package.

## Form Fields[#](https://doc.nette.org/en/2.4/forms#toc-form-fields)

Besides wide range of [built-in form fields](https://doc.nette.org/en/2.4/form-fields) you can add custom fields to the form as follows:

    $form = new Form;$form['date'] = new DateInput('Date:');

Use the extension method to add a new method to object form:

    Nette\Forms\Container::extensionMethod('addZip', function ($form, $name, $label = null) {    return $form->addText($name, $label)        ->setRequired(false)        ->addRule($form::PATTERN, 'At least 5 numbers', '[0-9]{5}');});$form = new Form;$form->addZip('zip', 'ZIP code:');

Fields are removed using unset:

    unset($form['zip']);

## Default Values[#](https://doc.nette.org/en/2.4/forms#toc-default-values)

There are two ways to set default values. Method `setDefaults()` on a form or a container:

    $form->addText('name', 'Name:');$form->addInteger('age', 'Age:');$form->setDefaults([    'name' => 'John',    'age' => '33']);

or method `setDefaultValue()` on a single input:

    $form->addEmail('email', 'Email:')    ->setDefaultValue('user@example.com');

Default values of select-boxes and radio lists are set with the key from passed array of values:

    $form->addSelect('country', 'Country', [    'cz' => 'Czech republic',    'sk' => 'Slovakia',]);$form['country']->setDefaultValue('sk'); // country defaults to Slovakia

For CheckBox:

    $form->addCheckbox('agree', 'Agree with conditions')    ->setDefaultValue(true);

Another useful option is the “emptyValue”. If value of the form field after form submit is same as its “emptyValue” the field behaves as not filled.

    $form->addText('phone', 'Phone:')    ->setEmptyValue('+42');

## Disabling Inputs[#](https://doc.nette.org/en/2.4/forms#toc-disabling-inputs)

In order to disable an input, you can call `$control->setDisabled(true)`.

    $form->addEmail('email', 'E-mail:')->setDisabled(true);

The input cannot be written to and its value won't be returned by `getValues()`.

If we need a read-only input (the input will be rendered as disabled but its value will be shown), we disable the input first and set the value afterwards. The reason is that setDisabled() method resets the value of the input.

    $form->addText('readonly', 'Readonly:')->setDisabled()->setValue('readonly value');

If you only need to remove input's value but not disabling it, you can use `setOmitted(true)`. This option is useful for omitting antispam inputs for example.

    $form->addText('antispam', 'Antispam:')->setOmitted(true);

## Low-level Forms[#](https://doc.nette.org/en/2.4/forms#toc-low-level-forms)

To add an item to the form, you don't have to call `$form->addXyz()`. Form items can be introduced exclusively in templates instead. This is useful if you, for example, need to generate dynamic items:

    {foreach $items as $item}    <p><input type=checkbox name="sel[]" value={$item->id}> {$item->name}</p>{/foreach}

After submission, you can retrieve the values:

    $values = $form->getHttpData($form::DATA_TEXT, 'sel[]');$values = $form->getHttpData($form::DATA_TEXT | $form::DATA_KEYS, 'sel[]');

In the first parameter, you specify element type (`DATA_FILE` for `type=file`, `DATA_LINE` for one-line inputs like `text`, `password` or `email` and `DATA_TEXT` for the rest). The second parameter matches HTML attribute `name`. If you need to preserve keys, you can combine the first parameter with `DATA_KEYS`. This is useful for `select`, `radioList` or `checkboxList`.

`getHttpData()` returns sanitized input. In this case, it will always be array of valid UTF-8 strings, no matter what is sent by the form. It's an alternative to working with `$_POST` or `$_GET` directly if you want to receive safe data.

## Cross-Site Request Forgery (CSRF) Protection[#](https://doc.nette.org/en/2.4/forms#toc-cross-site-request-forgery-csrf-protection)

Nette Framework protects your applications against [Cross-Site Request Forgery](https://doc.nette.org/en/2.4/vulnerability-protection#toc-cross-site-request-forgery-csrf) (CSRF) attacks. An attacker lures a victim on a webpage, which quietly performs a request to server the victim is logged into. The server would not recognize whether the user send the request willingly.

The protection is pretty simple:

    $form->addProtection('Security token has expired, please submit the form again');

Generating and validating authentication token can prevent this attack. It has a limited expiration time: a session lifespan. Thanks to that, it does not prevent using the application in multiple windows (as long as it is the same session). The first argument is the error message shown, if the token has expired.

This protection should be added to all form which change sensitive data.

## Forms in Presenters[#](https://doc.nette.org/en/2.4/forms#toc-forms-in-presenters)

When using forms in presenters, we use class [Nette\Application\UI\Form](https://api.nette.org/2.4/Nette.Application.UI.Form.html) which is an extension of `Nette\Forms\Form`.

### Using One Form in Multiple Presenters[#](https://doc.nette.org/en/2.4/forms#toc-using-one-form-in-multiple-presenters)

In a case when we need to use one form in multiple presenters, we have two options:

1.  either putting it into the presenters' hierarchy into a common ancestor and define a factory there
2.  or to define it in a separate factory class and create its instance in the presenters' factories.

Appropriate place for such class is e.g. `app/forms/SignInFormFactory.php`. Our factory class will look like this:

    use Nette\Application\UI\Form;class SignInFormFactory{    /**     * @return Form     */    public function create()    {        $form = new Form;        $form->addText('name', 'Name:');        // ...        $form->addSubmit('login', 'Log in');        return $form;    }}

Now in each presenters' factory, which use our form, we create a form instance using our form factory class using `create()` method:

    protected function createComponentSignInForm(){    $form = (new SignInFormFactory)->create();    $form['login']->caption = 'Continue'; // we can also modify our form    $form->onSuccess[] = [$this, 'signInFormSubmitted']; // and add event handlers    return $form;}

We could also process our submitted form on one place. It's as simple as moving our event handler to our factory class, renaming it from `signInFormSubmitted` to e.g. `submitted`. Alternatively, we can use an anonymous function as a handler:

    use Nette\Application\UI\Form;class SignInFormFactory{    /**     * @return Form     */    public function create()    {        $form = new Form;        $form->addText('name', 'Name:');        ...        $form->addSubmit('login', 'Log in');        $form->onSuccess[] = function (Form $form, \stdClass $values) {            // we process our submitted form here        };        return $form;    }}

### Submitting a Form[#](https://doc.nette.org/en/2.4/forms#toc-submitting-a-form)

If a form contains multiple submit buttons we need to distinguish, it is useful to bind the handlers to `onClick` event of the button, which is invoked immediately before `onSuccess`.

    $form->addSubmit('login', 'Log in')    ->onClick[] = [$this, 'signInFormSubmitted'];

If a form is submitted by pressing _enter_ key, the first submit button is invoked.

Handlers of `onSuccess` and `onClick` events are invoked only if the submitted values pass validation. You don't need to validate the input inside the handler functions. The form also has `onSubmit` event, which is invoked irrespectively of the validation.

Sometimes can be useful to reset the form to its state before submitting. It can be done by invoking `reset()` (available since nette/forms version 2.4.6) method on the form:

    $form->isSubmitted(); // true$form->reset(); // the form is now in the state as it was never submitted$form->isSubmitted(); // false

## Standalone Forms[#](https://doc.nette.org/en/2.4/forms#toc-standalone-forms)

Nette Forms can be used without the whole framework as a standalone package. The form is created easily like this:

    use Nette\Forms\Form;$form = new Form;$form->addText('name', 'Name:');$form->addPassword('password', 'Password:');$form->addSubmit('send', 'Sign up');echo $form; // renders the form

Such form is submitted through POST method to the same page. You can alter that easily:

    $form = new Form;$form->setAction('/submit.php');$form->setMethod('GET');...

We can find out whether the form was submitted and passed validation by calling `$form->isSuccess()`. If so, let's print out the data.

    if ($form->isSuccess()) {    echo 'Form was filled and submitted successfully';    $values = $form->getValues();    dump($values);}

You can access form items like you access array. In our case, you can find the first text input on index `$form['name']`.

It's recommended to redirect to other page after you have processed the data. You can avoid duplicate form submission this way.

### Rendering the Form[#](https://doc.nette.org/en/2.4/forms#toc-rendering-the-form)

Each element has `getLabel()` and `getControl()` methods which return HTML code of label and the element itself. Nette provides _getter_ and _setter_ property access as if you are [accessing the attribute itself](https://doc.nette.org/en/2.4/smartobject#toc-properties-getters-and-setters).

    <?php $form->render('begin') ?><?php $form->render('errors') ?><table><tr class="required">    <th><?php echo $form['name']->label // Calls getLabel() ?></th>    <td><?php echo $form['name']->control // Calls getControl()  ?></td></tr><tr class="required">    <th><?php echo $form['age']->label ?></th>    <td><?php echo $form['age']->control ?></td></tr>...</table><?php $form->render('end') ?>

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

    $sub1 = $form->addContainer('first');$sub1->addText('name', 'Your name:');$sub1->addEmail('email', 'Email:');$sub2 = $form->addContainer('second');$sub2->addText('name', 'Your name:');$sub2->addEmail('email', 'Email:');

# Form Validation

## Form Field Validation[#](https://doc.nette.org/en/2.4/form-validation#toc-form-field-validation)

The following rules are supported by most [built-in form fields](https://doc.nette.org/en/2.4/form-fields):

`Form::FILLED`is element filled?`Form::REQUIRED`alias of `Form::FILLED``Form::EQUAL`is the value equal to?`mixed $value` or `$values[]``Form::NOT_EQUAL`the value must not be equal to given value`mixed $value` or `$values[]``Form::IS_IN`checks if value is in array`mixed $value` or `$values[]``Form::IS_NOT_IN`checks if value is NOT in array`mixed $value` or `$values[]``Form::VALID`did input pass validation?  
`Form::BLANK`the item must not be filled

We can set a custom error message to all validation rules, or a [default one](https://api.nette.org/2.4/Nette.Forms.Validator.html#$messages) is used. [Multilingual forms](https://doc.nette.org/en/2.4/localization)' messages are automatically translated.

The following special substitute strings can be utilized in error message text:

%labelreplaced by label text%namereplaced by input identification%valuereplaced by submitted input value

Besides validation rules, conditions can be set. They are set much like rules, yet we use `addRule()`instead of `addCondition()` and of course we leave it without an error message (the condition just asks):

    $form->addPassword('password', 'Password:')    // if password is not longer than 5 characters ...    ->addCondition(Form::MAX_LENGTH, 5)        // ... then it must contain a number        ->addRule(Form::PATTERN, 'Must contain number', '.*[0-9].*');

Condition can be linked to a different element than the current one: Just replace `addCondition()` with `addConditionOn()` and specify the reference to another element as the first parameter. In the following case, the email will be required if the checkbox is ticked (ie. its Boolean value is `true`):

    $form->addCheckbox('newsletters', 'send me newsletters');$form->addEmail('email', 'Email:')    // if checkbox is checked ...    ->addConditionOn($form['newsletters'], Form::EQUAL, true)        // ... require email        ->setRequired('Fill your email address');

All conditions can be negated with `~` (a tilde), i.e. `addCondition(~Form::NUMBER, ...)` passes validation if field is not filled. Conditions can be grouped into complex structures with `elseCondition()` and `endCondition()` methods.

As you can see, language of validation rules and conditions is powerful. Even though all constructions work both server-side and client-side, in JavaScript.

We can add own validators. Methods `addRule()` and `addCondition()` do accept callback or lambda function as well:

    // user validation: checks if $item is divisible by $arg// note: this is a real function, not a method in the presenterfunction divisibilityValidator($item, $arg){    return $item->value % $arg === 0;}$form->addInteger('number', 'Number:')    ->addRule('divisibilityValidator', 'Number must be divisible by %d.', 8);

## JavaScript[#](https://doc.nette.org/en/2.4/form-validation#toc-javascript)

Validation rules are transferred to the JavaScript part over HTML5 attributes `data-nette-rules`, which contain JSON containing all rules and conditions. The validation itself is handled by another script, which hooks all `submit` events, iterates over all inputs and runs respective validations. Default implementation is `netteForms.js` file, which can be found at `src/assets`. All you have to do is link it with

    <script src="/path/to/netteForms.js"></script>

Custom validation rules can be added by extending `Nette.validators` object:

    <script>Nette.validators.divisibilityValidator = function(elem, args, val) {    return val % args === 0;};</script>

If our PHP validation callback is a static method in a class, we create the name for JavaScript by removing all backslashes `\` and by replacing the double colon by single underscore `_`, e.g. `App\MyValidator::divisibilityValidator` is converted to `AppMyValidator_divisibilityValidator`.

## Modifying Input Values[#](https://doc.nette.org/en/2.4/form-validation#toc-modifying-input-values)

Using `addFilter` method we can modify retrieved value before form is processed. We can combine `addFilter` with `addCondition` and `addConditionOn` methods.

    $form->addText('zip', 'Postcode:')    ->addCondition($form::FILLED)    ->addFilter(function ($value) {        return str_replace(' ', '', $value);    });

When we later access values in form processing, postcode won't contain any space.

## Postprocessing Errors[#](https://doc.nette.org/en/2.4/form-validation#toc-postprocessing-errors)

We often find that user's input is wrong even though each of the form's elements is technically valid. For example stumbling upon a duplicate key when inserting a new database row. Or invalid login credentials. In that case, we can add the error manually with `addError()` method. It can be called either on a specific input or on a form itself:

    try {    $values = $form->getValues();    $this->user->login($values->username, $values->password);    $this->redirect('Homepage:');} catch (Nette\Security\AuthenticationException $e) {    if ($e->getCode() === Nette\Security\IAuthenticator::INVALID_CREDENTIAL) {        $form->addError('Invalid password.');    }}

It's recommended to link the error directly to a form element because then the error will be displayed next to it, when using default renderer.

    $form['date']->addError("We apologize but this date has been already taken.");

## Whole Form Validation[#](https://doc.nette.org/en/2.4/form-validation#toc-whole-form-validation)

If you need custom validation functionality, you can define your own validation functions for the whole form and bind them to `onValidate` event. Typically, this is used to validate a combination of values:

    protected function createComponentSignInForm(){    $form = new Form;    ...    $form->onValidate[] = [$this, 'validateSignInForm'];    return $form;}public function validateSignInForm($form){    $values = $form->getValues();    if (...) { // validation logic        $form->addError('Password does not match.');    }}

You can bind multiple functions to the event. The function is considered successful if it doesn't add any error using `$form->addError()`.

## Disabling Validation[#](https://doc.nette.org/en/2.4/form-validation#toc-disabling-validation)

In certain cases, you need to disable validation. If a submit button isn't supposed to run validation after submitting (for example _Cancel_ or _Preview_ button), you can disable the validation by calling `$submit->setValidationScope([])`. You can also validate the form partially by specifying items to be validated.

    $form->addText('name')->setRequired();$details = $form->addContainer('details');$details->addInteger('age')->setRequired('age');$details->addInteger('age2')->setRequired('age2');$form->addSubmit('send1'); // Validates the whole form$form->addSubmit('send2')->setValidationScope(false); // Validates nothing$form->addSubmit('send3')->setValidationScope([$form['name']]); // Validates only 'name' input$form->addSubmit('send4')->setValidationScope([$form['details']['age']]); // Validates only 'age' input$form->addSubmit('send5')->setValidationScope([$form['details']]); // Validates 'details' container

`onValidate` event on the form is always invoked and is not affected by the `setValidationScope`. `onValidate` event on the container is invoked only when this container is specified for partial validation.

# Form Rendering

Forms' appearances can differ greatly. In fact, there are two extremes. One side is the need to render a set of very similar forms all over again, with little to none effort. Usually administrations and back-ends.

The other side are tiny sweet forms, every one being a piece of art. Their layout can best be written in HTML. Of course, besides those extremes there are many forms just in between.

## Default Renderer[#](https://doc.nette.org/en/2.4/form-rendering#toc-default-renderer)

Renderer automatically renders a form. It's set with `setRenderer()` method on a form and it gains control when calling `$form->render()` or `echo $form`. If we set no custom renderer, [Nette\Forms\Rendering\DefaultFormRenderer](https://api.nette.org/2.4/Nette.Forms.Rendering.DefaultFormRenderer.html) is used. All you have to write is:

    echo $form;

or in [Latte](https://latte.nette.org/en/):

    {control form}

and the form is alive. All input fields are rendered into a HTML table. The output could look like this:

    <table><tr class="required">    <th><label class="required" for="frm-name">Name:</label></th>    <td><input type="text" class="text" name="name" id="frm-name" value="" /></td></tr><tr class="required">    <th><label class="required" for="frm-age">Age:</label></th>    <td><input type="text" class="text" name="age" id="frm-age" value="" /></td></tr><tr>    <th><label>Gender:</label></th>    ...

_Nicely formatted, isn't it? :-)_

It's up to you, whether to use a table or not, and many web designers prefer different markups, for example a list. We may configure `DefaultFormRenderer` so it would not render into a table at all. We just have to set proper [$wrappers](https://api.nette.org/2.4/Nette.Forms.Rendering.DefaultFormRenderer.html#$wrappers). The first index always represents an area and the second one it's element. All respective areas are shown in the picture:

By default a group of `controls` is wrapped in `<table>`, and every `pair` is a table row `<tr>` containing a pair of `label` and `control` (cells `<th>` and `<td>`). Let's change all those wrapper elements. We will wrap `controls` into `<dl>`, leave `pair` by itself, put `label` into `<dt>` and wrap `control` into `<dd>`:

    $renderer = $form->getRenderer();$renderer->wrappers['controls']['container'] = 'dl';$renderer->wrappers['pair']['container'] = null;$renderer->wrappers['label']['container'] = 'dt';$renderer->wrappers['control']['container'] = 'dd';echo $form;

Results into the following snippet:

    <dl>    <dt><label class="required" for="frm-name">Name:</label></dt>    <dd><input type="text" class="text" name="name" id="frm-name" value="" /></dd>    <dt><label class="required" for="frm-age">Age:</label></dt>    <dd><input type="text" class="text" name="age" id="frm-age" value="" /></dd>    <dt><label>Gender:</label></dt>    ...</dl>

Wrappers can affect many attributes. For example:

*   add special CSS classes to each form input
*   distinguish between odd and even lines
*   make required and optional draw differently
*   set, whether error messages are shown above the form or close to each element

## Bootstrap Support[#](https://doc.nette.org/en/2.4/form-rendering#toc-bootstrap-support)

You can find [examples](https://github.com/nette/forms/tree/master/examples) of configuration forms for [Twitter Bootstrap 2](https://github.com/nette/forms/blob/a0bc775b96b30780270bdec06396ca985168f11a/examples/bootstrap2-rendering.php#L58) and [Bootstrap 3](https://github.com/nette/forms/blob/a0bc775b96b30780270bdec06396ca985168f11a/examples/bootstrap3-rendering.php#L58)

## Manual Rendering[#](https://doc.nette.org/en/2.4/form-rendering#toc-manual-rendering)

You can render forms manually for better control over the generated code. Place the form inside `{form myForm}` and `{/form}` pair macros. Inputs can be rendered using `{input myInput}` macro, which renders the input, and `{label myInput /}`, which renders its the label.

    {form signForm}<!-- Simple errors rendering --><ul class="errors" n:if="$form->hasErrors()">    <li n:foreach="$form->errors as $error">{$error}</li></ul><table><tr class="required">    <th>{label name /}</th>    <td>{input name}</td></tr><!--If you need manualy render radiolist --><p>{input radioList:itemKey} | {input radioList:itemKeyTwo}</p>...</table>{/form}

You can also connect a form with a template easily by using `n:name` attribute.

    function createComponentSignInForm(){    $form = new Form;    $form->addText('user')->setRequired();    $form->addPassword('password')->setRequired();    $form->addSubmit('send');    return $form;}

    <form n:name=signInForm class=form>    <p><label n:name=user>Username: <input n:name=user size=20></label>    <p><label n:name=password>Password: <input n:name=password></label>    <p><input n:name=send class="btn btn-default"></form>

You can also use `n:name` with `<select>`, `<button>` or `<textarea>` elements and the content.

You can render RadioList, Checkbox or CheckboxList by HTML elements individually. This is called partial rendering:

    {foreach $form[gender]->items as $key => $label}    <label n:name="gender:$key"><input n:name="gender:$key"> {$label}</label>{/foreach}

Or you can use basic macros `{input gender:$key}` and `{label gender:$key}`. The trick is using the colon. For a simple checkbox, use `{input myCheckbox:}`.

Macro `formContainer` helps with rendering of inputs inside a form container.

    {form signForm}<table>    <th>Which news you wish to receive:</th>    <td>        {formContainer emailNews}        <ul>            <li>{input sport} {label sport /}</li>            <li>{input science} {label science /}</li>        </ul>        {/formContainer}    </td>    ...</table>{/form}

How to set more attributes to HTML elements? Methods `getControl()` and `getLabel()` return element as a `Nette\Utils\Html` objects, which can be [easily adjusted](https://doc.nette.org/en/2.4/html-elements). In Latte:

    {form signForm class => 'big'}<table><tr class="required">    <th>{label name /}</th>    <td>{input name cols => 40, autofocus => true}</td></tr>

## Grouping Inputs[#](https://doc.nette.org/en/2.4/form-rendering#toc-grouping-inputs)

Input fields can be grouped into visual field-sets by creating a group:

    $form->addGroup('Personal data');

Creating new group activates it – all elements added further are added to this group. You may build a form like this:

    $form = new Form;$form->addGroup('Personal data');$form->addText('name', 'Your name:');$form->addInteger('age', 'Your age:');$form->addEmail('email', 'Email:');$form->addGroup('Shipping address');$form->addCheckbox('send', 'Ship to address');$form->addText('street', 'Street:');$form->addText('city', 'City:');$form->addSelect('country', 'Country:', $countries);

## HTML Attributes[#](https://doc.nette.org/en/2.4/form-rendering#toc-html-attributes)

You can also extend the forms with labels, `classes` and do other things.

Pay attention to the order of conditions, rules and usage of the extensions.

Setting class or JavaScript attributes:

    $form->addInteger('number', 'Number:')    ->setHtmlAttribute('class', 'bigNumbers');$form->addSelect('rank', 'Order by:', ['price', 'name'])    ->setHtmlAttribute('onchange', 'submit()'); // calls JS function submit() on change// applying on whole $form$form->getElementPrototype()->id = 'myForm';$form->getElementPrototype()->target = '_top';

Setting input type (HTML5):

    $form->addText('email', 'Your e-mail:')    ->setHtmlType('email')    ->setHtmlAttribute('placeholder', 'Please, fill in your e-mail address');

Setting description (rendered after the input by default):

    $form->addText('phone', 'Number:')    ->setOption('description', 'This number will remain hidden');

In order to add HTML content, you can use [Html](https://doc.nette.org/en/2.4/html-elements) class.

    use Nette\Utils\Html;$form->addText('phone', 'Phone:')    ->setOption('description', Html::el('p')        ->setHtml('This number remains hidden. <a href="...">Terms of service.</a>')    );

Html element can be also used instead of label: `$form->addCheckbox('conditions', $label)`.