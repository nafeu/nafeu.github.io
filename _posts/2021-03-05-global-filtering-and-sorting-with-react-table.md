---
layout: post
title: "Global Filtering and Sorting with React Table"
description: "Implementing a global filter and column sorting with React Table"
date: 2021-03-05
tags: [coding,react-table,javascript,sorting,filtering]
comments: true
share: true
---

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614631393/slec2nfokpvtvk2qekss.png)

This post is a continuation of my article on [Using React Query with React Table](https://nafeu.medium.com/using-react-query-with-react-table-884158535424). I recommend reading that first to understand the full context.

We will also be following along with the same set of example code which can be found at [github.com/nafeu/react-query-table-sandbox](https://github.com/nafeu/react-query-table-sandbox). We will be focusing on the [ReactTableFilterSort.jsx](https://github.com/nafeu/react-query-table-sandbox/blob/main/src/ReactTableFilterSort.jsx) file.

You can get it quickly set up with:

```bash
git clone https://github.com/nafeu/react-query-table-sandbox.git
cd react-query-table-sandbox
npm install
npm start
```

#### What We Are Building

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614631580/ap0kgbanwz3gnglfxi8h.gif)

### Understanding Our Data

In the [example code](https://github.com/nafeu/react-query-table-sandbox/blob/main/index.mjs), I prepared a **mock api** which returns data for an imaginary **software dev discussion group** website. The data is formatted like so:

```
[
  {
    "id": 1,
    "name": "How to use react-table in reporting dashboard",
    "active": 40,
    "status": "locked",
    "upvotes": 30
  },
  {
    "id": 2,
    "name": "How to use react-query for BI solution",
    "active": 31,
    "status": "resolved",
    "upvotes": 39
  }
  ...
]
```

For the purpose of this tutorial, our data will not get any more complex than this. How can we now leverage React Table to implement a global filter and sortability into our table? We will do so by modifying our existing `ReactQueryWithTable` example. Let's get started.

### Adding a Global Filter Component

First we want to import some additional hooks from the `react-table` library, namely the `useGlobalFilter`, `useAsyncDebounce` and `useSortBy` hooks:

```javascript
import {
  useTable,
  useGlobalFilter,
  useAsyncDebounce,
  useSortBy
} from 'react-table';
```

Then we want to create a new component called `GlobalFilter` which takes a few React Table defined props and renders a search box for us:

```javascript
const TWO_HUNDRED_MS = 200;

function GlobalFilter({
  preGlobalFilteredRows,
  globalFilter,
  setGlobalFilter,
}) {
  const [value, setValue] = useState(globalFilter);
  const onChange = useAsyncDebounce(value => {
    setGlobalFilter(value || undefined)
  }, TWO_HUNDRED_MS);

  return (
    <input
      value={value || ""}
      onChange={e => {
        setValue(e.target.value);
        onChange(e.target.value);
      }}
      placeholder={`Search`}
    />
  )
}
```


Using `useState` and `globalFilter`, we create a `value` state variable which keeps track of what we put into our search box, and we use `useAsyncDebounce` to call `setGlobalFilter` with a `200ms` input delay.

We can't really see it here, but `setGlobalFilter` performs quite a bit of magic. As our `globalFilter` object is connected to the rest of the table, updates from `setGlobalFilter` will modify the visible rows in our table.

### Updating our TableQuery Component

The only modification we do here is add `enabled: !tableData` to prevent the table from re-fetching in the background as it can mess up the filtered or sorted order. Implementing re-fetching with a filtered or sorted structure is a more advanced topic that won't be covered in this article.

```javascript
const {
  data: apiResponse,
  isLoading
} = useQuery('discussionGroups', fetchData, { enabled: !tableData });
```

### Updating our TableInstance Component

Here we just update the `tableInstance` instantiation with the hooks `useGlobalFilter` and `useSortBy`

```javascript
const tableInstance = useTable(
  { columns, data },
  useGlobalFilter,
  useSortBy
);
```

### Updating our TableLayout Component

Here is where the bulk of our updates are in comparison to the `ReactQueryWithTable.jsx` file:

```javascript
const TableLayout = ({
  getTableProps,
  getTableBodyProps,
  headerGroups,
  rows,
  prepareRow,
  state: { globalFilter },
  visibleColumns,
  preGlobalFilteredRows,
  setGlobalFilter
}) => {
  return (
    <table {...getTableProps()}>
      <thead>
        <tr>
          <th
            colSpan={visibleColumns.length}
          >
            <GlobalFilter
              preGlobalFilteredRows={preGlobalFilteredRows}
              globalFilter={globalFilter}
              setGlobalFilter={setGlobalFilter}
            />
          </th>
        </tr>
        {headerGroups.map(headerGroup => (
          <tr {...headerGroup.getHeaderGroupProps()}>
            {headerGroup.headers.map(column => (
              <th {...column.getHeaderProps(column.getSortByToggleProps())}>
                {column.render('Header')}
                <span>
                  {column.isSorted
                    ? column.isSortedDesc
                      ? ' ⬇️'
                      : ' ⬆️'
                    : ' ↕️'}
                </span>
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody {...getTableBodyProps()}>
        {rows.map((row, i) => {
          prepareRow(row)
          return (
            <tr {...row.getRowProps()}>
              {row.cells.map(cell => {
                return <td {...cell.getCellProps()}>{cell.render('Cell')}</td>
              })}
            </tr>
          )
        })}
      </tbody>
    </table>
  );
}
```


We pull out the following additional props from our table instance:

```txt
state: { globalFilter },
visibleColumns,
preGlobalFilteredRows,
setGlobalFilter
```

These allow us to access the `globalFilter` object, info on `visibleColumns`, the `globalFilter` setter and the `preGlobalFilteredRows` helper.

Inside our `<thead>` we insert a new `<tr>`, set it's `colSpan` to the count of `visibleColumns` and insert our `GlobalFilter` component like so:

```javascript
<tr>
  <th
    colSpan={visibleColumns.length}
  >
    <GlobalFilter
      preGlobalFilteredRows={preGlobalFilteredRows}
      globalFilter={globalFilter}
      setGlobalFilter={setGlobalFilter}
    />
  </th>
</tr>
```

This is what it renders:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614633798/wzf75sipcdgpiw1esekq.png)

After this `<tr>` and inside the same `<thead>`, we include a modified version of the `headerGroups` like so:

```javascript
{headerGroups.map(headerGroup => (
  <tr {...headerGroup.getHeaderGroupProps()}>
    {headerGroup.headers.map(column => (
      <th {...column.getHeaderProps(column.getSortByToggleProps())}>
        {column.render('Header')}
        <span>
          {column.isSorted
            ? column.isSortedDesc
              ? ' ⬇️'
              : ' ⬆️'
            : ' ↕️'}
        </span>
      </th>
    ))}
  </tr>
))}
```


One of the most important things here is the use of `column.getSortByToggleProps()`, this is what allows us to hook into React Table's sorting functionality.

Honestly, that is all it takes. We add some extra UI elements in the form of:

```javascript
<span>
  {column.isSorted
    ? column.isSortedDesc
      ? ' ⬇️'
      : ' ⬆️'
    : ' ↕️'}
</span>
```

Which display the sorting state. They utilize `.isSorted` and `.isSortedDesc` which again, are values provided to us by React Table thanks to `useSortBy`.

And there you have it, that's all it takes to implement a global filter and sorting using React Table. We are using all default configurations here, but React Table also allows you to control the algorithms behind sorting and filtering. Hope this has helped.

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614631580/ap0kgbanwz3gnglfxi8h.gif)

Happy Coding!