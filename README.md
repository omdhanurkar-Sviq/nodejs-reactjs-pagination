# Pagination using Nodejs-MSSQL-Sequelize-Readjs-Mui-DataGrid

## Nodejs code

Refer this Sample code


```bash

// get page and pageSize from req.params

 const getAllProcessData = async (page, pageSize) => {
  const offset = (page - 1) * pageSize;

  try {
    const results = await sequelize.query(
      `
      SELECT *
      FROM CAUFV  // replace the Tablename by yours
      ORDER BY AUFNR  // replace the column name by yours
      OFFSET :offset ROWS
      FETCH NEXT :pageSize ROWS ONLY
      `,
      {
        type: Sequelize.QueryTypes.SELECT,
        replacements: { offset, pageSize },
      }
    );

    // Add a query to get the total count
    const totalCountQuery = await sequelize.query(
      `
      SELECT COUNT(*) as totalCount
      FROM CAUFV   // replace the Tablename by yours
      `,
      {
        type: Sequelize.QueryTypes.SELECT,
      }
    );

    const totalCount = totalCountQuery[0].totalCount;

    return { data: results, totalCount };
  } catch (error) {
    throw new Error(`Error in get process data service: ${error.message}`);
  }
};
```
### Reactjs code

```bash
import React, { useEffect, useState } from "react";
import axios from "axios";
import { Grid, Pagination } from "@mui/material";
import { DataGrid, GridToolbar } from "@mui/x-data-grid";
import Loader from "../../components/Loading/loader";

const PAGE_SIZE = 100;

const ProcessData = () => {
  const [loading, setLoading] = useState(true);
  const [processOrderData, setProcessOrderData] = useState([]);
  const [processOrdersTotalCount, setProcessOrdersTotalCount] = useState(0);
  const [currentPage, setCurrentPage] = useState(1);

  const fetchData = async (page, pageSize) => {
    try {
      const response = await axios.get("your-api-endpoint", {
        page,
        pageSize,
      });

      // Assuming response structure has data and totalCount properties
      setProcessOrderData(response.data.data);
      setProcessOrdersTotalCount(response.data.totalCount);
      setLoading(false);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  };

  const handlePageChange = (event, value) => {
    setCurrentPage(value);
    fetchData(value, PAGE_SIZE);
  };

  useEffect(() => {
    fetchData(1, PAGE_SIZE);
  }, []);

  //columns for Data-Grid
  const columns =
    processOrderData?.length && processOrderData?.length > 0
      ? [
          {
            field: "index",
            headerName: "#",
            width: 80,
            renderCell: (params) => (
              <span>
                {(currentPage - 1) * PAGE_SIZE + params.row.index + 1}
              </span>
            ),
          },
          ...Object.keys(processOrderData[0])
            .filter((key) => key !== "index")
            .map((key) => ({
              field: key,
              headerName: key,
              width: 150,
            })),
        ]
      : [];

  return (
    <div>
      {loading ? (
        <Loader />
      ) : (
        <>
          <Grid item xs={12} sm={6} md={8} sx={{ padding: "20px" }}>
            {processOrderData && processOrderData.length ? (
              <DataGrid
                rows={processOrderData.map((row, index) => ({
                  ...row,
                  index,
                }))}
                columns={columns}
                getRowId={(row) => row.index}
                pageSize={PAGE_SIZE}
                components={{
                  Toolbar: () => (
                    <GridToolbar
                      components={{
                        Toolbar: () => (
                          <div style={{ display: "flex" }}>
                            <GridToolbar />
                          </div>
                        ),
                      }}
                    />
                  ),
                  Pagination: () => (
                    <Grid
                      container
                      justifyContent="flex-end"
                      sx={{ padding: "20px" }}
                    >
                      <Pagination
                        count={Math.ceil(processOrdersTotalCount / PAGE_SIZE)}
                        page={currentPage}
                        onChange={handlePageChange}
                        size="small"
                        showFirstButton
                        showLastButton
                      />
                    </Grid>
                  ),
                }}
              />
            ) : (
              <h1>Data Not Found</h1>
            )}
          </Grid>
        </>
      )}
    </div>
  );
};

export default ProcessData;


```
