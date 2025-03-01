---
group: php-developer-guide
subgroup: Security
title: XSS prevention strategies
menu_title: XSS prevention strategies
menu_order: 1100
migrated_to: https://developer.adobe.com/commerce/php/development/security/cross-site-scripting/
layout: migrated
---

[Cross-site scripting][], or XSS, is a security vulnerability that can be found in web applications. This vulnerability allows attackers to inject malicious code/styles into a web page viewed by users. Magento [extension](https://glossary.magento.com/extension) developers should be aware of these vulnerabilities to avoid introducing them in their code.

There are three main types of XSS vulnerabilities:

-  **Persisted XSS** - In this type of vulnerability, the source of unvalidated data comes from the database or [Backend](https://glossary.magento.com/backend) permanent store.
-  **Reflected (non-persistent) XSS** - This type of vulnerability occurs when data provided by a web client is used immediately by server-side scripts to parse and display a page to a user without properly sanitizing the request.
-  **DOM XSS** - For this vulnerability, the malicious data does not touch the web server. Rather, it is being reflected by the [JavaScript](https://glossary.magento.com/javascript) code, fully on the client side.

## Preventing XSS

XSS vulnerabilities can be prevented by validating and sanitizing user input as well as sanitizing dynamic values when rendering the view (HTML, mobile).

### Input Processing

Any request data can be manipulated by attackers and can contain malicious values such as:

-  form fields filled with control characters ("&lt;", ">" etc)
-  headers contain false IDs
-  URIs contain fake parts/query parameters
-  tampered cookies

To combat this, developers must validate any value coming in from requests.

It is better to validate/sanitize values as close as possible to the view context because only then you can be sure of the restrictions you have to impose on dynamic values and you are not risking security requirements for buisness requirements.

There is no reason, from a business standpoint, to disallow `<` `>` symbols in your users' "About me" section. By escaping control symbols when rendering HTML, allowing these characters would not be problematic. "About me" data may be delivered via RESTful API, where `{}` could cause issues. If sanitized earlier, the user data would be damaged and contain HTML control symbols (`< >`).

### Output Processing

Output processing involves sanitizing strings that may have come from external data sources before using it to render views. It is the main method of protecting your extension from XSS attacks.

The general rule is: Do not trust dynamic values.

#### PHTML templates

The `\Magento\Framework\Escaper` class is provided for `.phtml` templates and PHP classes responsible for generating HTML. It contains HTML sanitization methods for a variety of contexts.

The `$escaper` local variable is available inside the .phtml templates. See the [product listing template][] as example of `$escaper` usage in .phtml templates.

See [Template guide][] to read more about templates in Magento.

When using the `\Magento\Framework\Escaper` or `$escaper`:

-  If a method indicates that the content is escaped, do not escape: getTitleHtml(), getHtmlTitle() (the title is ready for HTML output)
-  Type casting and the php function `count()` do not need escaping (for example `echo (int)$var`, `echo (bool)$var`, `echo count($var)`)
-  Output in single quotes does not need escaping (for example echo 'some text')
-  Output in double quotes without variables does not need escaping (for example echo "some text")
-  For all other cases, escape the data using context-specific escape functions.

**The following code sample illustrates XSS-safe output in templates:**

```php
<?php echo $block->getTitleHtml() ?>
<?php echo $block->getHtmlTitle() ?>
<?php echo $escaper->escapeHtml($block->getTitle()) ?>
<?php echo (int)$block->getId() ?>
<?php echo count($var); ?>
<?php echo 'some text' ?>
<?php echo "some text" ?>
<a href="<?php echo $escaper->escapeUrl($block->getUrl()) ?>"><?php echo $block->getAnchorTextHtml() ?></a>
```

**When to use Escaper methods:**

**Case**: JSON inside an HTML attribute

**Escaper method**: escapeHtmlAttr

```php
<div data-bind='settings: <?= $escaper->escapeHtmlAttr($myJson) ?>'></div>
```

**Case**: JSON inside script tag

**Escaper method**: _no sanitization needed_

```php
<script>
let settings = <?= $myJson ?>
</script>
```

**Case**: HTML tag content that should not contain HTML

**Escaper method**: escapeHtml

Pass in an optional array of allowed tags that will not be escaped.

If a tag is allowed, the following attributes will not be escaped: `id`, `class`, `href`, `style` and `title`. Any other attribute for that allowed tag will be escaped.

`embed`, `iframe`, `video`, `source`, `object`, `audio`, `script` and `img` tags are not allowed, regardless of the content of this array.

```php
 <span class="label"><?php echo $escaper->escapeHtml($block->getLabel()) ?></span>
  // Escaping translation
  <div id='my-element'>
      <?php echo $escaper->escapeHtml(__('Only registered users can write reviews. Please <a href="%1">Sign in</a> or <a href="%2">create an account</a>', $block->getLoginUrl(), $block->getCreateAccountUrl()), ['a']) ?>
  </div>
```

**Case**: URL inside certain HTML attributes

**Escaper method**: escapeUrl

Certain attributes like `a.href` accept URIs of various types and must be sanitized.

```php
<a href="<?= $escaper->escapeUrl($myUrl) ?>">Click me</a>
<div attr-js-extracts="<?= $escaper->escapeHtmlAttr($myOtherUrl) ?>"></div>
```

**Case**: All JavaScript inside attributes must be escaped by escapeJs before escapeHtmlAttr:

**Escaper method**: escapeJS

```php
<div
    onclick="<?= $escaper->escapeHtmlAttr('handler("' . $escaper->escapeJs($aString) . '", ' . $escaper->escapeJs($aVar) .')') ?>">
    My DIV
</div>
```

**Case**: JavaScript string that must not contain JS/HTML

**Escaper method**: escapeJS

```php
<script>
let phrase = "Hi, my name is <?= $escaper->escapeJs($myName) ?>";
//Do not use HTMl context methods like escapeUrl
let redirectUrl = "<?= $escaper->escapeJs($myUrl) ?>";
location.href = redirectUrl;
</script>
```

**Case**: JavaScript variable that must not contain JS/HTML

**Escaper method**: escapeJS

```php
<script>
let <?= $escaper->escapeJs($dynamicVariable) ?> = <?= $myJson ?>;
settings.<?= $escaper->escapeJs($myProperty) ?> = true;
</script>
```

#### Knockout templates

In knockout templates, you can bind a UI component's property/function as the 'inner HTML' of an element. Such properties may contain dynamic data and must be sanitized within componenets. See [Magento binding syntax][] to learn more about UI component templates.

In order to notify developers that these properties/function results may contain HTML, Magento requires (with the help of a static test) that you name such properties/functions using "UnsanitizedHtml" suffix.

```html
<div data-bind="html: propUnsanitizedHtml"></div>
<p html="returnUnsanitizedHtml()"></p>
```

#### Dynamically created DOM elements

When using variables that are not supposed to contain HTML, the safest way to generate DOM elements is to create them programatically using the appropriate API, instead of using the _innerHtml_ property or jQuery's _.html()_ function.

```javascript
let newDiv = document.createElement("div");
newDiv.innerText = userName;
newDiv.setAttribute("custom-attribute", myAttribute);
parentElement.appendChild(newDiv);
```

#### UI component data providers

UI component data providers pass dynamic (user generated) data to UI components. The data they return is rendered in order to support component dynamic linking. Since user data is supposed to be treated as a literal value which is not refering to any other component, rendering of these properties must be disabled. See [UI component data providers][] to read more about data providers and component linking.

```php
$uiData = ['linkProperty' => '${ $.otherComponent.value }'];
$uiData['customer'] = $customer->getData();
//Customer data will be taken as is, linkProperty will be retrieved from otherComponent and taken as is
$uiData['__disableTmpl'] = ['customer' => true, 'linkProperty' => 1];

return $uiData;
```

### Static Test

To check your .phtml template for XSS vulnerabilities, use the _Magento2.Security.XssTemplate_ sniff from [Magento Coding Standard][].
This sniff finds all _echo_ calls in PHTML-templates and determines if the output is properly escaped.
It covers the following cases:

-  `/_ @noEscape _/` before output. Output does not require escaping. Test is green.
-  `/_ @escapeNotVerified _/` before output. Output escaping is not checked and should be verified. Test is green.
-  Methods which contain "html" in their names (for example echo $object->{suffix}Html{postfix}()). Data is ready for the HTML output. Test is green.
-  AbstractBlock methods `escapeHtml`, `escapeHtmlAttr`, `escapeUrl`, `escapeJs` are allowed. Test is green.
-  Type casting and php function `count()` are allowed (for example `echo (int)$var`, `(bool)$var`, `count($var)`). Test is green.
-  Output in single quotes (for example echo 'some text'). Test is green.
-  Output in double quotes without variables (for example echo "some text"). Test is green.
-  Other of previously mentioned. Output is not escaped. Test is red.

<!-- Link Definitions -->
[product listing template]: {{ site.mage2bloburl }}/78bb169ff9721c8d05c35b4c29a4464fd45bccbe/app/code/Magento/Catalog/view/frontend/templates/product/list.phtml#L16
[Template guide]: {{ page.baseurl }}/frontend-dev-guide/templates/template-overview.html
[Cross-site scripting]: https://owasp.org/www-community/attacks/xss/
[Magento binding syntax]: {{ page.baseurl }}/ui_comp_guide/concepts/magento-bindings.html
[UI component data providers]: {{ page.baseurl }}/ui_comp_guide/concepts/ui_comp_data_source.html
[Magento Coding Standard]: https://github.com/magento/magento-coding-standard
