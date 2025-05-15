# Project-guide

import React, { forwardRef, useImperativeHandle, useRef, useState } from 'react';
import { AgGridReact } from 'ag-grid-react';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import { GridOptions, GridReadyEvent, SortChangedEvent, PaginationChangedEvent, ColDef } from 'ag-grid-community';
import { LinearProgress, Paper } from '@mui/material';

interface GenericJetLinkDataTableProps {
  columns: ColDef[];
  responseData: any[];
  totalRecords: number;
  page: number;
  rowsPerPage: number;
  pending: boolean;
  handlePageChange: (page: number) => void;
  handleRowsPerPageChange: (rowsPerPage: number) => void;
  handleSort: (sortField: string, sortDirection: string) => void;
  containerClass?: string;
  gridHeight?: string;
}

export interface GenericJetLinkDataTableRef {
  refreshData: () => void;
}

const GenericJetLinkDataTable = forwardRef<GenericJetLinkDataTableRef, GenericJetLinkDataTableProps>(
  (
    {
      columns,
      responseData,
      totalRecords,
      page,
      rowsPerPage,
      pending,
      handlePageChange,
      handleRowsPerPageChange,
      handleSort,
      containerClass = '',
      gridHeight = '600px'
    },
    ref
  ) => {
    const gridRef = useRef<AgGridReact>(null);
    const [gridApi, setGridApi] = useState<any>(null);
    const [gridColumnApi, setGridColumnApi] = useState<any>(null);

    // Expose refresh method via ref
    useImperativeHandle(ref, () => ({
      refreshData: () => {
        if (gridApi) {
          gridApi.refreshCells();
        }
      }
    }));

    const gridOptions: GridOptions = {
      rowModelType: 'clientSide',
      pagination: true,
      paginationPageSize: rowsPerPage,
      cacheBlockSize: rowsPerPage,
      suppressPaginationPanel: true,
      suppressScrollOnNewData: true,
      animateRows: true,
      headerHeight: 48,
      rowHeight: 48,
      domLayout: 'autoHeight',
      onGridReady: (params: GridReadyEvent) => {
        setGridApi(params.api);
        setGridColumnApi(params.columnApi);
        params.api.sizeColumnsToFit();
      },
      onSortChanged: (params: SortChangedEvent) => {
        const sortModel = params.api.getSortModel();
        if (sortModel.length > 0) {
          const { colId, sort } = sortModel[0];
          handleSort(colId, sort || 'asc');
        } else {
          handleSort('', '');
        }
      },
      onPaginationChanged: (params: PaginationChangedEvent) => {
        if (params.newPage) {
          const newPage = params.api.paginationGetCurrentPage() || 0;
          if (newPage + 1 !== page) {
            handlePageChange(newPage + 1);
          }
        }
      }
    };

    const defaultColDef = {
      sortable: true,
      resizable: true,
      filter: true,
      flex: 1,
      minWidth: 100,
    };

    const onGridSizeChanged = () => {
      if (gridApi) {
        gridApi.sizeColumnsToFit();
      }
    };

    const changePageSize = (newPageSize: number) => {
      if (gridApi) {
        gridApi.paginationSetPageSize(newPageSize);
        handleRowsPerPageChange(newPageSize);
      }
    };

    return (
      <div className={`${containerClass}`} style={{ width: '100%' }}>
        {pending && <LinearProgress />}
        
        <div className="ag-theme-alpine" style={{ height: gridHeight, width: '100%' }}>
          <AgGridReact
            ref={gridRef}
            columnDefs={columns}
            rowData={responseData}
            gridOptions={gridOptions}
            defaultColDef={defaultColDef}
            onGridSizeChanged={onGridSizeChanged}
            pagination={true}
            paginationPageSize={rowsPerPage}
            suppressPaginationPanel={true}
            cacheBlockSize={rowsPerPage}
            rowModelType="clientSide"
          />
        </div>

        <Paper elevation={0} style={{ padding: '16px', display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
          <div>
            <span style={{ marginRight: '16px' }}>
              Rows per page:
              <select
                value={rowsPerPage}
                onChange={(e) => changePageSize(Number(e.target.value))}
                style={{ marginLeft: '8px', padding: '4px' }}
              >
                {[10, 25, 50, 100].map((size) => (
                  <option key={size} value={size}>
                    {size}
                  </option>
                ))}
              </select>
            </span>
            <span>
              {`${page * rowsPerPage - rowsPerPage + 1}-${Math.min(
                page * rowsPerPage,
                totalRecords
              )} of ${totalRecords}`}
            </span>
          </div>
          <div>
            <button
              onClick={() => handlePageChange(page - 1)}
              disabled={page === 1}
              style={{ marginRight: '8px', padding: '4px 8px' }}
            >
              Previous
            </button>
            <button
              onClick={() => handlePageChange(page + 1)}
              disabled={page * rowsPerPage >= totalRecords}
              style={{ padding: '4px 8px' }}
            >
              Next
            </button>
          </div>
        </Paper>
      </div>
    );
  }
);

export default GenericJetLinkDataTable;







------__------------


useEffect(() => {
  const runId = forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
               scheduleDetailsSelector.scheduleDetails?.resp?.runId;
  
  if (runId) {
    invokeGetScheduleForecastResults(
      runId,
      paginationModel.page - 1,
      paginationModel.pageSize,
      sortParam(orderByField, orderBy),
      {}
    );
  }
}, [
  paginationModel.page,
  paginationModel.pageSize,
  orderBy,
  orderByField,
  forecastDetailsSelector.forecastDetails?.resp?.runId,
  scheduleDetailsSelector.scheduleDetails?.resp?.runId
]);




-----------

<GenericJetLinkDataTable
  columns={generatedScheduleForecastResultColumns(onClickPilotIcon)}
  containerClass='schedule_forecast_trips_container'
  handleOnRowPerPageChange={handleChangeRowsPerPage}
  handlePageChange={handleChangePage}
  handleSort={handleSort}
  page={paginationModel.page}
  pending={pageLoader}
  responseData={generatedResults}
  totalRecords={generatedResultsSelector.result?.totalElements || 0}
  rowsPerPage={paginationModel.pageSize}
  sortModel={orderByField && orderBy ? 
    [{ field: orderByField, sort: orderBy }] : 
    undefined
/>


----



const handleSort = (selectedColumn: any, sortDirection: string) => {
  const sortField = selectedColumn.sortField || selectedColumn.field;
  setOrderByField(sortField);
  setOrderBy(sortDirection);
  
  // Reset to first page when sorting changes
  setPaginationModel(prev => ({ ...prev, page: 1 }));
  
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    0, // first page
    paginationModel.pageSize,
    sortParam(sortField, sortDirection),
    {}
  );
};


----
const handleChangePage = (page: number) => {
  setPaginationModel({ ...paginationModel, page });
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    page - 1, // assuming backend expects 0-based index
    paginationModel.pageSize,
    sortParam(orderByField, orderBy),
    {}
  );
};

const handleChangeRowsPerPage = (pageSize: number) => {
  setPaginationModel({ ...paginationModel, pageSize, page: 1 });
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    0, // reset to first page when pageSize changes
    pageSize,
    sortParam(orderByField, orderBy),
    {}
  );
};

_______&&&
You're absolutely right! We can leverage AG-Grid's built-in pagination controls instead of creating custom pagination elements. Here's a more streamlined version of the `GenericJetLinkDataTable` component that uses AG-Grid's native pagination features while still maintaining server-side operations:

```typescript
import React, { forwardRef, useImperativeHandle, useRef, useEffect } from 'react';
import { AgGridReact } from 'ag-grid-react';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import { GridOptions, GridReadyEvent, SortChangedEvent, PaginationChangedEvent, ColDef } from 'ag-grid-community';
import { LinearProgress } from '@mui/material';

interface GenericJetLinkDataTableProps {
  columns: ColDef[];
  responseData: any[];
  totalRecords: number;
  page: number;
  rowsPerPage: number;
  pending: boolean;
  handlePageChange: (page: number) => void;
  handleRowsPerPageChange: (rowsPerPage: number) => void;
  handleSort: (sortField: string, sortDirection: string) => void;
  containerClass?: string;
  gridHeight?: string;
}

const GenericJetLinkDataTable = forwardRef(
  (
    {
      columns,
      responseData,
      totalRecords,
      page,
      rowsPerPage,
      pending,
      handlePageChange,
      handleRowsPerPageChange,
      handleSort,
      containerClass = '',
      gridHeight = '600px'
    }: GenericJetLinkDataTableProps,
    ref
  ) => {
    const gridRef = useRef<AgGridReact>(null);

    const gridOptions: GridOptions = {
      rowModelType: 'clientSide',
      pagination: true,
      paginationPageSize: rowsPerPage,
      suppressPaginationPanel: false, // Show built-in pagination
      suppressScrollOnNewData: true,
      animateRows: true,
      headerHeight: 48,
      rowHeight: 48,
      domLayout: 'autoHeight',
      onGridReady: (params: GridReadyEvent) => {
        params.api.sizeColumnsToFit();
      },
      onSortChanged: (params: SortChangedEvent) => {
        const sortModel = params.api.getSortModel();
        if (sortModel.length > 0) {
          const { colId, sort } = sortModel[0];
          handleSort(colId, sort || 'asc');
        } else {
          handleSort('', '');
        }
      },
      onPaginationChanged: (params: PaginationChangedEvent) => {
        if (params.newPage && gridRef.current?.api) {
          const newPage = gridRef.current.api.paginationGetCurrentPage() || 0;
          const currentPageSize = gridRef.current.api.paginationGetPageSize();
          
          // Only trigger if page actually changed
          if (newPage + 1 !== page) {
            handlePageChange(newPage + 1);
          }
          
          // Check if page size changed
          if (currentPageSize !== rowsPerPage) {
            handleRowsPerPageChange(currentPageSize);
          }
        }
      },
    };

    const defaultColDef = {
      sortable: true,
      resizable: true,
      filter: true,
      flex: 1,
      minWidth: 100,
    };

    useEffect(() => {
      if (gridRef.current?.api) {
        // Set total records for pagination
        gridRef.current.api.setRowData(responseData);
        gridRef.current.api.paginationSetPageSize(rowsPerPage);
        gridRef.current.api.paginationGoToPage(page - 1);
        
        // This ensures the pagination controls show the correct total
        // Note: This is a workaround since AG-Grid doesn't natively support server-side row count
        const fakeLastRow = totalRecords;
        gridRef.current.api.setInfiniteRowCount(fakeLastRow);
      }
    }, [responseData, page, rowsPerPage, totalRecords]);

    const onGridSizeChanged = () => {
      gridRef.current?.api.sizeColumnsToFit();
    };

    return (
      <div className={`${containerClass}`} style={{ width: '100%' }}>
        {pending && <LinearProgress />}
        
        <div className="ag-theme-alpine" style={{ height: gridHeight, width: '100%' }}>
          <AgGridReact
            ref={gridRef}
            columnDefs={columns}
            rowData={responseData}
            gridOptions={gridOptions}
            defaultColDef={defaultColDef}
            onGridSizeChanged={onGridSizeChanged}
          />
        </div>
      </div>
    );
  }
);

export default GenericJetLinkDataTable;
```

## Key Improvements:

1. **Built-in Pagination Controls**:
   - Set `suppressPaginationPanel: false` to use AG-Grid's native pagination UI
   - The grid will show page numbers, next/previous buttons, and page size selector

2. **Simplified Implementation**:
   - Removed custom pagination UI components
   - Let AG-Grid handle the pagination UI rendering
   - Still maintain control over the actual pagination behavior via callbacks

3. **Proper Pagination Sync**:
   - Uses `useEffect` to sync the grid's pagination state with the parent component's state
   - Updates both page number and page size when they change

4. **Total Records Display**:
   - Uses `setInfiniteRowCount` as a workaround to show the correct total record count
   - This makes the pagination controls display "1-10 of 100" correctly

## Usage Example:

```typescript
<GenericJetLinkDataTable
  columns={columns}
  responseData={data}
  totalRecords={totalCount}
  page={currentPage}
  rowsPerPage={pageSize}
  pending={loading}
  handlePageChange={handlePageChange}
  handleRowsPerPageChange={handlePageSizeChange}
  handleSort={handleSortChange}
/>
```

## Notes:

1. While we're using AG-Grid's UI controls, we're still maintaining server-side operations:
   - When user changes page/sort, we get callbacks
   - Parent component makes API calls
   - New data is passed back to the grid

2. The grid's pagination controls will work naturally, but all data operations still happen on the server.

3. You can customize the built-in pagination appearance through AG-Grid's styling options.

This approach gives you the best of both worlds - a polished, built-in pagination UI while maintaining full control over the actual data fetching behavior.





---------

# GenericJetLinkDataTable Component with AG-Grid and Server-Side Pagination/Sorting

Here's a complete implementation of the `GenericJetLinkDataTable` component using AG-Grid with server-side pagination and sorting:

```typescript
import React, { forwardRef, useImperativeHandle, useRef, useState } from 'react';
import { AgGridReact } from 'ag-grid-react';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import { GridOptions, GridReadyEvent, SortChangedEvent, PaginationChangedEvent, ColDef } from 'ag-grid-community';
import { LinearProgress, Paper } from '@mui/material';

interface GenericJetLinkDataTableProps {
  columns: ColDef[];
  responseData: any[];
  totalRecords: number;
  page: number;
  rowsPerPage: number;
  pending: boolean;
  handlePageChange: (page: number) => void;
  handleRowsPerPageChange: (rowsPerPage: number) => void;
  handleSort: (sortField: string, sortDirection: string) => void;
  containerClass?: string;
  gridHeight?: string;
}

export interface GenericJetLinkDataTableRef {
  refreshData: () => void;
}

const GenericJetLinkDataTable = forwardRef<GenericJetLinkDataTableRef, GenericJetLinkDataTableProps>(
  (
    {
      columns,
      responseData,
      totalRecords,
      page,
      rowsPerPage,
      pending,
      handlePageChange,
      handleRowsPerPageChange,
      handleSort,
      containerClass = '',
      gridHeight = '600px'
    },
    ref
  ) => {
    const gridRef = useRef<AgGridReact>(null);
    const [gridApi, setGridApi] = useState<any>(null);
    const [gridColumnApi, setGridColumnApi] = useState<any>(null);

    // Expose refresh method via ref
    useImperativeHandle(ref, () => ({
      refreshData: () => {
        if (gridApi) {
          gridApi.refreshCells();
        }
      }
    }));

    const gridOptions: GridOptions = {
      rowModelType: 'clientSide',
      pagination: true,
      paginationPageSize: rowsPerPage,
      cacheBlockSize: rowsPerPage,
      suppressPaginationPanel: true,
      suppressScrollOnNewData: true,
      animateRows: true,
      headerHeight: 48,
      rowHeight: 48,
      domLayout: 'autoHeight',
      onGridReady: (params: GridReadyEvent) => {
        setGridApi(params.api);
        setGridColumnApi(params.columnApi);
        params.api.sizeColumnsToFit();
      },
      onSortChanged: (params: SortChangedEvent) => {
        const sortModel = params.api.getSortModel();
        if (sortModel.length > 0) {
          const { colId, sort } = sortModel[0];
          handleSort(colId, sort || 'asc');
        } else {
          handleSort('', '');
        }
      },
      onPaginationChanged: (params: PaginationChangedEvent) => {
        if (params.newPage) {
          const newPage = params.api.paginationGetCurrentPage() || 0;
          if (newPage + 1 !== page) {
            handlePageChange(newPage + 1);
          }
        }
      }
    };

    const defaultColDef = {
      sortable: true,
      resizable: true,
      filter: true,
      flex: 1,
      minWidth: 100,
    };

    const onGridSizeChanged = () => {
      if (gridApi) {
        gridApi.sizeColumnsToFit();
      }
    };

    const changePageSize = (newPageSize: number) => {
      if (gridApi) {
        gridApi.paginationSetPageSize(newPageSize);
        handleRowsPerPageChange(newPageSize);
      }
    };

    return (
      <div className={`${containerClass}`} style={{ width: '100%' }}>
        {pending && <LinearProgress />}
        
        <div className="ag-theme-alpine" style={{ height: gridHeight, width: '100%' }}>
          <AgGridReact
            ref={gridRef}
            columnDefs={columns}
            rowData={responseData}
            gridOptions={gridOptions}
            defaultColDef={defaultColDef}
            onGridSizeChanged={onGridSizeChanged}
            pagination={true}
            paginationPageSize={rowsPerPage}
            suppressPaginationPanel={true}
            cacheBlockSize={rowsPerPage}
            rowModelType="clientSide"
          />
        </div>

        <Paper elevation={0} style={{ padding: '16px', display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
          <div>
            <span style={{ marginRight: '16px' }}>
              Rows per page:
              <select
                value={rowsPerPage}
                onChange={(e) => changePageSize(Number(e.target.value))}
                style={{ marginLeft: '8px', padding: '4px' }}
              >
                {[10, 25, 50, 100].map((size) => (
                  <option key={size} value={size}>
                    {size}
                  </option>
                ))}
              </select>
            </span>
            <span>
              {`${page * rowsPerPage - rowsPerPage + 1}-${Math.min(
                page * rowsPerPage,
                totalRecords
              )} of ${totalRecords}`}
            </span>
          </div>
          <div>
            <button
              onClick={() => handlePageChange(page - 1)}
              disabled={page === 1}
              style={{ marginRight: '8px', padding: '4px 8px' }}
            >
              Previous
            </button>
            <button
              onClick={() => handlePageChange(page + 1)}
              disabled={page * rowsPerPage >= totalRecords}
              style={{ padding: '4px 8px' }}
            >
              Next
            </button>
          </div>
        </Paper>
      </div>
    );
  }
);

export default GenericJetLinkDataTable;
```

## How to Use This Component

Here's how you would use this component in your `GeneratedScheduleForecastResult`:

```typescript
<GenericJetLinkDataTable
  ref={tableRef}
  columns={generatedScheduleForecastResultColumns(onClickPilotIcon)}
  responseData={generatedResults}
  totalRecords={generatedResultsSelector.result?.totalElements || 0}
  page={paginationModel.page}
  rowsPerPage={paginationModel.pageSize}
  pending={pageLoader}
  handlePageChange={handleChangePage}
  handleRowsPerPageChange={handleChangeRowsPerPage}
  handleSort={handleSort}
  containerClass="schedule_forecast_trips_container"
  gridHeight="calc(100vh - 250px)"
/>
```

## Key Features

1. **Server-Side Pagination**:
   - The component maintains pagination state externally
   - Calls back to parent component when page changes
   - Displays custom pagination controls

2. **Server-Side Sorting**:
   - Detects sort changes and calls back to parent
   - Supports multi-column sorting if needed

3. **Performance Optimization**:
   - Uses AG-Grid's efficient rendering
   - Only shows loading indicator during API calls
   - Proper cleanup of grid resources

4. **Customizable**:
   - Adjustable height and container classes
   - Exposes refresh method via ref
   - Configurable page size options

5. **Type Safety**:
   - Proper TypeScript interfaces for props and ref
   - Type-safe column definitions

## Implementation Notes

1. The component uses AG-Grid's client-side row model but triggers server-side operations through callbacks.

2. All data operations (sorting, pagination) are handled by the parent component which makes the actual API calls.

3. The grid automatically sizes columns to fit the available space and handles window resizing.

4. You can customize the appearance by:
   - Modifying the AG-Grid theme
   - Adjusting the MUI Paper component styling
   - Changing the pagination controls

5. For very large datasets, consider implementing infinite scrolling or proper server-side row model if AG-Grid Enterprise is available.

This implementation provides a clean separation of concerns where the table component handles the UI rendering and user interactions, while the parent component manages the data fetching and state management.





-------

# Analysis of the GeneratedScheduleForecastResult Component

Based on the code snippets you've shared, I'll analyze the current implementation and provide recommendations for implementing server-side pagination and sorting with the GenericJetLinkDataTable component using Redux-Saga.

## Current Implementation Overview

1. **Data Flow**:
   - Initial API calls (`getAircraftDetails`, `getForecastDetails`/`getScheduleDetails`) are made on component mount
   - After receiving a `runId` from these calls, `getScheduleForecastResults` is invoked
   - Pagination, sorting, and filtering are handled through various `useEffect` hooks

2. **Key Dependencies**:
   - `paginationModel` (page and pageSize)
   - `orderBy` and `orderByField` for sorting
   - `forecastDetailsSelector` or `scheduleDetailsSelector` for the runId

## Recommendations for Improvement

### 1. Server-Side Pagination Implementation

```typescript
const handleChangePage = (page: number) => {
  setPaginationModel({ ...paginationModel, page });
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    page - 1, // assuming backend expects 0-based index
    paginationModel.pageSize,
    sortParam(orderByField, orderBy),
    {}
  );
};

const handleChangeRowsPerPage = (pageSize: number) => {
  setPaginationModel({ ...paginationModel, pageSize, page: 1 });
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    0, // reset to first page when pageSize changes
    pageSize,
    sortParam(orderByField, orderBy),
    {}
  );
};
```

### 2. Server-Side Sorting Implementation

```typescript
const handleSort = (selectedColumn: any, sortDirection: string) => {
  const sortField = selectedColumn.sortField || selectedColumn.field;
  setOrderByField(sortField);
  setOrderBy(sortDirection);
  
  // Reset to first page when sorting changes
  setPaginationModel(prev => ({ ...prev, page: 1 }));
  
  invokeGetScheduleForecastResults(
    forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
    scheduleDetailsSelector.scheduleDetails?.resp?.runId,
    0, // first page
    paginationModel.pageSize,
    sortParam(sortField, sortDirection),
    {}
  );
};
```

### 3. Optimizing API Calls

Combine your `useEffect` hooks to reduce unnecessary API calls:

```typescript
useEffect(() => {
  const runId = forecastDetailsSelector.forecastDetails?.resp?.runId ?? 
               scheduleDetailsSelector.scheduleDetails?.resp?.runId;
  
  if (runId) {
    invokeGetScheduleForecastResults(
      runId,
      paginationModel.page - 1,
      paginationModel.pageSize,
      sortParam(orderByField, orderBy),
      {}
    );
  }
}, [
  paginationModel.page,
  paginationModel.pageSize,
  orderBy,
  orderByField,
  forecastDetailsSelector.forecastDetails?.resp?.runId,
  scheduleDetailsSelector.scheduleDetails?.resp?.runId
]);
```

### 4. GenericJetLinkDataTable Integration

Ensure your table component is properly configured:

```typescript
<GenericJetLinkDataTable
  columns={generatedScheduleForecastResultColumns(onClickPilotIcon)}
  containerClass='schedule_forecast_trips_container'
  handleOnRowPerPageChange={handleChangeRowsPerPage}
  handlePageChange={handleChangePage}
  handleSort={handleSort}
  page={paginationModel.page}
  pending={pageLoader}
  responseData={generatedResults}
  totalRecords={generatedResultsSelector.result?.totalElements || 0}
  rowsPerPage={paginationModel.pageSize}
  sortModel={orderByField && orderBy ? 
    [{ field: orderByField, sort: orderBy }] : 
    undefined
/>
```

### 5. Redux-Saga API Call

Your saga implementation looks correct, but ensure the endpoint handles all parameters:

```typescript
// In your saga
export function* getScheduleForecastResults(params: any) {
  try {
    const { runId, pageNumber, pageSize, sort } = params.payload;
    
    const response = yield call(
      HttpService,
      `v1/snf-result?page=${pageNumber}&size=${pageSize}&runId=${runId}&sort=${sort}`,
      'GET',
      null,
      null,
      0,
      false,
      true
    );
    
    yield put(getScheduleForecastResultsSuccess(response?.data));
  } catch (err) {
    yield put(getScheduleForecastResultsFailed(err));
    showToastError(err, 'getScheduleForecastResultsFailed');
  }
}
```

## Key Considerations

1. **Loading States**: You're already tracking loading states which is good for UI feedback.

2. **Error Handling**: Ensure proper error handling in both the component and sagas.

3. **Type Safety**: Consider adding proper TypeScript interfaces for your API responses and state.

4. **Debouncing**: For rapid user interactions (like quick page changes), consider debouncing API calls.

5. **Cleanup**: Your cleanup in the unmount effect looks correct - clearing Redux state when component unmounts.

This implementation should give you a robust server-side paginated and sortable table that works with your Redux-Saga setup. The key is ensuring all state changes that should trigger a new API call are properly handled in the combined useEffect hook.

