JSON Editor
===========

JSON Schema -> HTML Editor -> JSON

If you have structured JSON data and need a web-based editor for it, JSON Editor can help you out.

Define your data structure using JSON Schema and let JSON Editor do the rest.

Check out an example: http://rawgithub.com/jdorn/json-editor/master/example.html

Requirements
-----------------

*  A recent version of jQuery
*  Twitter Bootstrap 2.X (for editor CSS only, bootstrap javascript not needed)

### Optional Requirements

*  jQueryUI Sortable to enable drag and drop re-ordering of array elements
*  a javascript templating engine to enable template macros (see below)

Usage
--------------

```javascript
// Initialize the editor with a JSON schema
$("#editor_holder").jsoneditor({
  schema: {
    type: "object",
    properties: {
      //...
    }
  }
});

// Set the editor's value with a JSON object
$("#editor_holder").jsoneditor('value',some_json_object);

// Get the editor's current value as a JSON object
console.log($("#editor_holder").jsoneditor('value'));

// Listen to changes on the editor
$("#editor_holder").on('change',function() {});

// Destroy the editor completely
$("#editor_holder").jsoneditor('destroy');
```

JSON Schema Support
-----------------
JSON Editor currently only supports a small subset of JSON Schema.  
All of the primitive types except `null` are supported (object, array, string, number, integer, boolean).

Only the following JSON schema keywords are supported.  All other keywords will be ignored.

*  id
*  title
*  description
*  default
*  enum (for type `string` only)
*  properties
*  items
*  $ref
*  definitions
*  format
*  minimum
*  maximum
*  minItems
*  maxItems

JSON Editor only supports arrays with a single `items` schema.

Currently, JSON Editor only supports schema references to the root node in the format `#/definitions/DEFINITION_NAME`.

JSON Editor supports all of the HTML5 input types for the `format` parameter.

The `minimum` and `maximum` schema keywords are only used when the format is set to `range`.

Template Macros
------------------
A unique feature of JSON Editor is the support for template macros.  This lets you specify a field's value in terms of other fields.  Templates only work for fields of type `string`.

```json
{
  "type": "object",
  "id": "person",
  "properties": {
    "fname": {
      "title: "First Name",
      "type": "string"
    },
    "lname": {
      "title": "Last Name",
      "type": "string"
    },
    "generated_email": {
      "title": "Generated Email",
      "type": "string",
      "template": "{{ fname }}.{{ lname }}@domain.com",
      "vars": {
        "fname": "person.fname",
        "lname": "person.lname"
      }
    }
  }
}
```

Any time the `fname` or `lname` field is changed, the `generated_email` field will re-calculate its value.

Any variables you want to use in the template must be declared in the `vars` object.  The key is the variable name and the value is the dot separated path to the field, starting at an ancestor node that has an `id` specified.  If there are no ancestor node with an `id` specified, the special keyword `root` can be used to refer to the outermost object.

By default, templates are configured to use the powerful swig templating engine (https://github.com/paularmstrong/swig).

You can use a different templating engine by overwriting the `$.jsoneditor.template` variable.  Here are examples for Handlebars and Mustache:

```js
// Handlebars uses the same API as swig, so it's an easy replacement
$.jsoneditor.template = Handlebars;

// Mustache needs a small wrapper object to work
$.jsoneditor.template = {
  compile: function(template) {
    return function(view) {
      return Mustache.render(template, view);
    }
  }
};
```


Editors
-----------------
Each primitive type has its own editor.  This can be overridden with the `editor` property.  The only custom editor included is the `table` editor, which is a more compact version of the `array` editor.

It is easy to add your own custom editors as well.

### Creating a Custom Editor

All editors must extend the `$.jsoneditor.AbstractEditor` class and must implement the `setValue` and `getValue` methods.  Here's a really simple example `textarea` editor:

```js
$.jsoneditor.editors.textarea = $.jsoneditor.AbstractEditor.extend({
  initialize: function() {
    this.input = $("<textarea>").appendTo(this.div);
  },
  getValue: function() {
    return this.input.val();
  },
  setValue: function(value) {
    this.input.val(value);
  }
});
```

In your JSON schema, you would specify the editor like this:

```json
{
  "type": "string",
  "editor": "textarea"
}
```