# ampersand-collection-jquery-datatable

# What

* You use [ampersand.js](https://ampersandjs.com/).
* You use [jQuery DataTables](https://datatables.net/).
* You have an [ampersand collection](https://github.com/AmpersandJS/ampersand-collection) that you want to render in a DataTable.
* You want your dataTable to respond to collection events (i.e. `add`, `remove`, `update`), and be displayed immediately, without having to get into the dataTable API on your own.

How refreshing.

# How

## Pre-conditions
* You should already have jQuery on the window or in your dependencies
* You should already have the DataTables plugin loaded onto jQuery

## Process
Feel free to take a peek at the `src/examples/dynamic-rows.js` && `src/examples/dynamic-rows.html` files.

The process is generally:

1. Generate a config object, e.g. `var config = {};`
1. Set an `el` property, where `config.el = raw_DOM_node_for_table;`  e.g `config.el = window.document.getElementById('my_table')`;
1. Set a `collection` property, where the value is some type of `ampersand-collection`
1. Add any table options that you would normally use for the DataTable to `.dtOptions`
1. Add any styles you would like to add to the table element to `.dtClasses` e.g. `display compact`
1. `var CDT = require('ampersand-collection-jquery-datatable')` (sorry for the long name!), and construct via `var myCollDataTable = new CDT(confg)`
1. To see other options, **check the contructor DocBlock**.  The latest should always be maintained here:

```js
/**
 * Constructor
 * @param  {options} object {
 *     collection: {AmpersandCollection} // ampersand collection,
 *     el: {Element}, // dataTable target
 *     dtOptions: {Object}, // dataTable config.  Set the `data` and `columns` props! *     dtClasses: {String=},  // classes to be applied to the target element/table
 *     noToolbar: {Boolean=},
 *     renderer: {Function} // delegate a custom function to init the dataTable.
 *                          // renderer receives `($el, modified-dtOptions)`,
 *                          // and should return the result of `$el.DataTables(...)
 * }
 * @return {CollectionDataTable}
 */
```

## Additional Features

* `.stateNodes` - Access the row DOM node for you state/model by peeking at `yourCollectionTable.stateNodes[yourModel.cid]`


# Full Example
```js
var jQuery = window.jQuery = require('jquery'),
    Collection = require('ampersand-collection'),
    Model = require('ampersand-model'),
    TestModel = Model.extend({ props: {a: 'string', b: 'string'} }),
    CollectionTable = require('../../ampersand-collection-jquery-datatable'),
    datatables= require('datatables'),
    domready = require('domready');

var colDefs = [{title: 'a', data: 'a'}, {title: 'b', data: 'b'}];
var dummyData = [
    new TestModel({a: 'a1', b: 'b1'}),
    new TestModel({a: 'a2', b: 'b2'})
];

domready(function() {
    var $myDt = jQuery('#myTable');
    var TestCollection = Collection.extend({
        indexes: ['a', 'b'],
        model: TestModel
    });
    var dummyCollection = new TestCollection();
    var acjd = new CollectionTable({
        collection: dummyCollection,
        el: $myDt[0],
        dtOptions: {
            data: dummyData,
            columns: colDefs
        }
    });

    // Test add/remove/update
    var newObj = new TestModel({a: 'new', b: 'new'});
    dummyCollection.add(newObj);  // dataTable responds, adds row!

    // remove obj
    var removeObj = new TestModel({a: 'REMOVE', b: 'REMOVE'});
    dummyCollection.add(removeObj);
    dummyCollection.remove(removeObj); // dataTable responds, removes row!

    // change
    dummyCollection
        .add({a: 'needs-to-be-updated', b: 'needs-to-be-updated'});
    var toChange = dummyCollection.get('nees-to-be-updated', 'a');
    toChange.a = 'updated-successfully';
    toChange.b = 'updated-successfully';
    // dataTable responds, deletes old row, adds row with new data back in
    // as your model changes.  Note: if your model changes a lot, this is expensive
    // re-drawing.  A debounce may worth implementing, or simply refreshing the row
    // from an updated data source

});
```
# Edge Cases and Warnings

1. Using the `init.dt` event and `initComplete` options may yield unexpected behavior.  Post-initialization an *empty table is drawn*.  ACJD then immediately adds row data, and re-draws.  Your defined `initComplete` is executed then.  The `init.dt` (or similair) event is not rethrown.  PRs are welcome in this regard.
1. Dynamic columns is not yet supported by DataTables (sorry!).  It will likely be in a 10.4/5/X or 11.X release?
1. If you are changing your models often or in bulk, you may want a mitigation strategy as it attempts to redraw on every add/delete, with some minor exceptions.

# Changelog

* 1.2.2 - Carry applied `classes` from old `tr` to new `tr` onchange.  Additional attrs may need consideration as well
* 1.2.1 - Drastically improve `change` on the collection--rows are no longer deleted and re-added.  Instead, the `.data()` setter is used, even if the data pointer is the same as the original, which triggers a view update on that row.
* 1.2.0 - `initComplete` handling update.  initComplete must be called after the table is instantiated due to the way that the table data is populated in ACJD.  Additionally, `.$dt` now points to the actual DT instance, and `.$api` points to a `.$dt.api()`.
* 1.1.2 - doc updates only
* 1.1.1 - bugfix & feature.  removing a state would sometimes scrap the full table. improved the indexing by looking up models in table by DOM node via `.stateNodes`
* 1.1.0 - added `.renderer` option.  some users have pre-defined utilities to pipe table options thru prior to initialization

# ToDo

1. Some tests, really.
1. Debounce for bulk changes
