---
layout: post
title:  "JIRA gadgets: Callback field gotchas"
date:   2016-12-17 15:02:43 +0100
comments: true
tags: jira gadget
---
If developing JIRA gadgets, you might come to the point that default gadget fields won't suffice
your requirements. In that case you'll quickly realise that you need to create your own callback field.

In general, those [callback fields][field reference] are fairly okay documented, but they lack of real world examples and have
some undocumented behaviour.

### Attach input field to root element

As documented, the field definition object needs to provide a `callback` function which will get the `parentDiv` passed.
JIRA will expect the parentDiv to have an HTML-input element directly attached, which will provide the field value once the
input data gets saved. Otherwise, the value of callback field cannot be determined and will never provide a value.

You need to develop a solution to persist and load the data from and to that input element on load and save, that fits your needs.

### jQuery AJAX not supporting promises

While developing JIRA plugins, we get used to jQuery related limitaitons. The fact that JIRA uses an ancient jQuery version
causes a lot of inconvenience, especially with 3rd party jQuery extensions.
Sadly, gadget development won't be an exception.

Full of self confidence, the documentation states the following: 

> The framework wraps the OpenSocial gadgets.io.makeRequest method with the standard jQuery interface. 
> You should write code as if you were just using the jQuery interface. - [Gadget Developers' JavaScript Cookbook][ajax reference]

Sadly, this is not entirely true. Due to the change of the AJAX implementation to gadgets.io.makeRequest, jQuerys AJAX
function won't behave as exepcted. We are used to call jQuery AJAX request through promise chaining, which won't work
with JIRA gadgets. It will result in the following error.

```javascript
'AJS.$.ajax(...).done is not a function'
```

Instead we need to add our callback function old fashioned to the AJAX options object. This is a little inconvenient, but
won't have a bigger impact within our gadget. If 3rd party jQuery libraries are in use, which rely on promise style
AJAX calls, this will lead to further workarounds.

```javascript
//Won't work:
AJS.$.ajax('/myResource').done(function (result) { console.log (result)});

//Needs to be refactored to this:
AJS.$.ajax({
    url: '/myResource',
    success: function(result) {
        console.log(result);
    }
});
```

[field reference]: https://developer.atlassian.com/display/GADGETS/Field+Definitions#FieldDefinitions-CallbackBuilder
[ajax reference]: https://developer.atlassian.com/display/GADGETS/Making+Ajax+Calls