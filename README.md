# Postcode lookup with no UI.

## The problem.

Addresses are generally long and hard to type. Typing your address out longhand can take a long time, and is very error prone.

Addresses can be accurately input from just the postcode and house number/name. All other parts of the address can be looked up from a database.

## Existing solutions.

1. User types postcode.
2. User clicks "lookup address".
3. Loading...
4. UI updated to present a list of every house on user's street.
5. User picks their house from the list.
6. UI updated again to remove list of houses.

Most existing solutions have clunky UI that jolts the user around the page, then it presents the user with every house on their street. Sometimes, if the developer of the website is feeling extra-lazy, the list of houses is not even in any kind of order. [Insert statistic about lost sales here.]

## The noUI way.

1. User types postcode.
2. Form fields updated with full address, minus the house number.
3. User types house number.

Seeing as we're only concerned with the postcode and house number as means for identification of address, that is all we need to make the user type. The user interface remains consistent, allowing the user to enter the information efficiently. A loading indicator is added to remaining address fields while the lookup occurs, and the form silently fills itself in.

# Usage.

```javascript
NOUI.Postcode.init(); // uses default configuration (see below).
```

Full HTML page example:

```html
<!doctype html>
<form>
	<fieldset>
		<legend>Your name.</legend>
		
		<label>
			<span>Title</span>
			<select name="title">
				<option>Mr</option>
				<option>Mrs</option>
				<option>Miss</option>
				<option>Ms</option>
				<option>Dr</option>
				<option>Rev</option>
			</select>
		</label>
		
		<label>
			<span>Forename</span>
			<input name="forename" placeholder="e.g. Alan" />
		</label>
		
		<label>
			<span>Surname</span>
			<input name="surname" placeholder="e.g. Turing" />
		</label>
	</fieldset>
	
	<fieldset>
		<legend>Your address.</legend>
		
		<label>
			<span>Postcode</span>
			<input name="address_postcode" placeholder="e.g. W9 1ER" />
		</label>
		<label>
			<span>House name/number</span>
			<input name="address_house" placeholder="e.g. 2" />
		</label>
		<label>
			<span>Street name</span>
			<input name="address_street" placeholder="e.g. Warrington Crescent" />
		</label>
		<label>
			<span>District</span>
			<input name="address_district" placeholder="e.g. Little Venice" />
		</label>
		<label>
			<span>Town / City</span>
			<input name="address_town" placeholder="e.g. London" />
		</label>
	</fieldset>
</form>
```

# Form configuration.

By default, the field names this library looks for all start with `address_`. This can easily be selected using CSS with the `input[name^="address_"]` selector.

Calling `NOUI.Postcode.init()` with no arguments assumes that the postcode field is named `address_postcode`, and the other fields in the form match the above example.

To provide a different configuration, pass in two arguments to the `init` function:

1. The CSS selector matching the postcode field. (Default: `[name="address_postcode"]`).
2. The CSS selector matching all address fields. (Default: `[name^="address_"]`).

# Using a different postcode search provider.

This library is compatible with any postcode lookup service by assigning a promise-based function wrapper to `NOUI.Postcode.lookup`.

The lookup function muse return a `Promise`, defining the resolution/rejection process. The purpose of this wrapper is to translate one API's output to a standardised footprint as consumed by this library.

An example of using a custom third-party postcode lookup library:

```javascript
NOUI.Postcode.lookup = function(postcode) {
	return new Promise(function(resolve, reject) {
		// use standard HTTP call behind the scenes.
		fetch("http://example.postcode.lookup?postcode=" + postcode)
		.then(function(response) {
			// the third-party API's response needs translating into something usable.
			response.json().then(function(jsonObject) {
				resolve({
					"address_street": jsonObject.results[0].streetName.content,
					"address_district": jsonObject.results[0].postalArea.content,
					"address_town": jsonObject.results[0].postalTown.content,
					// hard-code this, because our example API doesn't supply the country.
					"address_country": "United Kingdom",
				});
			});
		})
		.catch(function(error) {
			// hand the error to this library's handler.
			reject(error);
		});
	});
};
```

