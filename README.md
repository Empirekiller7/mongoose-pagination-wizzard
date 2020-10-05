# mongoose-pagination-wizard

[![npm version](https://img.shields.io/npm/v/mongoose-pagination-wizard.svg)](https://www.npmjs.com/package/mongoose-pagination-wizard)
[![Build Status](https://travis-ci.org/Empirekiller7/mongoose-pagination-wizard.svg?branch=master)](https://travis-ci.org/Empirekiller7/mongoose-pagination-wizard)
[![Dependency Status](https://david-dm.org/Empirekiller7/mongoose-pagination-wizard.svg)](https://david-dm.org/Empirekiller7/mongoose-pagination-wizard)
[![devDependencies Status](https://david-dm.org/Empirekiller7/mongoose-pagination-wizard/dev-status.svg)](https://david-dm.org/Empirekiller7/mongoose-pagination-wizard?type=dev)

> A cursor based custom pagination library for [Mongoose](http://mongoosejs.com) with customizable labels.

> This library is a fork from [aravindnc's pagination library](https://github.com/aravindnc/mongoose-paginate-v2).

> I added the search and column sorting features and linted with Eslint. These features were inspired on [archr's datatables library](https://github.com/archr/mongoose-datatables).

>All deserved credits go to Aravind NC (aravindc) (https://aravindnc.com) and the contribuitors of his library, and also to Antonio Sandoval (archr).
<br>

[![NPM](https://nodei.co/npm/mongoose-pagination-wizard.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/mongoose-pagination-wizard)

If you are looking for aggregate query pagination, use this one [mongoose-aggregate-paginate-v2](https://github.com/aravindnc/mongoose-aggregate-paginate-v2)

## Why This Plugin

mongoose-pagination-wizard is a cursor based pagination library (based on [mongoose-paginate-v2](https://github.com/aravindnc/mongoose-paginate-v2)) having a page wrapper. The plugin can be used as both page as well as cusror based pagination. The main usage of the plugin is you can alter the return value keys directly in the query itself so that you don't need any extra code for transformation.

This version of the library adds the search field and column sorting features from [mongoose-datatables](https://github.com/archr/mongoose-datatables).

The below documentation is not perfect. Feel free to contribute. :)

## Installation

```sh
npm install mongoose-pagination-wizard
```

## Usage

Add plugin to a schema and then use model `paginate` method:

```js
const mongoose = require("mongoose");
const mongoosePaginate = require("mongoose-pagination-wizard");

const mySchema = new mongoose.Schema({
  /* your schema definition */
});

mySchema.plugin(mongoosePaginate);

const myModel = mongoose.model("SampleModel", mySchema);

myModel.paginate().then({}); // Usage
```

### Model.paginate([query], [options], [callback])

Returns promise

**Parameters**

- `[query]` {Object} - Query criteria. [Documentation](https://docs.mongodb.org/manual/tutorial/query-documents)
- `[options]` {Object}
  - `[select]` {Object | String} - Fields to return (by default returns all fields). [Documentation](http://mongoosejs.com/docs/api.html#query_Query-select)
  - `[collation]` {Object} - Specify the collation [Documentation](https://docs.mongodb.com/manual/reference/collation/)
  - `[sort]` {Object | String} - Sort order. [Documentation](http://mongoosejs.com/docs/api.html#query_Query-sort)
  - `[populate]` {Array | Object | String} - Paths which should be populated with other documents. [Documentation](http://mongoosejs.com/docs/api.html#query_Query-populate)
  - `[projection]` {String | Object} - Get/set the query projection. [Documentation](https://mongoosejs.com/docs/api/query.html#query_Query-projection)
  - `[lean=false]` {Boolean} - Should return plain javascript objects instead of Mongoose documents? [Documentation](http://mongoosejs.com/docs/api.html#query_Query-lean)
  - `[leanWithId=true]` {Boolean} - If `lean` and `leanWithId` are `true`, adds `id` field with string representation of `_id` to every document
  - `[offset=0]` {Number} - Use `offset` or `page` to set skip position
  - `[page=1]` {Number}
  - `[limit=10]` {Number}
  - `[customLabels]` {Object} - Developers can provide custom labels for manipulating the response data.
  - `[pagination]` {Boolean} - If `pagination` is set to false, it will return all docs without adding limit condition. (Default: True)
  - `[forceCountFn]` {Boolean} - Set this to true, if you need to support \$geo queries.
  - `[read]` {Object} - Determines the MongoDB nodes from which to read. Below are the available options.
    - `[pref]`: One of the listed preference options or aliases.
    - `[tags]`: Optional tags for this query. (Must be used with `[pref]`)
  - `[options]` {Object} - Options passed to Mongoose's `find()` function. [Documentation](https://mongoosejs.com/docs/api.html#query_Query-setOptions)
  <br>
    - <b>Suported Options</b>:
      - collation
      - lean
      - leanWithId
      - populate
      - projection
      - read
      - select
      - sort
      - pagination
      - search
      - order
      - columns
      - forceCountFn
  <br>
- `[callback(err, result)]` - If specified, the callback is called once pagination results are retrieved or when an error has occurred

**Return value**

Promise fulfilled with object having properties:

- `docs` {Array} - Array of documents
- `totalDocs` {Number} - Total number of documents in collection that match a query
- `limit` {Number} - Limit that was used
- `hasPrevPage` {Bool} - Availability of prev page.
- `hasNextPage` {Bool} - Availability of next page.
- `page` {Number} - Current page number
- `totalPages` {Number} - Total number of pages.
- `offset` {Number} - Only if specified or default `page`/`offset` values were used
- `prevPage` {Number} - Previous page number if available or NULL
- `nextPage` {Number} - Next page number if available or NULL
- `pagingCounter` {Number} - The starting sl. number of first document.
- `meta` {Object} - Object of pagination meta data (Default false).

Please note that the above properties can be renamed by setting customLabels attribute.

### Sample Usage

#### Return first 10 documents from 100

```javascript
const options = {
  page: 1,
  limit: 10,
  collation: {
    locale: "en",
  },
};

Model.paginate({}, options, function (err, result) {
  // result.docs
  // result.totalDocs = 100
  // result.limit = 10
  // result.page = 1
  // result.totalPages = 10
  // result.hasNextPage = true
  // result.nextPage = 2
  // result.hasPrevPage = false
  // result.prevPage = null
  // result.pagingCounter = 1
});
```

### Search Fields

Filter your model's fields by value. Since it uses the <b>$regex</b> operator, it currently only suports search for string value fields.
```javascript
 var const = {
      limit: 10,
      page: 1,
      lean: true,
      search: {
        value: 'Book of pagination',
        fields: ['title', 'shortName', 'abstract']
      }
 }

 Model.paginate({}, options, function (err, result) {
    //docs filtered by search value
});
```
### Sorting

Provide the 'sort' property with the values 1 for ascending, or -1 for descending order sorting.

### Column Sorting

Filter your model's fields by columns, provivind the type of sorting ('asc' or 'desc'). You <b>MUST</b> provide both the <b>columns</b> and <b>order</b>  properties. This is very useful for datatable support.

```javascript
 var const = {
      limit: 10,
      page: 1,
      lean: true,
      columns: [
         { data: 'email' }
         { data: 'name' },
           ...
      ],
      order: [
        {
          column: 0,
          dir: 'asc'
        },
        {
          column: 1,
          dir: 'desc'
        }
          ...
      ]
 }

 Model.paginate({}, options, function (err, result) {
    //docs sorted by column
});
```
### With custom return labels

Now developers can specify the return field names if they want. Below are the list of attributes whose name can be changed.

- totalDocs
- docs
- limit
- page
- nextPage
- prevPage
- hasNextPage
- hasPrevPage
- totalPages
- pagingCounter
- meta

You should pass the names of the properties you wish to changes using `customLabels` object in options.
Set the property to false to remove it from the result.
Same query with custom labels

```javascript
const myCustomLabels = {
  totalDocs: "itemCount",
  docs: "itemsList",
  limit: "perPage",
  page: "currentPage",
  nextPage: "next",
  prevPage: "prev",
  totalPages: "pageCount",
  pagingCounter: "slNo",
  meta: "paginator",
};

const options = {
  page: 1,
  limit: 10,
  customLabels: myCustomLabels,
};

Model.paginate({}, options, function (err, result) {
  // result.itemsList [here docs become itemsList]
  // result.paginator.itemCount = 100 [here totalDocs becomes itemCount]
  // result.paginator.perPage = 10 [here limit becomes perPage]
  // result.paginator.currentPage = 1 [here page becomes currentPage]
  // result.paginator.pageCount = 10 [here totalPages becomes pageCount]
  // result.paginator.next = 2 [here nextPage becomes next]
  // result.paginator.prev = null [here prevPage becomes prev]
  // result.paginator.slNo = 1 [here pagingCounter becomes slNo]
  // result.paginator.hasNextPage = true
  // result.paginator.hasPrevPage = false
});
```

### Other Examples

Using `offset` and `limit`:

```javascript
Model.paginate({}, { offset: 30, limit: 10 }, function (err, result) {
  // result.docs
  // result.totalPages
  // result.limit - 10
  // result.offset - 30
});
```

With promise:

```js
Model.paginate({}, { offset: 30, limit: 10 }).then(function (result) {
  // ...
});
```

#### More advanced example

```javascript
var query = {};
var options = {
  select: "title date author",
  sort: { date: -1 },
  populate: "author",
  lean: true,
  offset: 20,
  limit: 10,
};

Book.paginate(query, options).then(function (result) {
  // ...
});
```

#### Zero limit

You can use `limit=0` to get only metadata:

```javascript
Model.paginate({}, { limit: 0 }).then(function (result) {
  // result.docs - empty array
  // result.totalDocs
  // result.limit - 0
});
```

#### Set custom default options for all queries

config.js:

```javascript
var mongoosePaginate = require("mongoose-pagination-wizard");

mongoosePaginate.paginate.options = {
  lean: true,
  limit: 20,
};
```

controller.js:

```javascript
Model.paginate().then(function (result) {
  // result.docs - array of plain javascript objects
  // result.limit - 20
});
```

#### Fetch all docs without pagination.

If you need to fetch all the documents in the collection without applying a limit. Then set `pagination` as false,

```javascript
const options = {
  pagination: false,
};

Model.paginate({}, options, function (err, result) {
  // result.docs
  // result.totalDocs = 100
  // result.limit = 100
  // result.page = 1
  // result.totalPages = 1
  // result.hasNextPage = false
  // result.nextPage = null
  // result.hasPrevPage = false
  // result.prevPage = null
  // result.pagingCounter = 1
});
```

#### Setting read preference.

Determines the MongoDB nodes from which to read.

```js
const options = {
  lean: true,
  limit: 10,
  page: 1,
  read: {
    pref: "secondary",
    tags: [
      {
        region: "South",
      },
    ],
  },
};

Model.paginate({}, options, function (err, result) {
  // Result
});
```

Below are some references to understand more about preferences,

- https://github.com/Automattic/mongoose/blob/master/lib/query.js#L1008
- https://docs.mongodb.com/manual/core/read-preference/
- http://mongodb.github.io/node-mongodb-native/driver-articles/anintroductionto1_1and2_2.html#read-preferences

## Note

There are few operators that this plugin does not support natively, below are the list and suggested replacements,

- $where: $expr
- $near: $geoWithin with \$center
- $nearSphere: $geoWithin with \$centerSphere

But we have added another option. So if you need to use $near and $nearSphere please set `forceCountFn` as true and try running the query.

```js
const options = {
  lean: true,
  limit: 10,
  page: 1,
  forceCountFn: true,
};

Model.paginate({}, options, function (err, result) {
  // Result
});
```

## License

[MIT](LICENSE)
