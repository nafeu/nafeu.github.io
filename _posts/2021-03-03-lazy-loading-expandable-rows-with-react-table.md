---
layout: post
title: "Lazy-Loading Expandable Rows with React Table"
description: "An implementation of lazy-loading, expandable rows using React Table and React Query"
date: 2021-03-03
tags: [coding,dev,react-query,react-table,javascript]
comments: true
share: true
---

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614630753/mjaokhagbhftysjjxd1u.png)

This post is a continuation of my article on [Using React Query with React Table](https://nafeu.medium.com/using-react-query-with-react-table-884158535424). I recommend reading that first to understand the full context.

We will also be following along with the same set of example code which can be found at [github.com/nafeu/react-query-table-sandbox](https://github.com/nafeu/react-query-table-sandbox). We will be focusing on the [ReactTableExpanding.jsx](https://github.com/nafeu/react-query-table-sandbox/blob/main/src/ReactTableExpanding.jsx) file.

You can get it quickly set up with:

```bash
git clone https://github.com/nafeu/react-query-table-sandbox.git
cd react-query-table-sandbox
npm install
npm start
```

#### The Expandable Table We Want To Build

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614625868/aoflp1y1x5a0gmc83pit.gif)

### Understanding Our Data

When we think of table data, it's easier to visualize when we consider it to be in the form of a flat structure, for instance, the following collection only has a single layer of depth:

```
[
  { id: 1, name: 'foo' },
  { id: 2, name: 'bar' },
  { id: 3, name: 'baz' }
]
```

And in a table would render:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614625266/ii54imvarxkwkvlneggb.png)

But what if we had more of a hierarchical (or tree-like) structure such as:

```
[
  {
    id: 1,
    name: 'foo'
    children: [
      { id: 4, name: 'qux' },
      { id: 5, name: 'quux' }
    ]
  },
  { id: 2, name: 'bar' },
  { id: 3, name: 'baz' }
]
```

This is where things get wonky and we have to shift our approach a tiny bit. Let's say we want to render this, but only render the `children` when we click on an individual row. For example, if we were to click on the row of `id: 1`, we would want to see a flat table rendered like:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614625301/yci97q9pgj5ih95fmovs.png)

In the real world, we have many instances where this is the case and they come in the form of drill-down tables. This is helpful for BI dashboards, video game leaderboards, accounting, or pretty much any case where you need further investigate a single row in a table.

In [`index.mjs`](https://github.com/nafeu/react-query-table-sandbox/blob/main/index.mjs) from our example code, we have defined an API that returns mock data about an imaginary **software dev discussion group**.

The `api/` endpoint returns data in the form of:

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

The `api/child` endpoint returns similar data but provides new entries every time, whereas the root `api/` endpoint returns the original payload with slight modifications. This is to simulate an "active website" where data changes in pseudo real-time.

Although the example is a bit rough, imagine for a moment that each of these _discussion topics_ have _sub-issues_ which are structured in the same way. These will be used to persist a "hierarchical" structure in our table.

In our [front-end code](https://github.com/nafeu/react-query-table-sandbox/blob/main/src/ReactTableExpanding.jsx), we have:

```javascript
const fetchParentData = () => axios.get(`http://localhost:8000/api`);
const fetchChildData = () => axios.get(`http://localhost:8000/api/child`);
```

Which are promise-based API helpers that retrieve the partially dynamic **parent** data and novel set of **child** data respectively (it is required for React Query to use **promise-based** API helpers).

### Recursively Modifying Row Data

Before we dig into the react code, we have to figure out a way of recursively inserting one set of `rows` into an existing set of `rows`, for instance, how do we get from:

```txt
row-0
row-1
row-2
```

to

```txt
row-0
  row-0.0
  row-0.1
row-1
row-2
```

and even further to

```txt
row-0
  row-0.0
    row-0.1.0
    row-0.2.1
  row-0.2
row-2
row-3
```

and so on and so forth. In our example code, we do this using the `recursivelyUpdateTable` function which is as follows:

```javascript
const recursivelyUpdateTable = ({ tableData, childData, id }) => {
  return insertIntoTable({
    existingRows: tableData,
    subRowsToInsert: childData,
    path: id.split('.')
  });
}

const insertIntoTable = ({ existingRows, subRowsToInsert, path }) => {
  const id = path[0];
  let updatedRows;

  const isBaseCase = path.length === 1;

  if (isBaseCase) {
    return existingRows.map((row, index) => {
      const isMatchedRowWithoutSubRows = index === Number(id) && !row.subRows;

      if (isMatchedRowWithoutSubRows) {
        return {
          ...row,
          subRows: subRowsToInsert
        }
      }

      return row;
    });
  }

  return existingRows.map((row, index) => {
    const isMatchedRowWithSubRows = index === Number(id) && row.subRows;

    if (isMatchedRowWithSubRows) {
      const [, ...updatedPath] = path;

      return {
        ...row,
        subRows: insertIntoTable({
          existingRows: row.subRows,
          subRowsToInsert,
          path: updatedPath
        })
      }
    }

    return row;
  });
}
```

Here we take:
- `tableData` -> `existingRows` which is our existing table data
- `childData` -> `subRowsToInsert` which represents the new rows we want to insert
- `id` -> `path` which is formatted as `x.y.z` and split into `[x, y, z]` where `x`, `y` and `z` can be indices representing specific rows in a collection and the entire `id` effectively serves as a `path` to a specific `node` (row),

We use these values and some basic knowledge of recursive algorithms to first grab our current path for traversal:

```javascript
const id = path[0];
```

Then we formalize our base case (which in our case is when there just one path index left), decide if the node that we have reached has sub-rows or not, and then insert the row:

```javascript
const isBaseCase = path.length === 1;

if (isBaseCase) {
  return existingRows.map((row, index) => {
    const isMatchedRowWithoutSubRows = index === Number(id) && !row.subRows;

    if (isMatchedRowWithoutSubRows) {
      return {
        ...row,
        subRows: subRowsToInsert
      }
    }

    return row;
  });
}
```

If we are not in our base case, we recursively use the function `insertIntoTable` and feed in a subset version of our current rows:

```javascript
return existingRows.map((row, index) => {
  const isMatchedRowWithSubRows = index === Number(id) && row.subRows;

  if (isMatchedRowWithSubRows) {
    const [, ...updatedPath] = path;

    return {
      ...row,
      subRows: insertIntoTable({
        existingRows: row.subRows,
        subRowsToInsert,
        path: updatedPath
      })
    }
  }

  return row;
});
```

This may be a little complicated to get, especially if you haven't written much recursive code in a while, so feel free to take your time to trace through and understand it.

The purpose of this article isn't to focus on recursion but instead to focus on using React Table to pull off the lazy-loading expandable rows. This helper however is still a very important part of it.

### Building On Our React Query + React Table Proposal

In my [previous post](https://nafeu.medium.com/using-react-query-with-react-table-884158535424) I proposed an outline for combining React Query and React Table, this code also exists in our example code at [ReactQueryWithTable.jsx](https://github.com/nafeu/react-query-table-sandbox/blob/main/src/ReactQueryWithTable.jsx)

The proposal consists of structuring a query layer, data processing layer and rendering layer between the three components as follows:

```txt
TableQuery
TableInstance
TableLayout
```

Let's build on top or _expand_ on this example, refer now to [ReactTableExpanding](https://github.com/nafeu/react-query-table-sandbox/blob/main/src/ReactTableExpanding.jsx)

#### Updating our TableQuery Component

Let's take a look at our `TableQuery` component. We create a new variable called `isRowLoading` and implement a `handleClickRow` handler function which will be called anytime an individual row is clicked on our table.

It should be noted that `isRowLoading` is a loading state variable. It is a mapping of `paths` to a boolean value, that helps us keep track of which row is currently in the process of fetching data:

```javascript
const [isRowLoading, setIsRowLoading] = useState({});

const handleClickRow = async ({ id }) => {
  setIsRowLoading({ [id]: true });

  const { data: childData } = await fetchChildData();

  setIsRowLoading({ [id]: false })

  if (tableData) {
    const updatedTableData = recursivelyUpdateTable({ tableData, childData, id });

    setTableData(updatedTableData);
  }
}
```

We also use `recursivelyUpdateTable` to modify our existing table data with the child data that has been received from our API, this is why our recursive function is so important. One of the beauties of React Table is that it already knows how to work with embedded structures, all you need to do is provide them using a `subRows` property inside an individual row and it takes care of the rest.

We also have this invocation of `useQuery` that we modify:

```javascript
const {
  data: apiResponse,
  isLoading
} = useQuery('discussionGroups', fetchParentData, { enabled: !tableData });
```

We add `enabled: !tableData` to prevent the table form re-fetching in the background as it can mess up the embedded structure. Implementing re-fetching with an embedded structure is a more advanced topic that won't be covered in this article.

We also pass some more props into the `TableInstance` like so:

```javascript
return (
  <TableInstance
    tableData={tableData}
    onClickRow={handleClickRow}
    isRowLoading={isRowLoading}
  />
);
```

#### Updating our TableInstance Component

Aside from utilizing two new props `onClickRow` and `isRowLoading`, we will mainly focus on a new column header that we add:

```javascript
{
  Header: () => null,
  id: 'expander',
  Cell: ({ row, isLoading, isExpanded }) => {
    const toggleRowExpandedProps = row.getToggleRowExpandedProps();

    const onClick = async event => {
      if (!isLoading) {
        if (!isExpanded) {
          await onClickRow(row);
        }
        toggleRowExpandedProps.onClick(event);
      }
    }

    if (isLoading) {
      return <span>🔄</span>
    }

    return (
      <span
        {...row.getToggleRowExpandedProps({
          style: {
            paddingLeft: `${row.depth}rem`,
          },
        })}
        onClick={onClick}
      >
        {row.isExpanded ? '🔽' : '▶️'}
      </span>
    )
  },
},
```

Here is where some of the magic happens. We use the `Cell` property to instruct React Table on how it should format an individual cell, in this instance, we add an additional column in front of our table which will contain the expander button and we also add some additional logic to trigger our custom `onClickRow` handler if the row is in the appropriate loading and expanding state.

You may be wondering, where did `{ row, isLoading, isExpanding }` come from? The `row` object is actually something that React Table provides for us to access when using the `Cell` property, this is super helpful for us as `row` also has a useful method called `getToggleRowExpandedProps()` which we can use to manually trigger React Table's row expansion functionality.

We also add the `useExpanded` hook in our table instance instantiation with the option `autoResetExpanded` set to false:

```javascript
const tableInstance = useTable(
  { columns, data, autoResetExpanded: false },
  useExpanded
);
```

And we pass the `isRowLoading` object into the `TableLayout`:

```javascript
return (
  <TableLayout
    {...tableInstance}
    isRowLoading={isRowLoading}
  />
);
```

#### Updating our TableLayout Component

In our `TableLayout` component, we extract an extra prop `state.expanded` out of our `tableInstance`. This is provided to us by React Table thanks to the `useExpanded` hook, it provides a mapping similar to how we defined our `isRowLoading` mapping.

```javascript
const TableLayout = ({
  getTableProps,
  getTableBodyProps,
  headerGroups,
  rows,
  prepareRow,
  isRowLoading,
  state: { expanded }
}) => {
  ...
}
```

The main update is in our individual row rendering:

```javascript
return (
  <tr {...row.getRowProps()}>
    {row.cells.map(cell => {
      return <td {...cell.getCellProps()}>
        {cell.render('Cell', {
          isLoading: isRowLoading[row.id],
          isExpanded: expanded[row.id]
        })}
      </td>
    })}
  </tr>
)
```

In the `cell.render` method, we add an extra object that is filled with:

```javascript
{
  isLoading: isRowLoading[row.id],
  isExpanded: expanded[row.id]
}
```

This works in tandem with our previously declared code in the `TableInstance` component where we are able to feed custom values directly into a `Cell`. We defined `isRowLoading` -> `isLoading` ourselves and are feeding `expanded` -> `isExpanded` to enforce loading state and expanded state. Doing so isn't even that hard to read, that is what makes React Table so incredible to work with.

Once you've got all of this put together, you've got yourself this fancy table:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1614625868/aoflp1y1x5a0gmc83pit.gif)

Happy Coding!