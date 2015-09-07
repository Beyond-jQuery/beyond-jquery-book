# Using and Understanding HTML Element Attributes {#element-attributes}

Here in chapter five, prepare yourself for a full-scale, in-depth, no-holds-barred discussion on everything related to element attributes. Everything you learned in the ["finding elements" chapter](#finding-elements) will prove to be useful as you apply this knowledge during your journey through attributes. I'll make sure you have a [proper understanding of attributes](#what-are-attributes), how they came about and [became part of HTML, and how they fit into the web API](#attribute-history). Additionally, you'll learn how to [use attributes to locate DOM elements](#finding-elements-with-attributes). While this was covered a bit in the ["finding elements" chapter](#finding-elements), you'll find that the coverage here is much more comprehensive. And finally, I'll dive deeper into attributes and demonstrate how you can [read, add, and update them on any DOM element](#reading-attributes). Special sections on [`data-`](#data-attributes) and [`class`](#class-attributes) attributes will be included as well.

Other than a couple exceptions, all of the web API code in this chapter has full support across _all_ [modern browsers](#modern-browsers) _and_ even Internet Explorer 8. After you complete this chapter, you will not only have a complete understanding of attributes, but you will also have the confidence to read them, modify them, _and_ use them to select elements in all browsers, even those as old as Internet Explorer 8. Read on to continue your quest to move Beyond jQuery!


## What is an attribute? {#what-are-attributes}

HTML elements, declaratively speaking, are made up of three parts: name, content, and attributes, with the last two being optional. Take a look at the following simple fragment, which I'll reference as I explain these three parts a bit more.

{title="html tags are made up of three parts", lang=html}
~~~~~~~
<form action="/rest/login.php" method="POST">
   <input name="username" required>
   <input type="password" name="password" required>
</form>
~~~~~~~

In the above markup, you see three elements tags: one `<form>` and two `<input>`s. The `<form>` element has a tag name of "form". In fact, `tagName` is a property available on every object in the DOM that implements the `Element` interface. This property was [standardized as part of W3C's DOM Level 2 Core specification][element.tagname]. In my above HTML, the `<form>` element, represented as a [`HTMLFormElement` object][html5-form], has a `tagName` property with a value of "form". The two `<input>` elements are represented as [`HTMLInputElement` objects][html5-input], and unsurprisingly they each have `tagName` values of "input".

Content, the second part of an element, describes any other nodes that are are descendants of an element. My example `<form>` has two `<input>` elements as content, while the two `<input>` elements have _no_ content. In fact, `<input>` elements are not allowed to have any content. This restriction was likely in place since `<input>` elements were first introduced in [the HTML 2 specification][html2], but was only first [explicitly mentioned in the HTML 3 official standard document][html3-input].

I> ## A note about my example form markup
I>
I> Normally you would want to associate each form field with a `<label>` that contains a text node with the field's display name. Also, a submit button is usually prudent, but I left all of these out of my fragment to keep it simple and focused on the discussion of attributes.

Attributes, the third and final part of an element, also optional, provide a way to annotate elements directly in your markup. You may use them to provide data or state. For example, the `<form>` element above contains two such attributes: `action` and `method`, which together tell the form to send a POST request (`method`) to the "/rest/login.php" server endpoint (`action`) when the form is submitted. The first input has a `name` attribute of "username" and the second has a `name` of "password". This information is used to construct the request and tie these elements to their values when the server parses the form submit. While not evident in the above HTML, you can even create your own proprietary attributes and reference them in your code for the purposes of associating state or data with elements in your markup. Though not strictly required, the more standard way to do this is with `data-` attributes, which [will be mentioned later on in this chapter](#data-attributes).

In addition to providing data or state, some attributes are used to define specific behaviors for multi-purpose elements. Take a look at the `type` attribute on the second input in the above fragment, as an example of this. This `type` attribute defines the second input to be a password input, which signals the browser to mask any characters entered into this field by the user. The first input _could_ include a `type` attribute with a value of "text", but this is not necessary as all `<input>` elements are, by default, text inputs. This default has been in place since the inception of HTML, and [is visible in one of the earliest drafts of the specification][html1-input]. Another example of a behavior imposed by an attribute can be seen on both of the inputs above. Notice the `required` attributes on each of the inputs - this is a signal to any browser that supports [the `constraints API`][html5-constraints] to prevent form submission if either of these fields are left empty by the user.


### History and standardization {#attribute-history}

Attributes have always been a part of HTML, and were [described in the first document that details HTML tags][html-tags], written by Tim Berners-Lee in 1992. In this article, Berners-Lee describes the same two general types of attributes that are used in HTML today  - boolean, and variable - both of which I will elaborate on further shortly. The passage that calls out attributes is near the start of the document:

> Some tags take parameters, called attributes. The attributes are given after the tag, separated by spaces. Certain attributes have an effect simply by their presence, others are followed by an equals sign and a value.

Berners-Lee went on to mention a few such attributes, using the `href`, `type`, and `name` attributes of anchor tags as an example. Note though that the `name` attribute on `<a>` elements is no longer available as [it was removed in the HTML5 specification][html5-anchor]. Since this first description of HTML, the number and importance of element attributes has increased greatly.

Unbounded custom attributes are not, and have never been _officially_ supported in any HTML specification. But you may prefix an attribute name of your choice with "data-" as of the HTML5 spec - more on that [later](#data-attributes). However, if you would like to introduce a purely custom attribute into your markup, such as "myproject-uuid", you are certainly free to do so. The page will render without issue, and there will be no errors in your browser's developer tools console. Everything will work just fine. The only drawback is that your document will fail validation, as it will contain non-standard attributes - attributes that are not mentioned in any accepted HTML standard. Non-standard custom attributes are actually quite common, and even prevalent in popular JavaScript frameworks, such as AngularJS, which relies on non-standard custom attributes heavily to facilitate communication with element directives.

{#boolean-attributes}
The latest _completed_ HTML spec - HTML5 - defines [four different types of attributes][html5-attribute-syntax]. One type commonly known as a ["boolean attribute"][html5-boolean-attribute] is expressed as an element attribute without any explicit value. Take the `required` attribute commonly found on `<input>` elements as one example (seen in the [previous section's](#what-are-attributes) HTML fragment). As the specification states, "The presence of a boolean attribute on an element represents the true value, and the absence of the attribute represents the false value." The HTML 5.1 standardized [`hidden` attribute][html51-hidden], which instructs the browser to _not_ render any element bearing the attribute, is another example of this first type.

A second type of attribute is described as "unquoted". A little known fact is that you may omit quotes around your attribute values, provided the attribute value does not contain spaces, equal signs, or angle brackets (`<` and `>`), or an empty string (among other less notable character restrictions). So, the HTML fragment in the previous section can be re-written as follows:

{title="unquoted attributes are perfectly legal", lang=html}
~~~~~~~
<form action=/rest/login.php method=POST>
   <input name=username required>
   <input type=password name=password required>
</form>
~~~~~~~

{#valid-attribute-values}
The final two types of HTML element attributes are very similar: single-quoted and double-quoted. Both are similar in that they largely have the same restrictions, and are much more common than unquoted attributes. Though, double-quoted attributes are arguably the _most_ common of all types. In contrast to unquoted attribute values, those that are surrounded by either single or double quotation marks _may_ contain spaces, equal signs, angle brackets, or an empty string. The portion of the latest W3C specification that describes attribute values mentions only that they may never contain any ambiguous ampersand characters. An "ampersand character" is an `&` sign followed by a standardized ASCII character code and terminated with a semicolon (`;`). An "ambiguous ampersand character" is one where this ASCII character code does _not_ match any of the character codes defined in the [named character references][html5-named-character-references] portion of the spec.


### How do attributes differ from properties? {#attributes-vs-properties}

Now that you have a solid understanding of what element attributes are, you may _still_ be confused regarding their relation to element "properties", _especially_ if you have been [using jQuery for a while][jquery-16]. At a very basic level, properties and attributes are entirely different from each other. While attributes are declared at the HTML level in the element's markup, properties are declared and updated on the element's object representation. For example, consider the following element:

{title="element for attributes vs properties explanation", lang=html}
~~~~~~~
<div class="bold">I'm a bold element</div>
~~~~~~~

The `<div>` has a `class` attribute with a value of "bold". But we can set properties on this element too. Suppose we want to set a property of `index` with a value of `0`:

{title="setting an element property", lang=html}
~~~~~~~
<div class="bold">I'm a bold element</div>

<script>
   document.querySelector('.bold').index = 0;
</script>
~~~~~~~

{#expando-properties}
After executing the above fragment, our `<div>` now has a `class` attribute with a value of "bold" _and_ a property of `index` with a value of `0`. The property is set on the underlying JavaScript object, which in this case is an implementation of the [`HTMLDivElement`][htmldivelement] interface. Element object properties such as our `index` are also known as "expando" properties, which is just a terse way to classify non-standard element object properties. Understand that not _all_ element properties are expando properties. Don't worry if this is still not entirely clear. I'll talk more about standardized element properties before this section is complete.

While properties and attributes are conceptually and syntactically different, they are very closely linked together in some cases. In fact, all standardized element attributes have corresponding properties defined in the element's object representation. For the most part, each standard attribute and property pair share the same value. And in all but one case, the attribute and property share the same name as well. These standardized attributes are special in that you can update them without touching the markup. Updating the corresponding element object's property value will cause the browser to update the attribute's value in the document, and updating the attribute value will in turn up the element's property value. Let's take a look at a simple example, where we define an anchor link with an initial `href`, and then update the anchor to point to a different location using JavaScript:

{title="standard element attributes and properties are linked", lang=html}
~~~~~~~
<a href="http://www.widen.com/blog/">Read the Widen blog</a>

<script>
   document.querySelector('A').href = 'http://www.widen.com/blog/ray-nicholus';
</script>
~~~~~~~

After executing the script in the above code block, the anchor now appears in the document as follows:

{title="after updating the href property in the previous example", lang=html}
~~~~~~~
<a href="http://www.widen.com/blog/ray-nicholus">Read the Widen blog</a>
~~~~~~~

In this case, the [`HTMLAnchorElement`][html51-a], which is the object representation of an `<a>`, has an `href` property defined on its prototype that is directly connected to the `href` attribute on the element tag. This `href` property is actually inherited from the [`URLUtils` interface][whatwg-urlutils], which the `HTMLAnchorElement` object also implements. `URLUtils` is an interface formally defined in the [WHATWG URL Living Standard][whatwg-url] specification.

There are many other element attributes with connected properties, such as `id` (all elements), `action` (form elements), and `src` (script elements), to name a few. Remember that all attributes that appear in the HTML specifications fall into this category. But there are a few special cases and points to consider. First, `class` attributes are a bit different in that the corresponding property name is not `class`, but `className`. This is due to the fact that ["class" is a reserved word in JavaScript][es6-reserved-words]. More on the `class` attribute [later on](#class-attributes). Also keep in mind that the `checked` attribute, common to radio and checkbox input elements, is only _initially_ connected to the corresponding element property value. Consider the following code to demonstrate this limitation a bit more clearly:

{title="the checked boolean attribute is a bit unusual", lang=html}
~~~~~~~
<input type="checkbox" checked>

<script>
    // this does not remove the checked attribute
    document.querySelector('INPUT').checked = false;
</script>
~~~~~~~

After executing the above script, you may expect the `checked` attribute to be removed from the input element, since this would happen for other boolean attributes, such as `required` and `disabled`. However, the `checked` attribute remains on the element, even though the property value has been changed to `false` and the checkbox is indeed unchecked.

"Custom" attributes, that is, attributes that are _not_ defined in any accepted specification, are not linked in any way to a similarly named property on the element object. Any properties you create to match non-standard attributes are also considered expando properties.


## Finding elements using attributes {#finding-elements-with-attributes}

Building upon the [class](#selecting-classes) and [ID](#selecting-ids) selector examples from [the previous chapter](#finding-elements), this section is going to provide a much more comprehensive guide to selecting any and _all_ attributes using the web API. While ID and class attribute selection is commonly accomplished using a selector syntax specific to these two types of attributes, you are able to use the more general attribute selection approaches found in this chapter as well. In some cases, some of the generic but powerful attribute selectors demonstrated here are most appropriate when looking for multiple elements that follow a known ID or class pattern.

For the sake of consistency and reference, jQuery examples will be provided throughout this section. But attributes can be selected in a number of ways without jQuery, simply by using either `querySelector` or `querySelectorAll`. Since attribute selectors were first introduced as [part of the W3C CSS 2 specification][css2-attributes] **all of the simple web API examples here are supported all the way back to Internet Explorer 8**! You truly don't need jQuery to write simple but powerful attribute selectors.


### Finding elements using attribute names {#find-by-attribute-name}

We'll cover values in quite a bit of detail shortly, but let's first focus on attribute names. Why might you want to only focus on an attribute name? Perhaps for many reasons, including:

1. Locating `disabled` elements or `required` form fields.
2. Finding one or more elements that include a custom attribute that groups these elements in some way.
3. Locating invalid markup in a document, such as `<img>` elements without a `src` attribute.

The following jQuery and web API examples will revolve around #1 above. For this, a small HTML fragment will be used for reference:

{title="selecting attributes by name - sample HTML", lang=html}
~~~~~~~
<form action="/submitHandler.php" method="POST">
    <input name="first-name">
    <input name="last-name" required>
    <input type="email" name="email" required>
    <button disabled>submit</button>
</form>
~~~~~~~


#### jQuery

There is one way to select elements given their attributes in jQuery, and that is by passing a valid CSS 2+ attribute selector string into the jQuery function:

{title="selecting by attribute name - jQuery", lang=javascript}
~~~~~~~
var $result = $('[required], [disabled]');
~~~~~~~

The above code will result in a `$result` jQuery object that contains the "last-name" and "email" `<input>` elements, along with the disabled submit `<button>`. In case the comma in the selector string is causing you some confusion, I covered this in [the previous chapter's multiple element selector section](#multiple-selectors). This jQuery code relies entirely on the web API.


#### Web API

As with many of the native solutions in the ["finding elements" chapter](#finding-elements), the code required to locate elements using attribute names is eerily similar to the jQuery solution you have just seen.

{title="selecting by attribute name - web API - all modern browsers + IE8", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('[required], [disabled]');
~~~~~~~

Similar to the jQuery example, the above code will populate the `result` variable with a `NodeList` containing the "last-name" and "email" inputs, along with the disabled submit button.

{#empty-class-attribute}
While `disabled` and `required` are [boolean attributes](#boolean-attributes), the above code will yield the same results even if we assigned them values. The attribute selector simply matches on the attribute name - the value (or lack of one) is irrelevant. This means you can easily locate all elements in a document that have _any_ CSS classes assigned to them. For example, look at the following simple attribute selector:

{title="selecting all elements with a class attribute - modern browsers + IE8", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('[class]');
~~~~~~~

Given the following HTML:

{title="elements with and without CSS classes", lang=html}
~~~~~~~
<div class="bold">I'm bold</div>
<span>I'm not</span>
~~~~~~~

...the `result` variable in the previous selector will yield one element - the `<div>`. But beware, simply adding an _empty_ `class` attribute to the `<span>` may result in an unexpected result set:

{title="elements with and without CSS classes", lang=html}
~~~~~~~
<div class="bold">I'm bold</div>
<span class>I'm not</span>
~~~~~~~

Even though the `<span>` does not have any CSS classes assigned to it, the mere presence of the `class` attribute means that our selector includes it in the result set alongside the `<div>`. This is probably not what we wanted. This is not a deficiency of the selector API, but it _is_ important to understand exactly how attribute name selectors function. Note that you will run into the same "problem" using jQuery if you don't have a firm grasp of this CSS selector.


### Finding elements using attribute names _and_ values {#attribute-names-and-values}

Sometimes locating an element or group of elements by attribute name alone is not sufficient. You may want to, for example, locate all password input fields, in which case you would need to find all `<input>` elements with a `type` attribute of "password". Or perhaps you need to locate all anchor elements that link to a specific endpoint, in which case you'd need to key on the desired value of the `href` attribute of all `<a>` elements.

To setup our jQuery and web API examples, let's use the following HTML and state that our goal is to locate all anchors that link to the ajax-form web component documentation page.

{title="elements with and without CSS classes", lang=html}
~~~~~~~
<section>
   <h2>web components</h2>
   <ul>
       <li>
           <a href="http://file-input.raynicholus.com/">file-input</a>
       </li>
       <li>
           <a href="http://ajax-form.raynicholus.com/">ajax-form</a>
       </li>
   </ul>
</section>
<section>
   <h2>no-dependency libraries</h2>
   <ul>
      <li>
          <a href="http://ajax-form.raynicholus.com/">ajax-form</a>
      </li>
      <li>
          <a href="http://fineuploader.com/">Fine Uploader</a>
      </li>
   </ul>
</section>
~~~~~~~


#### jQuery

In order to find all anchor elements that point to the ajax-form library page, we'll use a standardized CSS selector string passed into the `jQuery` function, as we've seen so many times before:

{title="find anchors with specific href attributes - jQuery", lang=javascript}
~~~~~~~
var $result = $('A[href="http://ajax-form.raynicholus.com/"]');
~~~~~~~

The above selector returns a jQuery object containing the two ajax-form `HTMLAnchorElement` objects from our example markup.


#### Web API

You've already seen how a standard CSS selector is required to select by attribute name and value when using jQuery, so of course the same selector is most appropriate when attempting to find specific anchor elements without jQuery. The solution here is, as you have seen in most other element selection examples, almost identical to the jQuery approach, but more efficient:

{title="find anchors with specific href attributes - web API - modern browsers + IE8", lang=javascript}
~~~~~~~
var result =
    document.querySelectorAll('A[href="http://ajax-form.raynicholus.com/"]');
~~~~~~~

The `result` variable is a `NodeList` containing both ajax-form anchors from our example HTML at the start of this section. Notice that I'm combining the attribute name/value selector with a [tag name selector](#tag-selector). This ensures that any other elements that may include a non-standard `href` attribute are ignored (along with any `<link>` elements), since we are only concerned with anchor links.

Remember the [empty class attribute example](#empty-class-attribute) from the selecting by attribute name section? During our search for all elements with CSS classes, we were unable to ignore empty `class` attributes with a simple attribute name selector. But if we pair an attribute name/value selector with the [exclusion selector from the "finding elements chapter"](#excluding-elements), we can effectively filter out empty `class` attributes:

{title="find anchors with specific href attributes - web API - modern browsers", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('[class]:not([class=""]');
~~~~~~~

Using the sample HTML from the initial [empty class attribute example](#empty-class-attribute) section, the above code block, `result` contains a `NodeList` containing _only_ the `<div>` with a `class` attribute of "bold". The `<span>` with an empty `class` attribute has been successfully skipped over.


### The power of wildcard and fuzzy attribute selectors

This last part of the attribute selectors section focuses on more advanced use cases. Below, I'll demonstrate four very powerful attribute selector tricks that are also easy to understand _and_ supported in all modern browsers as well as Internet Explorer 8. The pattern you have already seen (many times) between the jQuery and the web API selector code continues in this last set of examples. So, let's just forgo the jQuery vs. web API code snippets, as they are mostly redundant when discussing element selectors. If you really want to run the following examples "the jQuery way" just replace `document.querySelectorAll()` with `$()` and be prepared for your code to run a bit slower.


#### Looking for specific characters {#substring-attribute-selector}

Remember the example from the section on [attribute names and values](#attribute-names-and-values)? We wanted to locate any anchor links in a document that pointed to a very specific endpoint. But what if we don't care about the entire URL? What if we are only concerned with the domain? Consider the following HTML fragment:

{title="multiple anchors to different domains with varying paths", lang=html}
~~~~~~~
<a href="http://fineuploader.com/">home page</a>
<a href="http://fineuploader.com/demos">demos</a>
<a href="http://docs.fineuploader.com/">docs</a>
<a href="http://fineuploader.com/purchase">purchase</a>
~~~~~~~

If we want to locate all anchor links at http://fineuploader.com, the instance substring attribute selector, [first standardized in the W3C CSS 3 specification][css3-attribute-substrings], allows us to do this.

{title="selecting all anchors pointing to http://fineuploader.com", lang=javascript}
~~~~~~~
var result =
   document.querySelectorAll('A[href*="http://fineuploader.com"]');
~~~~~~~

The `result` variable above is a NodeList containing all anchor links _except_ for the third one. Why is this? Well, we are looking for an href attribute that contains the string "http://fineuploader.com". The third anchor link does _not_ contain this string. Perhaps this is not our intent, and we simply want to find all anchor links that point in some way to fineuploader.com. Simple!

{title="selecting all anchors that reference fineuploader.com", lang=javascript}
~~~~~~~
var result =
   document.querySelectorAll('A[href*="fineuploader.com"]');
~~~~~~~


#### Looking for specific words

Instead of looking for groups of characters, perhaps we need to locate a specific "word" inside of an attribute value. For example, we can write an alternate CSS class selector using this attribute word selector. Given the following HTML fragment:

{title="elements with varying classes - for an alternate CSS class selector example", lang=html}
~~~~~~~
<div class="one two three">1 2 3</div>
<div class="onetwothree">123</div>
~~~~~~~

Let's say we want to find only the element with a CSS class of "two". In addition to the [CSS class selector](#selecting-classes) I demonstrated in the last chapter, we can make use of a special attribute selector to accomplish this as well.

{title="alternate class selector", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('[class~=two]');
~~~~~~~

The `result` variable is a `NodeList` containing one entry - the first `<div>` in our sample element collection - exactly what we were looking for. But why do we need to create another class selector? We don't and the above example is not exactly practical, though it does illustrate the behavior of this selector very well. A more realistic example may be to locate a specific word in an element `title` attribute. Consider this set of elements:

{title="elements with various title attributes", lang=html}
~~~~~~~
<a href="https://github.com/rnicholus/frame-grab.js"
    title="frame-grab repo">frame-grab GitHub repo</a>

<a href="https://github.com/rnicholus/frame-grab.js/blob/master/docs/api.md"
    title="frame-grab docs">frame-grab documentation</a>

<a href="https://www.youtube.com/watch?v=hHBhP03JHIQ"
    title="frame-grab + fine-uploader">Video frame uploader</a>

<img src="https://travis-ci.org/rnicholus/frame-grab.js.svg?branch=master"
    title="frame-grab build status">

<a href="https://foo.bar/subframe-grabber"
    title="window-subframe-grabber">
    Locates all iframes inside of a given iframe</a>
~~~~~~~

Imagine the two links and one image, all obviously related to the frame-grab library, exist among a number of other unrelated links and images in a large document. But we want to find only those resources that directly relate to the frame-grab library. We can't key on "frame-grab.js" using [the substring attribute selector](#substring-attribute-selector) since not all of the elements contain an `href` or `src` attribute with "frame-grab.js". We also don't want to key on the phrase "frame-grab", as this would include the last link, which does _not_ relate to the frame-grab library.  Instead we need to select all elements with a `title` attribute that contains the specific phrase "frame-grab".

{title="locate all title attrs with 'frame-grab' word", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('[title~=frame-grab]');
~~~~~~~

Above, `result` is a `NodeList` that contains all elements in our HTML sample _except_ for the last anchor link, which is the exact result we are looking for.


#### Attribute values that start or end with...

The final set of useful advanced attribute selectors to be aware of allow you to locate elements in a document with attribute values that either start with or end with one or more specific characters. Perhaps you are wondering at this point why such a selector would be useful, pragmatically speaking. As we have done so many times before, let's start with a bit of HTML, and then discuss how these two attribute selectors may be important to us.

{title="markup for demonstrating start and end attribute value selectors", lang=html}
~~~~~~~
<img id="dancing-cat" src="/images/dancing-cat.gif">
<img id="still-cat" src="/images/still-cat.png">
<img id="dancing-zebra" src="dancing-zebra.gif">
<a href="#dancing-zebra">watch the zebra dance</a>
<a href="/logout">logout</a>
~~~~~~~

The above fragment is likely to appear inside a large document, among many other anchor links and images, but can be considered representative of a number of such elements. Say that we want to locate two things in this document:

1. All GIF images.
2. All anchors that reference elements on the current page.

The refreshing reality is that we can meet both goals in one line _without_ any third-party dependencies!

{title="locate all GIFs and references to the current page", lang=javascript}
~~~~~~~
var result = document.querySelectorAll('A[href^="#"], [src$=".gif"]');
~~~~~~~

The above selector combines a "starts with" and an "ends with" attribute value selector (respectively) using the [multiple selectors syntax](#multiple-selectors) covered in the previous chapter on finding elements. Our "starts with" selector targets any anchor element with an `href` attribute that starts with a hash mark, which would only include anchors that reference the current page. And the second selector focuses on elements with a `src` attribute value that ends with ".gif". This would include references to GIF images, assuming the image URL ends with the expected extension.


## Reading and modifying element attributes {#reading-attributes}

So now you know exactly what attributes are (and what they are _not_), and you are well-versed in selecting elements by their attribute names and values. The final aspect of attributes for me to discuss involves reading and updating an element's attributes, as well as creating new attributes. You'll discover that the appropriate approach for parsing, adding, removing, and changing attributes may depend on the type of attribute. In this final section, I will go over three distinct types of attributes: class attributes, data attributes, and all other generic native and custom attributes.


### Class attributes {#class-attributes}

Element class attributes seem to be a popular topic in "Beyond jQuery" so far. I [discussed them at length in the "Finding HTML Elements" chapter](#selecting-classes), mentioned how the `class` attribute differs from its element property name in the [attributes vs. properties section](#attributes-vs-properties) earlier in this chapter, and I even showed you how to select elements using class attributes given various types of values recently in the [finding elements with attributes section](#finding-elements-with-attributes). Well, here we are again discussing classes. But this time, I'm going to show you how to read a specific element's classes, as well as add, toggle, and remove an element's classes.


#### Reading classes

All of the situations in the reading and modifying attributes section assume you already have a handle on a specific element or elements. So, since we already have an element, perhaps we would like to know what specific CSS classes it is associated with. Or maybe we just want to find out if it is connected to one specific class. "Reading classes" will examine both of these needs.

Let's start with an actual element, for reference:

{title="reading classes reference element", lang=html}
~~~~~~~
<p class="product-name out-of-stock manual-tool">saw</p>
~~~~~~~

Imagine this element is the name of a specific tool among many other such tools in a larger document. Let's say we want to know two things about the particular tool element we've landed on:

1. Is this tool in stock?
2. Is this a manual or a power tool?

The solution provided by jQuery makes use of its `hasClass` API method:

{title="reading classes using jQuery", lang=javascript}
~~~~~~~
var inStock = !$toolEl.hasClass('out-of-stock');
var type = $toolEl.hasClass('manual-tool') ? 'manual' : 'power';
~~~~~~~

The `inStock` boolean variable will be set to a value of `false` since the element contains an "out-of-stock" class. While `type` is "manual" due to the presence of a "manual-tool" class. No surprises here.

But we don't want to use jQuery! So, how do we do all of this without it? Luckily, the modern web API offers an equally elegant solution, thanks to the [`classList` property on the `Element` interface][dom-classlist]. The WHATWG web standards organization initially drafted the specification for `classList`, and the W3C has [included it in its DOM4 document][dom4-classlist] as well.

Note that the `classList` property is a [`DomTokenList`][dom-domtokenlist]. The `DomTokenList` interface contains four notable methods, each of which I will demonstrate throughout this section. You'll see how `classList` can be used to perform all sorts of operations on an element's `class` attribute soon, but first I'm going to focus one such method - [`contains`][dom-classlist-contains].

To determine if a specific element contains a specific CSS class, the DOM API provides an intuitive property on the `classList` object: `contains`.

{title="reading classes using the DOM API - all modern browsers except for IE9", lang=javascript}
~~~~~~~
var inStock = !toolEl.classList.contains('out-of-stock');
var type = toolEl.classList.contains('manual-tool') ? 'manual' : 'power';
~~~~~~~

The above code is _identical_ to the jQuery example - just replace `hasClass` with `classList.contains` and [reap the performance benefits][jsperf-classlist-contains]!

If you need to support older browsers, you will need to resort to a regular expression in order to determine if your target element contains a certain class. Luckily, this too is fairly simple.

{title="reading classes using the DOM API - any browser", lang=javascript}
~~~~~~~
var hasClass = function(el, className) {
   return new RegExp('(^|\\s)' + className + '(\\s|$)').test(el.className);
};
var inStock = !hasClass(toolEl, 'out-of-stock');
var type = hasClass(toolEl, 'manual-tool') ? 'manual' : 'power';
~~~~~~~

Whether you are using jQuery or not, if you want to get a list of _all_ CSS classes associated with an element, you must either directly access the `class` attribute, or the `className` property on the `Element` object. In both cases, the value will be a space-delimited string of all CSS classes attached to the element.


#### Adding and removing classes

Below, we have an element, and we need to remove the "red" class, and add a "blue" one instead.

{title="HTML for add/remove classes demo", lang=html}
~~~~~~~
<p class="red">I'm red. Make me blue!</p>
~~~~~~~

We all know that the `addClass` and `removeClass` jQuery functions are used to add and remove CSS classes from elements, respectively.

{title="remove a class and add another - jQuery", lang=javascript}
~~~~~~~
$pEl.removeClass('red').addClass('blue');
~~~~~~~

The jQuery solution is pretty, and we can do it all in one line without sacrificing readability. Can we do the same thing without jQuery? Well, the web API approach is a tad more wordy, and chaining isn't built it, but it's just as easy. [The non-jQuery solution is faster as well][jsperf-classlist-addremove].

{title="remove a class and add another - web API - all modern browsers except for IE9", lang=javascript}
~~~~~~~
pEl.classList.remove('red');
pEl.classList.add('blue');
~~~~~~~

Once again, `classList` to the rescue! Perhaps you are saying to yourself, "The native solution makes me type a few more characters! This will greatly affect my productivity!" Really? If you are adding and removing classes via JavaScript so often that a few more characters will have a profoundly negative impact on agility, then perhaps it is time to instead re-evaluate your application design!

Stuck supporting IE9 and older? A solution that covers every browser under the sun is similar to the fallback for `contains` from the previous section.

{title="remove a class and add another - web API - any browser", lang=javascript}
~~~~~~~
var removeClass = function(el, className) {
   el.className =
      el.className.replace(new RegExp('(^|\\s)' + className + '(\\s|$)'), ' ');
};
removeClass(el, 'red');
pEl.className += ' blue';
~~~~~~~

That _is_ quite a bit harder than `addClass(...)` and `removeClass(...)`. Luckily, `classList` is standardized and a suitable replacement for jQuery's class manipulation going forward.


#### Toggling classes

Say we have an element, and we'd like to toggle its visibility, perhaps in response to a button click. We'll [cover events a bit later](#browser-events), so let's just focus on the logic required to toggle the visibility of this element.

{title="element that is initially hidden", lang=html}
~~~~~~~
<section class="hide">
  <h1>User Info</h1>
</section>
~~~~~~~

jQuery provides the familiar `toggleClass` method, and it is used as follows:

{title="toggle visibility of the above section - jQuery", lang=javascript}
~~~~~~~
// removes "hide" class
$sectionEl.toggleClass('hide');

// re-adds "hide" class
$sectionEl.toggleClass('hide');
~~~~~~~

This is _just as easy_ without jQuery, provided you are using a modern browser (except for IE9):

{title="toggle visibility of the above section - web API - modern browsers except IE9", lang=javascript}
~~~~~~~
// removes "hide" class
sectionEl.classList.toggle('hide');

// re-adds "hide" class
sectionEl.classList.toggle('hide');
~~~~~~~

The solution for IE9 and older is a bit hairier, but still possible. It involves checking if the class exists, and then either adding, or removing it, depending on the current state.

{title="toggle visibility of the above section - web API - any browser", lang=javascript}
~~~~~~~
var toggleClass = function(el, className) {
  var pattern = new RegExp('(^|\\s)' + className + '(\\s|$)');
  if (pattern.test(el.className)) {
    el.className = el.className.replace(pattern, ' ');
  }
  else {
    el.className += ' ' + className;
  }
};

// removes "hide" class
toggleClass(sectionEl, 'hide');

// re-adds "hide" class
toggleClass(sectionEl, 'hide');
~~~~~~~


### Data attributes {#data-attributes}

While CSS class attributes are typically used for styling elements, data attributes are, as you might expect, used to attach data to elements. Data attributes must be prefixed with "data-", and can contain stringified data associated with a particular element. Any [valid attribute value](#valid-attribute-values) is acceptable. While it is possible to construct and use non-standard element attributes, the W3C HTML5 specification [declares that custom attributes should in fact be data attributes][html5-data].

There are other ways to attach more complex data to elements. I'll cover data attributes, the HTML5 `dataset` object, the history of element data along with jQuery's role in solving this problem, and much, much more related to element data in [the next chapter](#html-data).


### Working with other standard & custom attributes

As you have already seen, [`class` attributes](#class-attributes) are special attributes that require a more specific approach to properly manipulate and read them. In fact, `class` attributes are among two types of "special" attributes, with [`data-`](#data-attributes) being the other. But what about all of the other element attributes? How can we best work with them? This section will cover reading, writing, removing, and creating both standard and custom attributes. You are probably already familiar and comfortable with jQuery's support for these tasks, but you'll see just how easy it is to work with attributes using the power of the browser.


#### Reading attributes

Let's start with a simple input element that includes both a [boolean attribute](#boolean-attributes) and a standard string value attribute.

{title="input element with boolean and non-boolean attributes", lang=html}
~~~~~~~
<input type="password" name="user-password" required>
~~~~~~~

Suppose we are given this element and we want answers to two questions:  

1. What type of `<input>` is this element?
2. Is this `<input>` a required field?

This is one area (of many) where jQuery fails miserably to ease the burden on the developer. While reading an attribute value is simple, there is no API method dedicated to detecting the presence of an attribute on a specific element. While it is still possible to do this with jQuery, the solution is not very intuitive, and will likely require those new to the library to do a bit of googling.

{title="reading an attribute value and checking if an attr exists - jQuery", lang=javascript}
~~~~~~~
// returns "password"
$inputEl.attr('type');

// returns "true"
$inputEl.is('[required]');
~~~~~~~

jQuery does not define a `hasAttr` method. Instead, you must check the element using [a CSS attribute name selector](#find-by-attribute-name). The web API _does_ provide these conveniences, and has done so since Internet Explorer 8!

{title="reading an attribute value and checking if an attr exists - web API - all moderns browsers + IE8", lang=javascript}
~~~~~~~
// returns "password"
inputEl.getAttribute('type');

// returns "true"
inputEl.hasAttribute('required');
~~~~~~~

The `getAttribute` method was first defined on the `Element` interface all the way back in 1997 [as part of the W3C DOM Level 1 Core specification][dom1core-getattribute]. And `hasAttribute` was added to the same interface 3 years later, in 2000, [in the DOM Level 2 Core spec][dom2core-hasattribute].

We can make the second half of the jQuery example a bit more intuitive, simply by breaking out of the jQuery object and operating directly on the underlying `Element`:

{title="alternate way to check if an attr exists - jQuery", lang=javascript}
~~~~~~~
// returns "true"
$inputEl[0].hasAttribute('required');
~~~~~~~

So if you're stuck with jQuery for whatever reason, consider the above example as a more straightforward way to determine if an element contains a particular attribute. As an added bonus, you'll find that bypassing jQuery as much as possible here is, as always, [much more performant][jsperf-is-vs-hasattr] than relying on the library wholesale.


#### Modifying attributes

We have a handle on a specific `<input>` element in our document, and the element looks like this:

{title="input element for modifying attributes demo", lang=html}
~~~~~~~
<input name="temp" required>
~~~~~~~

We want to modify this `HTMLInputElement` in three ways:

1. Make it an "email" input field.
2. Ensure it is _not_ required.
3. Rename it to "userEmail".   

jQuery requires we solve this problem using `attr()` to add and change attributes and `removeAttr()` to remove them.

{title="modifying attributes - jQuery", lang=javascript}
~~~~~~~
$inputEl
  .attr('type', 'email') // #1
  .removeAttr('required') // #2
  .attr('name', 'userEmail'); // #3
~~~~~~~

Without jQuery, our solution looks almost identical, and has the same wide browser support. The `Element` interface was defined to have a `setAttribute` method [since W3C's DOM Level 1 Core specification][dom1core-setattribute]. With this method, we can change and add element attribute, just like jQuery's `attr()` method. To remove attributes, we have `removeAttribute`, another method [also defined on the `Element` interface in DOM Level 1 Core][dom1core-removeattribute]. With these two methods, we can modify our input element as described above quite easily.

{title="modifying attributes - web API - any browser", lang=javascript}
~~~~~~~
inputEl.setAttribute('type', 'email'); // #1
inputEl.removeAttribute('required'); // #2
inputEl.setAttribute('name', 'userEmail'); // #3
~~~~~~~

Apart from the lack of chaining support, the native approach is just as intuitive as the route that relies on jQuery. This is one area in which web standards have _always_ been adequate, and jQuery has never provided more than a minor convenience advantage. As you have seen throughout this section, working with attributes in general is surprisingly easy without any assistance from a library.


[css2-attributes]: http://www.w3.org/TR/CSS2/selector.html#attribute-selectors

[css3-attribute-substrings]: http://www.w3.org/TR/css3-selectors/#attribute-substrings

[dom-classlist]: https://dom.spec.whatwg.org/#dom-element-classlist

[dom-classlist-contains]: https://dom.spec.whatwg.org/#dom-domtokenlist-contains

[dom-domtokenlist]: https://dom.spec.whatwg.org/#domtokenlist

[dom1core-getattribute]: http://www.w3.org/TR/REC-DOM-Level-1/level-one-core.html#ID-666EE0F9

[dom1core-removeattribute]: http://www.w3.org/TR/REC-DOM-Level-1/level-one-core.html#method-removeAttribute

[dom1core-setattribute]: http://www.w3.org/TR/REC-DOM-Level-1/level-one-core.html#method-setAttribute

[dom2core-hasattribute]: http://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-666EE0F9

[dom4-classlist]: http://www.w3.org/TR/dom/#dom-element-classlist

[element.tagname]: http://www.w3.org/TR/2000/REC-DOM-Level-2-Core-20001113/core.html#ID-104682815

[es6-reserved-words]: http://www.ecma-international.org/ecma-262/6.0/#sec-keywords

[html1-input]: http://www.w3.org/MarkUp/HTMLPlus/htmlplus_41.html

[html2]: http://www.w3.org/MarkUp/html-spec/html-spec_toc.html

[html3-input]: http://www.w3.org/MarkUp/html3/input.html

[html5-anchor]: http://www.w3.org/TR/html5/text-level-semantics.html#the-a-element

[html5-attribute-syntax]: http://www.w3.org/TR/html5/syntax.html#attributes-0

[html5-boolean-attribute]: http://www.w3.org/TR/html5/single-page.html#boolean-attributes

[html5-constraints]: http://www.w3.org/TR/html5/forms.html#constraints

[html5-data]: http://www.w3.org/TR/html5/dom.html#embedding-custom-non-visible-data-with-the-data-*-attributes

[html5-form]: http://www.w3.org/TR/html5/forms.html#the-form-element

[html5-input]: http://www.w3.org/TR/html5/forms.html#the-input-element

[html5-named-character-references]: http://www.w3.org/TR/html5/syntax.html#named-character-references

[html51-a]: http://www.w3.org/TR/html51/semantics.html#the-a-element

[html51-hidden]: http://www.w3.org/TR/html51/editing.html#the-hidden-attribute

[html-tags]: http://www.w3.org/History/19921103-hypertext/hypertext/WWW/MarkUp/Tags.html

[htmldivelement]: http://www.w3.org/TR/html5/grouping-content.html#the-div-element

[jquery-16]: http://blog.jquery.com/2011/05/03/jquery-16-released/

[jsperf-classlist-contains]: http://jsperf.com/classlist-contains-vs-hasclass

[jsperf-classlist-addremove]: http://jsperf.com/jquery-addclass-removeclass-vs-dom-classlist

[jsperf-is-vs-hasattr]: http://jsperf.com/hasattribute-vs-jquery-is

[whatwg-url]: https://url.spec.whatwg.org

[whatwg-urlutils]: https://url.spec.whatwg.org/#urlutils