Backbone.validation by Thomas Pedersen is a useful extension to Backbone which allows us to easily add 
complex validation rules to our Backbone models. You might be wondering how this extension differs from
the out-of-the box approach to writing validation rules, so let's briefly review what we learned in the
basics section.

Backbone models support a ```.validate()``` method for adding validation rules to a model, which is undefined 
unless we override it. If the model and attributes we're using pass our validation rules, ```validate()``` will 
return nothing, otherwise it can return an error of our choice. Below we can see an example of some simple
validation against a Photo model's ```src``` attribute.

```javascript
var Photo = Backbone.Model.extend({
  validate: function(attrs) {
    if (src === "") {
      return "source can't be empty!";
    }
  }
});

var one = new Photo({
  src : ""
});

one.bind("error", function(model, error) {
  alert(model.get("src") + " " + error);
});

one.set({
  caption: "this is a photo"
});
```

Todo: differences.

## Getting started

It's easy to get up and running. You only need to have Backbone (including underscore.js) in your page before including the Backbone.Validation plugin. If you are using the default implementation of the callbacks, you also need to include jQuery.

### Configure validation rules on the Model

To configure your validation rules, simply add a validation property with a property for each attribute you want to validate on your model. The validation rules can either be an object with one of the built-in validators or a combination of two or more of them, or a function where you implement your own custom validation logic. If you want to provide a custom error message when using one of the built-in validators, simply define the `msg` property with your message.

#### Example

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: {
		  required: true,
		  msg: 'Name is required'
		},
	    age: {
		  range: [1, 80]
		},
		email: {
		  pattern: 'email'
		},
		someAttribute: function(value) {
		  if(value !== 'somevalue') {
		    return 'Error';
		  }
		}
	  }
});



### Validation binding

The validation binding code is executed with a call to `Backbone.Validation.bind(view)`. The [validate](http://documentcloud.github.com/backbone/#Model-validate) method on the view's model is then overridden to perform the validation. In addition, the model is extended with an `isValid()` method.

There are several places that it can be called from, depending on your circumstances.

	// Binding when rendering
	var SomeView = Backbone.View.extend({
	  render: function(){
	    Backbone.Validation.bind(this);
	  }
	});

	// Binding when initializing
	var SomeView = Backbone.View.extend({
	  initialize: function(){
	    Backbone.Validation.bind(this);
	  }
	});

	// Binding from outside a view
	var SomeView = Backbone.View.extend({
	});
	var someView = new SomeView();
	Backbone.Validation.bind(someView);

### Specifying error messages

You can specify an error message per attribute:

	MyModel = Backbone.Model.extend({
    	validation: {
        	email: {
            	required: true,
            	pattern: "email",
            	msg: "Please enter a valid email"
            }
        }
	});

Or, you can specify an error message per validator:

	MyModel = Backbone.Model.extend({
    	validation: {
        	email: [{
            	required: true,
            	msg: "Please enter an email address"
        	},{
            	pattern: "email",
            	msg: "Please enter a valid email"
        	}]
        }
	});


## Configuration

### Callbacks

The `Backbone.Validation.callbacks` contains two methods: `valid` and `invalid`. These are called after validation of an attribute is performed.

The default implementation of `invalid` tries to look up an element within the view with an name attribute equal to the name of the attribute that is validated. If it finds one, an `invalid` class is added to the element as well as a `data-error` attribute with the error message. The `valid` method removes these if they exists.

The default implementation of these can of course be overridden:

	_.extend(Backbone.Validation.callbacks, {
      valid: function(view, attr, selector) {
	    // do something
	  },
      invalid: function(view, attr, error, selector) {
		// do something
	  }
    });

You can also override these per view when binding:

	var SomeView = Backbone.View.extend({
	  render: function(){
	    Backbone.Validation.bind(this, {
		  valid: function(view, attr) {
		    // do something
		  },
	      invalid: function(view, attr, error) {
			// do something
		  }
		});
	  }
	});

### Selector

If you need to look up elements in the view by using for instance a class name or id instead of name, there are two ways to configure this.

You can configure it globally by calling:

	Backbone.Validation.configure({
		selector: 'class'
	});

Or, you can configure it per view when binding:

	Backbone.Validation.bind(this.view, {
        selector: 'class'
    });

If you have set the global selector to class, you can of course set the selector to name or id on specific views.

### Force update

Sometimes it can be useful to update the model with invalid values. Especially when using automatic modelbinding and late validation (e.g. when submitting the form).

You can turn this on globally by calling:

	Backbone.Validation.configure({
		forceUpdate: true
	});

Or, you can turn it on per view when binding:

	Backbone.Validation.bind(this.view, {
        forceUpdate: true
    });

Note that when switching this on, the error event is no longer triggered.

## Events

After validation is performed, the model will trigger some events with the result of the validation.

### validated

The `validated` event is triggered after validation is performed, either it was successful or not. `isValid` is `true` or `false` depending on the result of the validation.

	model.bind('validated', function(isValid, model, attrs) {
		// do something
	});

### validated:valid

The `validated:valid` event is triggered after a successful validation is performed.

	model.bind('validated:valid', function(model) {
		// do something
	});

### validated:invalid

The `validated:invalid` event is triggered after an unsuccessful validation is performed.

	model.bind('validated:invalid', function(model, attrs) {
		// do something
	});

## Built-in validators

### method validator

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: function(value) {
          if(value !== 'something') {
            return 'Name is invalid';
          }
		}
	  }
	});

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: {
		  fn: function(value) {
	       	if(value !== 'something') {
	       	  return 'Name is invalid';
	      	}
	      }
		}
	  }
	});

### named method validator

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: 'validateName'
	  },
	  validateName: function(value, attr) {
		if(value !== 'something') {
          return 'Name is invalid';
        }
	  }
	});

	var SomeModel = Backbone.Model.extend({
	  validation: {
		name: {
		  fn: 'validateName'
		}
	  },
	  validateName: function(value, attr) {
		if(value !== 'something') {
          return 'Name is invalid';
        }
	  }
	});

### required

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: {
		  required: true | false
		}
	  }
	});

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    name: {
		  required: function() {
			return true | false;
		  }
		}
	  }
	});

### acceptance

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    termsOfUse: {
		  acceptance: true
		}
	  }
	});

### min

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    age: {
		  min: 1
		}
	  }
	});

### max

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    age: {
		  max: 100
		}
	  }
	});

### range

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    age: {
		  range: [1, 10]
		}
	  }
	});

### length

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    postalCode: {
		  length: 4
		}
	  }
	});

### minLength

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    password: {
		  minLength: 8
		}
	  }
	});

### maxLength

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    password: {
		  maxLength: 100
		}
	  }
	});

### rangeLength

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    password: {
		  rangeLength: [6, 100]
		}
	  }
	});

### oneOf

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    country: {
		  oneOf: ['Norway', 'Sweeden']
		}
	  }
	});

### equalTo

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    password: {
		  required: true
		},
		passwordRepeat: {
			equalTo: 'password'
		}
	  }
	});

### pattern

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    email: {
		  pattern: 'email'
		}
	  }
	});

where the built-in patterns are:

* number
* email
* url
* digits

or specify any regular expression you like:

	var SomeModel = Backbone.Model.extend({
	  validation: {
	    email: {
		  pattern: /^sample/
		}
	  }
	});

See the [wiki](https://github.com/thedersen/backbone.validation/wiki) for more details about the validators.

## Extending Backbone.Validation

### Adding custom validators

If you have custom validation logic that are used several places in your code, you can extend the validators with your own. And if you don't like the default implementation of one of the built-ins, you can override it.

	_.extend(Backbone.Validation.validators, {
      myValidator: function(value, attr, customValue, model) {
        if(value !== customValue){
          return 'error';
        }
      },
      required: function(value, attr, customValue, model) {
        if(!value){
          return 'My version of the required validator';
        }
      },
   	});

   	var Model = Backbone.Model.extend({
      validation: {
        age: {
       	  myValidator: 1 // uses your custom validator
        }
      }
   	});

The validator should return an error message when the value is invalid, and nothing (`undefined`) if the value is valid. If the validator returns `false`, this will result in that all other validators specified for the attribute is bypassed, and the attribute is considered valid.

### Adding custom patterns

If you have custom patterns that are used several places in your code, you can extend the patterns with your own. And if you don't like the default implementation of one of the built-ins, you can override it.

	_.extend(Backbone.Validation.patterns, {
	  myPattern: /my-pattern/,
	  email: /my-much-better-email-regex/
	});

	var Model = Backbone.Model.extend({
      validation: {
        name: {
          pattern: 'myPattern'
        }
      }
	});

### Overriding the default error messages

If you don't like the default error messages there are several ways of customizing them.

You can override the default ones globally:

	_.extend(Backbone.Validation.messages, {
		required: 'This field is required',
		min: '{0}' should be at least {1} characters
	});

The message can contain placeholders for arguments that will be replaced:

* `{0}` will be replaced with the name of the attribute being validated
* `{1}` will be replaced with the allowed value configured in the validation (or the first one in a range validator)
* `{2}` will be replaced with the second value in a range validator


