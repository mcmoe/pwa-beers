# ![](../img/logo-25px.png) PWA Beers - Step 04 - Adding full text search

> This is an optional step, that helps you dive deeper into Polymer. If your main interest in *PWA Beers* is the PWA aspect, you can simply copy the content of the `step-04` folder into your working folder `app`.

We did a lot of work in laying a foundation for the app in the last step, so now we'll do something simple;
we will add full text search (yes, it will be simple!).

We want to add a search box to the app, and we want the results on the beer list change according to what the user types into the search box.

## Modifying `beer-list` local DOM

We are using the [iron-flex-layout](https://www.webcomponents.org/element/PolymerElements/iron-flex-layout) component to create a responsive two columns layout using a flexbox CSS model. If you want to better understand how `iron-flex-layout` works, we suggest you the [Flexbox layout with iron-flex-layout guide](https://elements.polymer-project.org/guides/flex-layout).

We need to add a  [`paper-input`](https://www.webcomponents.org/element/PolymerElements/paper-input) element for the input fields, give them some magical data-binding properties and adding a Polymer [filter](https://www.polymer-project.org/2.0/docs/devguide/templates#filtering-and-sorting-lists) function to process the input for the [dom-repeat](https://www.polymer-project.org/2.0/docs/devguide/templates#dom-repeat) template repeater.  

This lets a user enter search criteria and immediately see the effects of their search on the beer list.  

Let's begin by adding `iron-flex-layout` and `paper-input` as dependencies:

```
bower install --save PolymerElements/iron-flex-layout PolymerElements/paper-input
```

Now our `bower.json` should look like:

```json
{
  "name": "pwa-beers",
  "description": "PWA Beers with Polymer 2.x",
  "main": "index.html",
  "dependencies": {
    "polymer": "Polymer/polymer#^2.0.0",
    "app-route": "PolymerElements/app-route#^2.0.0",
    "iron-pages": "PolymerElements/iron-pages#^2.0.0",
    "iron-selector": "PolymerElements/iron-selector#^2.0.0",
    "paper-toolbar": "PolymerElements/paper-toolbar#^2.0.0",
    "paper-material": "PolymerElements/paper-material#^2.0.0",
    "iron-flex-layout": "PolymerElements/iron-flex-layout#^2.0.0",
    "paper-input": "PolymerElements/paper-input#^2.0.0"
  },
  "devDependencies": {
    "web-component-tester": "Polymer/web-component-tester#^6.0.0",
    "webcomponentsjs": "webcomponents/webcomponentsjs#^1.0.0"
  }
}
```

And then, let's import both components into `beer-list`:

```html
<link rel="import" href="../../bower_components/iron-flex-layout/iron-flex-layout-classes.html">
<link rel="import" href="../../bower_components/paper-input/paper-input.html">
```

We notify to our element that we want it to use the `iron-flex-layout` shared styles, we add some CSS rules, and we place the `paper-input`:

```html
<dom-module id="beer-list">
  <template>
    <!-- local DOM for your element -->
    <style include="iron-flex iron-flex-alignment">
      :host {
        display: block;
      }
      paper-material {
        margin: 10px;
        padding: 10px;
        background-color: white;
      }
      .sidebar {
        min-width: 150px;
        min-height: 50px;
      }
      .beers {
        min-width: 400px;
      }
    </style>
    <div class="layout horizontal wrap start center-justified">
      <paper-material class="sidebar">
        <!--Sidebar content-->
        Search: <paper-input></paper-input>
      </paper-material>
      <div class="beers layout flex" >
        <dom-repeat  id="beerList" items="{{beers}}">
          <template>
            <beer-list-item name="{{item.name}}" description="{{item.description}}">
            </beer-list-item>
          </template>
        </dom-repeat>

        <paper-material>Number of beers in list: {{beers.length}}</paper-material>  
      </div>        
    </div>
  </template>
```


## Two-ways data-binding

Now we need to link the search input field value to a property of the object.

In the template we use value to link the `input` event of the `input` item to the `filterText` property,
and we add a label under it to show the current value of `filterText`:

```html
<div>Search: <paper-input value="{{filterText}}"></paper-input></div>
<div>Current search:</div>
<div>{{filterText}}</div>

```
In the element registration we declare the `filterText` property as a string:

```javascript
static get properties() {
  return {
    beers: {
      type: Array,
      // When initializing a property to an object or array value, use a function to ensure that each element
      // gets its own copy of the value, rather than having an object or array shared across all instances of the element
      value: function() { return []; }
    },
    filterText: {
      type: String
    }
  }
} 
```
And now we have a two-way data-binding between the input field and the label under it.

[![Screenshot](../img/step-04_01.t.jpg)](../img/step-04_01.jpg)


## Adding a filter to the list

To filter or sort the displayed items in your list, we specify a `filter` or `sort` property on the `dom-repeat` (or both):

+ `filter`: it specifies a filter callback function following the standard [Array `filter` API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

+ `sort`: it specifies a comparison function following the standard [Array `sort` API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

In both cases, the value can be either a function object, or a string identifying a function defined on the host element.

The only problem in our case is that, by default, the filter and sort functions only run when the array itself is mutated (for example, by adding or removing items). And we want it to run when we modify an outside property, `filterText`.In this case, we can use a computed binding to return a dynamic filter (or sort) function when one or more dependent properties changes.


```html
<dom-repeat  id="beerList" items="{{beers}}"  filter="[[computeBeerFilter(filterText)]]">
  <template>
    <beer-list-item name="{{item.name}}" description="{{item.description}}">
    </beer-list-item>
  </template>
</dom-repeat>
```

```javascript
computeBeerFilter(filterText) {
  if (!filterText) {
    // set filter to null to disable filtering
    return null;
  }
  return function(item) {
    return item.name.match(new RegExp(filterText, 'i'));
  }
}
```   
 
And we have a working filter for our beers!

[![Screenshot](../img/step-04_02.t.jpg)](../img/step-04_02.jpg)

## Additional experiments

You have maybe noticed it, the *total beers* that you added in the *additional experiments* section some steps ago shows only that, the total. It would be nice if it showed the *current beers* metric, i.e. the number of beers currently showed in page, after filtering.

How could you do it? Polymer has several kinds of properties, and one of them is going to help us here: the  [computed properties](https://www.polymer-project.org/2.0/docs/devguide/observers#computed-properties). These are virtual properties whose values are calculated from other properties.

To define a computed property, add it to the properties object with a computed key mapping to a computing function.

We need a virtual properties, `currentBeers` that watches `beers` and `filterText` and that recalculates the number of beers on the page.

```javascript
static get properties() {
  return {
    beers: {
      type: Array,
      // When initializing a property to an object or array value, use a function to ensure that each element
      // gets its own copy of the value, rather than having an object or array shared across all instances of the element
      value: function() { return []; }
    },
    filterText: {
      type: String,
      value: ''
    },
    currentBeers: {
      type: String,
      computed: "getCurrentBeers(beers, filterText)"
    }
  }
}       
    
[...]

getCurrentBeers() {
    // Do something clever...
}
```

Inside the function we can go through `beers` and incrementing a counter if the beer matches the `beerFilter` filter.

[![Screenshot](/img/step-04_03.t.jpg)](/img/step-04_03.jpg)

## Next

We have now added full text search! Now let's go on to [step-05](../step-05) to learn how to add sorting capability to the beer app.
