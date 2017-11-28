UIComponent
-------------
The new framework model `UIComponent` allows 
1. Metadata enablement of handwritten form
2. Auto generation of Model Form

It has following properties:

Property          | Required | Default |         Description
------------------|----------|---------|----------------------------
`name`            | yes      |    -    | name of UI component, matches Polymer proptotype.is property
`templateName`    | no       |    -    | Based on templateName component is generatred
`filePath`        | no       |    -    | If component is file based, file location
`content`         | no       |    -    | html content of the component, if not stored it is generated from model metadata
`modelName`       | no       |    -    | If this component renders default UI for a model then model name
`fields`          | no       |    -    | fields of default model to be rendered
`gridConfig`      | no       |    -    | fields to display in grids
`autoInjectFields`| no       |    -    | Whether remaining model fields should be auto injected or not
`excludeFields`   | no       |    -    | Fields which are to be excluded from injection


### Enable metadata enrichment of handwritten Polymer element

1. Post an entry in `UIComponent` as below:

```json
{
    "name":"my-owesome-element",
    "filePath":"/path/to/my-owesome-element.html"
}
```

2. Replace `Polymer` call in your element with `MetaPolymer`

3. Replace your client side import statement

```html
<link rel="import" href="/path/to/my-owesome-element.html">

with

<link rel="import" href="/components/my-owesome-element.html">
```

This will enable `my-owesome-element` to be dynamically enriched at runtime using `UIElement` defined for 
this component.

### Auto generate model form

Lets take the second use case to auto-generate data entry form for a particular model, say `Apprentice`
defined as below:

```json
{
	"name": "Apprentice",
	"idInjection": true,
	"properties": {
		"name": {
			"type": "string",
			"required": true
		},
		"email": {
			"type": "email",
			"required": false
		},
		"dateOfBirth": {
			"type": "date",
			"required": true
		},
		"stipend": {
			"type": "number",
			"required": false
		},
		"primarySkill": {
			"type": "string",
            "refcodetype":"SkillCode",
			"required": true
		},
		"language": {
			"type": "string",
			"required": true,
            "in":["Chinese","English","French","Hindi","Hebrew"]
		},
		"nationality": {
			"type": "string",
			"enumtype": "NationalityEnum",
			"required": false
		},
		"elligible": {
			"type": "boolean",
			"required": false,
			"default": false
		},
		"interests": {
			"type": [
				"string"
			]
		},
		"fromTime": {
			"type": "timestamp"
		},
		"toTime": {
			"type": "timestamp"
		}
	},
	"validations": [],
	"relations": {
		"experienceDetails": {
			"type": "hasMany",
			"model": "ExperienceData"
		},
		"department": {
			"type": "belongsTo",
			"model": "Department"
		}
	},
	"acls": [],
	"methods": {}
}

```

1. Define UI Routes
We create two `UIRoute` entries. First one displays list of `Apprentice` and the second one shows a form for 
editing a particular `Apprentice` record.

(Note that below route definition are generic) 

```json
[
  {
    "type": "elem",
    "name": "elem",
    "path": "/ui/:modelName",
    "import": "elements/model-ui-generator.html",
    "args" : {"viewType":"list"}
  },
  {
    "type": "elem",
    "name": "elem",
    "path": "/ui/:modelName/:modelId",
    "import": "elements/model-ui-generator.html",
    "args" : {"viewType":"form"}
  }
]
```


2. Create the Navigation Link to navigate to the `Apprentice` list

```json
{
    "name": "ApprenticeList",
    "url": "/ui/apprentice",
    "label": "Apprentice",
    "icon": "social:person",
    "group": "root",
    "topLevel":true,
    "sequence":5
}
```


2. Finally define `UIElement`:

We create two `UIElement` entries one each for showing list of `Apprentice` and for `Apprentice` form.

```json
[
    {
        "name": "apprentice-form",
        "modelName": "Apprentice",
        "content" : "<ev-fields tplid='main' list='firstName,lastName'> </ev-fields><ev-fields tplid='details' list='birthDate,annualIncome'></ev-fields> ",
        "templateName": "default-form.html",
        "autoInjectFields" : true,
        "excludeFields" : ["nationality", "profession"]
    },
    {
        "name": "apprentice-list",
        "modelName": "Apprentice",
        "templateName": "default-list.html",
        "autoInjectFields" : false,
        "gridConfig" : {"modelGrid":[{"key":"firstName","label":"Custom Title"},"lastName","language","birthDate"]}
    }
]
```

`default-list.html` and `default-form.html` are the templates defined for generated elements.

### Defining templates

Defining and writing a new template is just like writing a Polymer component with following specifics:

You can use (:componentName, :modelName, :modelAlias, :plural) as placeholders which are replaced with actual
values.


```html
<dom-module id=":componentName">
  <template>
    <style>

    </style>
    <div class="content layout vertical">
      <div class="evform layout vertical">
        <div class="layout horizontal">
          <h2 class="flex">:modelName</h2>
          <div>
              <template is="toolbar"></template>
            <paper-button raised primary on-tap="doSave" ev-action-model=":modelAlias">Save</paper-button>
            <paper-button raised on-tap="doClear" ev-action-model=":modelAlias">Clear</paper-button>
            <template is="dom-if" if="{{:modelAlias.id}}">
              <paper-button raised on-tap="doFetch">Reset</paper-button>
              <paper-button raised on-tap="doDelete" ev-action-model=":modelAlias">Delete</paper-button>
            </template>
          </div>
        </div>
        <div id="fields" class="layout horizontal wrap">
        </div>
        <div id="grids" class="layout vertical">
        </div>
      </div>
    </div>
  </template>
  <script>
 
  	MetaPolymer({
        is: ":componentName",
        properties : {
            :modelAlias : {
              type : 'object',
              notify : true,
              value : {}
            }
        },
        behaviors:[EV.FormValidationBehavior,EV.ModelHandler]
    });
        
    </script>
</dom-module>
```


UIMetadata (Obsolete)
-------------

Rendering and managing forms generally poses a challange in keeping UI in sync with model changes. Furthermore, reflecting model validations into UI, being able to add extra client side validations, automatic handling of rest-APIs are certain standard and repetitive tasks that tend to make client side handling heavy and error-prone.  
`UIMetadata` provides a standard and easy way of maintaining your form UI along with dynamic validations based on model definitions along with handling standard ajax calls automatically [fetch, save, delete, refresh of form data].

`UIMetadata` model, available as part of ev-foundation can be used to define your form-UI. It has following properties,

### Properties/Settings

Property | Required | Default |         Description
---------|----------|---------|----------------------------
`code`   | yes      |    -    | the record key
`description`| no   |    -    | description
`modeltype`| no     |    -    | model type for form data. When specified, the metadata record is enriched at runtime with model properties.
`resturl`| no       | `modeltype.plural` | Rest end point where data are posted.
`controls`| yes     | derived from model properties | array of controls to display on form
`skipMissingProperties`| no | false | By default any model property, not listed in controls array are added dynamically. To disable auto injection of controls, set this property to true.
`exclude`| no | - | array of properties to explicitly exclude from auto injection.
`properties` | no | - | 'properties' to be added to the generated form element.
`functions` | no | - | 'functions' to be added to the generated form element.
`evValidations`| no | - | Collection of 'Validation' objects



### Controls

Each record in `controls` array represents a control on the form and can have following properties.

Property | Required | Default |         Description
---------|----------|---------|----------------------------
`fieldid`| yes | - | model property to which this control binds
`source`| no | - | All other properties can be populated from a source 'Field' model. This property refers to the `Field.key` property.
`container`| no | 'others' | container to which this control is added.
`uitype`| no | - | Control type 'text', 'boolean', 'number', 'date', 'typeahead', 'chips', 'grid'
`default`| no | - | Default value to be populated in the form for this control
`label` | no | - | Label / i18n label key
`required` | no | false | Whether the field is mandatory. This can be an expression on form model to dynamically decide whether field is mandatory or not.
`disabled` | no | false | Whether the field is disabled. This can be an expression on form model to dynamically decide whether field is enabled or disabled.
`hidden` | no | false | Whether the field is hidden. This can be an expression on form model to dynamically decide whether field is visible or hidden.
`class`  | no | - | Additional CSS class attributes to be set on control
`minlength`| no | - | Minimum length validation on input (text)
`maxlength`| no | - | Maximum length validation on input (text)
`min`| no | - | Minimum value validation on input (date, number). You can use short-hands in case of date.
`max`| no | - | Maximum length validation on input (date, number). You can use short-hands in case of date.
`pattern` | no | - | input pattern validation for text input
`precision`| no | - | input precision for decimal/number input
`listdata` | no | - | For combo-input, list of dropdown choices.
`listurl`  | no | - | Alternatively the input choices for combo can be retrieved dynamically from a rest url.
`bindto`   | no | - | Model to which control value is bound
`bindAttribute`   | no | - | user defined value-property of control.
`modeltype`   | no | - | incase uitype is oe-data-table, this value represents the Model whose properties will represent oe-data-table's columns.
`displayproperty` | no | - | [Combo and typeahead], operate on an array of selectable items. When the selectable-item is an object, you can extract one field from model to be displayed as menu title.
`valueproperty` | no | - | [Combo and typeahead], operate on an array of selectable items. When the selectable-item is an object, you can extract one field from model to be set as selected `value`.
`selectionBinding`| no | - | [Combo and typeaheaad] In some cases (e.g. Accounts drop down), while selected value is bound as `accountId`, it is desired to have the selected item also available so as to display may be accountName or accountBalance.

### Mechanics of form rendering

#### As part of server's response to UIMetadata request, following steps take place:

- UIMetadata record, corresponding to the requested 'code' that matches user's call context is selected.
- If the record has 'modeltype' property defined, corresponding model definition along with its properties and relationships is loaded.
- If controls have source property defined, these definitions are loaded from 'Field' collection.
- Finally controls properties are enriched as follows,
    - Value defined directly in UIMetadata control gets preference
    - Followed by source 'Field'
    - Lastly any value defined on the model are applied.
- However, for validations 'strictest-of-all' rules are applied.
	- If a property is defined as `required` on model, it can not be overridden to `required=false`.
	- Maximum of `minlength/min` and minimum of `maxlength/max` property is used.
	- If a property has a validateWhen condition for any of the validation rules that rule is not applied in the generated form for that property.In this case the server takes care of the validateWhen condition while we save the form data.
- Properties that are defined on model but are not listed in 'controls' array are added with defaults. The defaults are arrived using field-source (where source-key=model-property-name) and property settings defined at model level.

- `default` specified on all controls/source-field/model are used to form the default-view-model object as part of response.
- All boolean properties assume a default value of 'false' unless specified otherwise.


#### Rendering the form on client side

As explained above for UIRoute type 'meta', the final rendered form takes uimetadata name along with form-template as an input. 

- To render a metadata driven form or page, the metadata and html form-template are pulled from server.
- Each control defined in metadata is added to its corresponding container in form-template that bears `ev-container="container-name"` attribute. i.e. a control with `container:'main'` is added to 
an html container in template (div, ev-vbox, ev-hbox etc.) that has `ev-container="main"` attribute value.
- If the corresponding container is not found, the control is appended in the end.

#### Form Navigation handling

- `ev-selection-group='tabs'`
> - The form-template may have `<iron-pages>` linked to `paper-tabs`. Handling of `selected` tab is required as default value for `selected` has to be set to 0 so that first tab is visible by default.

>  - This can be achieved by setting `ev-selection-group="mytabs"` on both paper-tabs and iron-pages. The form rendering automatically binds paper-tabs and iron-pages to a single property with default as 0, there by showing first tab as default.

- `ev-<group>-next` and `ev-<group>-prev`
>	The form can also be some kind of wizard form where one iron-page is displayed at a time and user can click next/prev to browse through steps.
>For `<iron-pages ev-selection-group='mywizardform'>`,
> - Add an icon, button, paper-fab and give attribute as `ev-mywizardform-next` to auto handle 'next' action.
> - Add an icon, button, paper-fab and give attribute as `ev-mywizardform-prev` to auto handle 'prev' action.


#### Form action handling

`ev-action=[new,save,delete,reset,refresh]`

- ev-action="new" resets the VM with `defaultVM`
- ev-action="reset", resets the VM with original values
- ev-action="refresh", pulls the latest data (corresponding to modelId) from server and sets as 'VM'
- ev-action="save", posts the current VM on the `resturl` 
- ev-action="delete", sends a `delete` request on the `resturl`

Completion of insert/update/delete/fetch raises following events
`ev-formdata-inserted, ev-formdata-updated, ev-formdata-deleted, ev-formdata-loaded`. These are default handled in `ev-app-shell` to display appropriate message toast.
            
#### Lazy data fetch

For cases where some extra data needs to be fetched from server you can use `ev-action="request-response"` as below

`<paper-fab icon="icons:fetch" 
                   ev-action="request-response"
                   request-url="/api/details"
                   response-to="extraDetails">
 </paper-fab>`

### UIMetadata Form without route change

`<ev-meta-component>`