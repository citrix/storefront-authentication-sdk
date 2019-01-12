# JavaScript API

The StoreFront Authentication SDK allows clients and servers to negotiate capabilities, such as: credential types and label types. In this context, a label is information displayed to the users, whereas the credential is the data provided by users in order to authenticate. Both the credentials and labels are advertised to the StoreFront server by Receiver for Web via the X-Citrix-AM- CredentialTypes & X-Citrix-AM-LabelTypes headers provided in the request to the start conversation URL.

In certain use cases, a new credential or label type needs to be defined, linked to custom behavior on both the client and the server.

The StoreFront Authentication SDK already has the server-side APIs for adding new credential and label types. The API described here is to provide support for new credential & label types in Receiver for Web, in particular to allow the following customizations:

* Control the credential and label types advertised by the client
* When a new credential or label type is detected in the form presented by the server, the client-side customization controls what is rendered 
* Allow for data to be sent to the server without intervention from the user.

**Note**: This API is an advanced feature and hence is only available if the Receiver for Web “classic” experience is disabled.

#Installation

The plugins developed using this API are expected to be packaged and deployed by creating a StoreFront Feature Package, with the WebReceiver feature as its parent. This package can then be deployed with the pre-existing Add-DSWebFeature PowerShell command that will deploy the relevant files. This command also modifies the web.config file to add the plugin definitions.

##Adding plugins to Web.Config

The configuration of Receiver for Web is controlled by the web.config file, as such, when installing a plugin an element needs to be added to it for the plug into be registered with Receiver for Web. This is done via the use of merge files, and removed using a corresponding unmerge file.

Below is an example of one such merge file:

```
<?xml version="1.0" encoding="utf-8"?>
<merge>
  <!-- Add the plugin definition -->
<addElement xpath="/configuration/citrix.deliveryservices/webReceiver/clientSettings/plugins"
           ensurexpath="true" key="name">
    <plugin name="test plugin" src="plugins/YourCompany/sample.js"/>
  </addElement>
</merge>
``` 

The required content for the plugin element is:

* The name that has to match the name defined within the plugin file itself in order for Receiver for Web to load it correctly
* The location of the file relative to the top level directory for the Receiver for Web site.

Additional child elements can be added to the plugin element which can be found below:

###Script

####Parameters

src: the source location of the file, which unless absolutely defined, is relative to the top level directory for the Receiver for Web site.

####Purpose

To provide additional scripts in addition to the primary file. Each script element should have a single file and must be contained within a wrapping scripts element.

####Example

```
<plugin name="Example Dependency" src="plugins/YourCompany/Script.js">
	<scripts>
		<script src="chrome-extension://example-extension/example.js"/>
	</scripts>
</plugin>
```

###Style

####Parameters

src: the source location of the file, which unless absolutely defined, is relative to the top level directory for the Receiver for Web site.

####Purpose

To provide style sheets alongside the primary JavaScript file. Each style element should have a single file and must be contained within a wrapping styles element.

####Example

```
<plugin name="Style Example" src="plugins/YourCompany/Script.js">
	<styles>
		<style src="plugins/YourCompany/Styles.css" />
		<style src="plugins/YourCompany/508-Styles.css" />
	</styles>
</plugin>
```

###Param

####Parameters

name: the name of the parameter, used as the key in the data structure provided. 

value: the value of the parameter.

####Purpose

To provide data to the plugins, passed in as a parameter in the plugins ‘initialize’ function. The data is provided as a JavaScript object with each parameter stored as a key/value pair. The parameters are intended to be for passing in data which are infrequently changed or may need to vary between installations. Passing them as parameters allows for them to be updated/configured without having to produce a new plugin and re-installing every time a value needs to change. As such it’s recommended that any values that would usually be hard coded within the plugin are passed in as parameters.

####Example

```
<plugin name="Parameter Example" src="plugins/YourCompany/Script.js">
	<params>
		<param name="data-gatherer" value="plugins/YourCompany/example.ocx" />
		<param name="timeout" value="500" />
	</params>
</plugin>
```

###Combinations

A single plugin can have any combination of these child elements, as in the below example:

```
<plugin name="Combined Example" src="plugins/YourCompany/Script.js">
	<scripts>
		<script src="chrome-extension://example-extension/example.js"/>
	</scripts>
	<params>
		<param name="timeout" value="500" />
	</params>
	<styles>
		<style src="plugins/YourCompany/Styles.css" />
	</styles>
</plugin>
```
 
#Defining Custom Credentials and Labels

##Introduction to credential and label handlers

Credential Handlers are objects specifically designed to provide additional behavior to Receiver for Web. Each custom credential or label required must provide the necessary code within a credential or label handler.

##Plugins

Receiver for Web has a plugin loading mechanism which is used by the Authentication SDK JavaScript API to load the credential and label handlers into Receiver for Web.

Each plugin requires a name attribute that must match the name provided in the corresponding entry in the web.config file. Also required by the JavaScript API, is that the plugin has an initialize function that has a single parameter for the object, containing the data provided as parameters in the web.config file. Credential and label handlers are added into Receiver for Web by calling the functions defined below within the initialize function of a plugin. Multiple handlers can be added in a single plugin.

##Functions

###CTXS.ExtensionAPI.addCustomCredentialHandler

This function adds the custom credential handler provided into Receiver for Web, with any missing functions providing default behavior.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|credentialHandler|Object|Credential handler to be added.|

####Example

```javascript
// Add a sample Auto post credential to Receiver for Web CTXS.ExtensionAPI.addCustomCredentialHandler({
	// The name of the credential, must match the type returned by the server
	getCredentialTypeName: function () { return "sample-auto-post"; },
	// Function where information to be auto-posted is generated
	getDataToAutoPost: function ($form, callback) { callback(navigator.userAgent); }
});
```
###CTXS.ExtensionAPI.addCustomAuthLabelHandler

This function provides the corresponding behavior to addCustomCredentialHandler for custom label handlers.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|labelHandler|Object|Label handler to be added.|

####Example
 
```javascript
// Sample Label handler
CTXS.ExtensionAPI.addCustomAuthLabelHandler({
	// The name of the label, must match the type returned by the server
	getLabelTypeName: function () { return "test-label"; },
	// Rendering instructions for the label
 	getLabelTypeMarkup: function (requirements){
 	return $("<object/>")	
 		.attr("src", requirements.label.binary)
 		.addClass("flash-object");
 },
	// Instruction to parse the label as if it was a standard type
	parseAsType: function () {
	return "image";
}
});
```

#Label Types

Custom labels are intended for displaying information in new ways, and are not expected to provide additional inputs.

##Functions

###getLabelTypeName

The purpose of this function is to provide the label type value that the handler provides functionality for, and the response should match the type value specified by the server.

####Parameters

None.

####Returns

Expected return value is a string containing the label type.

####Example

```
 // The name of the label, must match the type returned by the server
 getLabelTypeName: function () { return "test-label"; }
```
 
###getLabelTypeMarkup

The purpose of this function is to provide the markup for rendering the custom label.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|requirements|object|Parsed contents of the requirements node provided by the server. See Appendix I for details.|

####Returns
Expected return value is a jQuery object with all necessary markup to render the label.

####Example

```javascript
// Rendering instructions for a flash object label
getLabelTypeMarkup: function (requirements) {
	return $("<object/>")
 	.attr("src", requirements.label.binary)
 	.addClass("flash-object");
}
```

###parseAsType

This function allows a custom label type to be parsed from XML into JavaScript object form as if it were a standard, existing label type. This is intended to increase the simplicity of parsing where non-standard tags are not present. Returning a non-standard type will result in a parse error.

####Parameters

None

###Returns

String – name of existing label type to parse as, one of: none, plain, heading, information, warning, error, confirmation, & image.

####Example

```javascript
// Instruction to parse the label as if it was a standard type
parseAsType: function () { return "plain"; }
```
 
###parseLabelNode

The function allows custom labels providing custom XML tags to be parsed into the label object for use when generating markup. The intention is that this is only used where parseAsType is not appropriate, as this function is only called if parseAsType does not return a valid string.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|labelNode|jQuery Object|XML structure for the label. Contains the label node of the requirement provided by the server, to be parsed into a jQuery object used by rest of form generation structure.|

####Returns

A JavaScript object with all of the information parsed from the requirements node. The type property is set by Receiver for Web afterwards, so would override any value applied within this function. There is no mandatory data to provide in the response. 

####Example

```javascript
// Provide custom parsing for a custom progress tracking label.
// Label consists of a series of discrete images, displayed according to progress level.
parseLabelNode: function(labelNode) {
	var textNode = $("Text", labelNode);
	var progressArray = $("progressArray", labelNode).children();
	return {
		text: textNode.text(),
		progressStates: progressArray
	};
}
```

#Credential Types

Custom credentials are intended to provide custom input structures for a field, allowing non-standard fields to be used.

##Custom Credential Data

Default credentials only provide an ID and the credential type as fields within the credential node of a given requirement, custom credentials can also provide additional data in the form of key/value pairs within the credential node. This data is added as part of the http://citrix.com/authentication/response/1/extensions namespace, and can include request data for an embedded flash/activeX object, or image data to display. All data values must be passed in as a string and adhere to a structure of Key and Value tags wrapped inside Item tags, as shown within the below example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AuthenticateResponse xmlns="http://citrix.com/authentication/response/1"
xmlns:ext="http://citrix.com/authentication/response/extensions/1">
	<AuthenticationRequirements>
		<Requirements>
			<Requirement>
				<Credential>
					<ID>exampleID</ID>
					<Type>example-credential</Type>
					<ext:Data>
						<ext:Item>
							<ext:Key>value1</ext:Key>
							<ext:Value>value</ext:Value>
						</ext:Item>
						<ext:Item>
							<ext:Key>value2</ext:Key>
							<ext:Value>true</ext:Value>
						</ext:Item>
						<ext:Item>
							<ext:Key>value3</ext:Key>
							<ext:Value>data:image/png;base64,...</ext:Value>
						</ext:Item>
					</ext:Data>
				</Credential>
				<Label/>
				<Input />
			</Requirement>
		</Requirements>
	</AuthenticationRequirements>
</AuthenticateResponse>
```

##Functions

###getCredentialTypeName

The purpose of this function is to provide the credential type value that the handler provides functionality for, and the response should match the type value specified by the server.

####Parameters

None.

####Returns

Expected return value is a string containing the credential type.

####Example

```javascript
// The name of the credential, must match the type returned by the server
getCredentialTypeName: function () { return "test-credential"; }
```
###getCredentialTypeMarkup

The purpose of this function is to provide the markup for rendering the custom credential.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|requirements|object|Parsed contents of the requirements node provided by the server. See Appendix I for details.|

####Returns

Expected return value is a jQuery object with all necessary markup to render the credential, including the input element that contains the information sent back to the server upon submission.

####Default Behaviour

If this function is omitted from the credential handler, the handler will return an empty div with the credential type as a class.

####Example

```javascript
// Rendering instructions for an image selector credential
// Images are provided in the ‘display’ field of the displayValues in the requirement as data strings.
getCredentialTypeMarkup: function (requirements) {
	var $row = $("<tr/>"),
		name = requirements.credential.id,
		displayValues = requirements.input.radioButton.displayValues,
		initialSelection = requirements.input.radioButton.initialSelection;
	displayValues.forEach(function (data) {
		var value = data.value,
			btnId = name + '-' + value;
		var $cell = $("<td/>");
		var $image = $("<img/>").attr("src", data.display);
		var $radioButton = $("<input type='radio'>")
			.attr("id", btnId)
			.attr("name", name)
			.attr("value", value);
		if (value === initialSelection) {
			$radioButton.prop("checked", true);
	}
		$cell.append($image).append($radioButton);
		$row.append($cell);
	});
	var $body = $("<tbody/>").append($row);
	return $("<table/>").append($body);
}
```

#Auto Post Credentials

The Authentication SDK’s JavaScript API provides support for credentials that gather, collate and return data to the server without user intervention. Receiver for Web handles populating the credential field on behalf of auto post credentials.

##Functions
Auto post credentials support all of the functions other custom credentials use, along with one additional function.

###getDataToAutoPost
This function is key to all auto post credentials, as its presence distinguishes an auto post credential from all other custom credentials. This function is called by Receiver for Web upon displaying the form, and is for collecting the information required to authenticate, and returning it via the provided callback to be submitted. The requirements node for the credential is also passed to this function to allow for server provided initial information to be accessed as part of the data collection.

####Parameters

|Parameter|Type|Description|
|---|---|---|
|`$form`|jQuery object|JQuery selector containing the form the credential is stored in. Provides access to the rest of the form to allow for updating of information such as progress bars and setup instructions (e.g. allowing an ActiveX control to run).|
|requirements|object|Parsed contents of the requirements node provided by the server. See Appendix I for details.|
|callback|function|Callback function to be called to complete the auto post segment, the function has a single parameter and expects the data gathered to be provided as a string.|

####Example

```javascript
// Function where information to be auto-posted is generated, and posted via the callback provided as a parameter
getDataToAutoPost: function ($form, requirements, callback) { callback( navigator.userAgent ); }
```

###getCredentialTypeMarkup
As Receiver for Web handles field population for auto post credentials, getCredentialTypeMarkup can be considered an optional function for auto post credentials. Instead of providing the HTML defining an input, as expected for a user-submitted custom credential, it’s expected that this function will define a div containing additional elements for use by the credential. Additional elements could include script tags containing an ActiveX control or a flash object that contains the data gathering logic.

####Example

```javascript
// Additional custom markup for a credential
getCredentialTypeMarkup: function (requirements) {
	// URL of browser plugin that does gathering passed in as input’s initial value
	var gathererSource = requirements.input.text.initialValue;
	var $div = $("<div/>").addClass(this.getCredentialTypeName()).attr('id', "auto-post-test");
	var $gatherer = $("<object/>").attr("data", gathererSource);
	$div.append($gatherer);
	return $div;
}
```

##Limitations

The Authentication SDK’s JavaScript API provides support for credentials that gather, collate and return data to the server without user intervention. In order to do this, there are some additional restrictions applied to such ‘auto post’ fields.

###Form Structure

For an auto post field to behave as expected, it must be the only requirement present on the form requiring an input, standalone labels are permitted. This is because an auto post field may submit the form before user-driven credentials or other auto-post fields can be populated. To that end, Receiver for Web will take whichever credential appears first, be it an auto-post or user- driven credential, as the intended use for the form. Once a user-driven credential has been added to the form, all auto-post credentials will be blocked and reported as parse errors. Likewise, when an auto-post credential has been added, all other credentials, including other auto post credentials and standalone labels, will be blocked and reported as parse errors. Standalone labels are allowed before the auto-post credential, so it’s recommended that the auto-post credential is the last requirement in the form if you wish to display any labels.

For authentication conversations where multiple auto post credentials are required, or where an auto post credential can have a branching logical flow (e.g. check whether a required browser extension is installed, then asking the user to install if not) it is recommended that each of these steps are handled as separate forms in the conversation. This is because within authentication conversations, knowledge of branching and context should be the responsibility of the server.

###Returned values

Any data gathered by the credential must be encoded into a string in order to be returned to the server.


#Creating a plugin and examples

When creating a plugin to provide credential and label handlers to Receiver for Web, it is recommended that the following basic template is used:

```javascript
(function ($) {
	CTXS.ExtensionAPI.addPlugin({
		// This is required and must match the plugin name defined in the web.config.
		// If it doesn't match, the plugin will not load correctly
		name: "Example",
		initialize: function (args) {
			// Add credential/label handlers here
		}
	});
	// Additional functions used within the handlers can be added here
})(jQuery);
```

All that is needed to build on this template is to ensure that the plugin name matches that of the entry in the web.config file, and to add the relevant credential or label handlers.

##Example Credential Handler

Below is a template for a minimal credential hander:

```javascript
// Sample Credential handler
CTXS.ExtensionAPI.addCustomCredentialHandler({
// The name of the credential, must match the type returned by the server
getCredentialTypeName: function () { return "example-credential"; },
// Rendering instructions for the credential
getCredentialTypeMarkup: function (requirements) {
}
// Rendering instructions go here
});
```

##Example Auto Post Credential Handler

Below is an example minimally defined auto post credential hander designed to gather the user agent string for demonstration purposes:

```javascript
// Auto-post credential
CTXS.ExtensionAPI.addCustomCredentialHandler({
// The name of the credential, must match the type returned by the server
getCredentialTypeName: function () { return "example-auto-post"; },
/* Function where information to be auto-posted is generated,
}
and posted via the callback provided as a parameter */
getDataToAutoPost: function ($form, callback) {
callback(navigator.userAgent);
});
```

##Example Label Handler

Below is an example of a minimally defined custom label. This example will add a provided flash object to the form. As the flash object is provided as a binary string by the server, the label requests to be parsed as an image, which already leverages a ‘binary’ node.

```javascript
// Sample Label handler
CTXS.ExtensionAPI.addCustomAuthLabelHandler({
// The name of the label, must match the type returned by the server
getLabelTypeName: function () { return "test-label"; },
// Rendering instructions for the label
getLabelTypeMarkup: function (requirements) {
return $("<object/>")
.attr("src", requirements.label.binary)
.addClass("flash-object");
},
// Instruction to parse the label as if it was a standard type
parseAsType: function () {
return "image";
}
});
```

#Appendix I – Requirements Object

Having received the authentication requirements from the server as an XML document, Receiver for Web parses this into a JavaScript object. The contents of individual requirements within this parsed object form the Requirements object for a given label or credential, which is passed to custom credentials and labels as a parameter for getCredentialTypeMarkup, getLabelTypeMarkup and getDataToAutoPost.

##Expected Structure
For credential handler functions, the entire requirement will be passed, meaning that the object will adhere to the basic pattern outlined below, for label handlers, only the label component of the requirement will be passed.

```javascript
{
	"credential": {...},
	"label": {..},
 	"input": {...}
}
```

###Credential Element

The credential element contains, by default:

|Property name|Value type|Description|
|---|---|---|
|Id|String|ID of the requirement|
|Type|String|The credential type|

###Label

Labels provide different parsing based on what type they have. These expectations are also applicable to custom labels with the ‘parseAsType’ function defined.
  
|Property Name|Expected for|Mapping from XML|Value type|Description|
|---|---|---|---|---|
|type|All|Type → type|String|The label type|
|src|Image|Binary → src|String|Source for the image to display|
|text|plain <br> heading<br> information <br> warning<br>  error<br>  confirmation <br> inprogress|Text → text|String|The text to display.
|highlightFields|plain <br> heading <br> information <br> warning <br> error<br>  confirmation <br> inprogress<br>|HighlightFields → highlightFields <br> <br> each entry maps<br> <br> HighlightFields[CredentialID] → highlightFields| Array of Strings|An optional list of other credential IDs to be highlighted.|

###Input

Every input node has a field for each possible input, however it is expected that at most only one is populated at any given time. They are:

* assistiveText
* text
* radioButton
* comboBox
* multiComboBox
* button
* checkBox

##Example – Test Forms

The test forms StoreFront sample now includes additional forms including custom credentials and labels to test. Below is the XML representation for one of these forms as provided by the server:

```xml
<AuthenticateResponse xmlns="http://citrix.com/authentication/response/1">
<Status>success</Status>
<Result>more-info</Result>
<StateContext/>
<AuthenticationRequirements>
  <PostBack>/Citrix/StoreAuth/TestForms</PostBack>
  <CancelPostBack>/Citrix/StoreAuth/TestForms/Cancel</CancelPostBack>
  <CancelButtonText>Cancel</CancelButtonText>
  <Requirements>
    <Requirement>
      <Credential>
        <Type>none</Type>
      </Credential>
      <Label>
        <Text>Example Custom Types</Text>
        <Type>heading</Type>
      </Label>
      <Input/>
    </Requirement>
    <Requirement>
      <Credential>
        <ID>customCredentialId</ID>
        <Type>test-credential</Type>
      </Credential>
      <Label>
        <Text>Select the middle image</Text>
        <Type>plain</Type>
      </Label>
      <Input>
        <RadioButton>
          <InitialSelection>0</InitialSelection>
          <DisplayValues>
            <DisplayValue>
              <Display>data:image/png;base64,...</Display>
              <Value>0</Value>
            </DisplayValue>
            <DisplayValue>
              <Display>data:image/png;base64,...</Display>
              <Value>1</Value>
            </DisplayValue>
            <DisplayValue>
              <Display>data:image/png;base64,...</Display>
              <Value>2</Value>
            </DisplayValue>
          </DisplayValues>
        </RadioButton>
      </Input>
    </Requirement>
    <Requirement>
      <Credential>
        <ID>customLabelId</ID>
        <Type>textcredential</Type>
      </Credential>
      <Label>
        <Text>surprise!</Text>
        <Type>test-label</Type>
      </Label>
      <Input>
        <AssistiveText>Hover over the label to see the text to type in</AssistiveText>
        <Text>
          <InitialValue/>
          <Constraint>.+</Constraint>
        </Text>
      </Input>
    </Requirement>
    <Requirement>
      <Credential>
        <ID>nextButtonId</ID>
        <Type>none</Type>
      </Credential>
      <Label>
        <Type>none</Type>
      </Label>
      <Input>
        <Button>Next</Button>
      </Input>
    </Requirement>
  </Requirements>
</AuthenticationRequirements>
</AuthenticateResponse>
```
Following parsing, the relevant requirement object is passed to the credential for markup generation, as part of this the label specific nodes are provided to the label for markup generation. In the example above, the following would be passed to the credential handler for test-credential:

```json
{
  "credential": {
    "id": "customCredentialId",
    "type": "test-credential"
  },
  "label": {
    "type": "plain",
    "text": "Select the middle image",
    "highlightFields": [ ]
  },
  "input": {
    "assistiveText": null,
    "text": null,
    "radioButton": {
      "initialSelection": "0",
      "displayValues": [
        {
          "display": "data:image/png;base64,...",
          "value": "0"
        },
        {
          "display": "data:image/png;base64,...",
          "value": "1"
        },
        {
          "display": "data:image/png;base64,...",
          "value": "2"
        }
      ]
    },
    "comboBox": null,
    "multiComboBox": null,
    "button": null,
    "checkBox": null
  }
}
```
 
Whereas the label handler for test-label would receive the following:

```javascript
{
 "type": "test-label",
 "text": "surprise!",
 "highlightFields": [ ]
 }
```
 
#Appendix II – Parse errors

##How to access parse errors

Parsing errors are outputted to the browser console, prefixed with “ctxsFormsAuthentication: ”.

##Error Messages Associated with Custom Credentials & Labels

**“More than one field in a form containing an auto-post credential is not allowed.”**

######Likely Cause

Due to limitations surrounding submitting auto post credentials, auto post credentials must be the only submittable field in the form.

This error will occur if any field is present alongside an auto post field in a form, the only exception being a standalone label, which must be placed before the auto post field.

######Possible next steps

Check if the desired auto post field is the only field in the form. If you have more than one auto post field, spread them across multiple forms.

**“Unknown label type \<TYPE>” or “Unrecognised label type : \<TYPE>”**

######Likely Cause

The label is not recognised. If it’s a custom label, it may not be registering the label handler correctly.

######Possible next steps
Check that the string provided by getLabelTypeName matches the type provided by the server for that form.