
# mlr3db

<!-- badges: start -->

[![r-cmd-check](https://github.com/mlr-org/mlr3db/actions/workflows/r-cmd-check.yml/badge.svg)](https://github.com/mlr-org/mlr3db/actions/workflows/r-cmd-check.yml)
[![CRAN
Status](https://www.r-pkg.org/badges/version-ago/mlr3db)](https://cran.r-project.org/package=mlr3db)
[![Mattermost](https://img.shields.io/badge/chat-mattermost-orange.svg)](https://lmmisld-lmu-stats-slds.srv.mwn.de/mlr_invite/)
<!-- badges: end -->

Package website: [release](https://mlr3db.mlr-org.com/) \|
[dev](https://mlr3db.mlr-org.com/dev/)

Extends the [mlr3](https://mlr3.mlr-org.com/) package with a DataBackend
to transparently work with databases. Three additional backends are
currently implemented:

- `DataBackendDplyr`: Relies internally on the abstraction of
  [dplyr](https://dplyr.tidyverse.org/) and
  [dbplyr](https://dbplyr.tidyverse.org/). This allows working on a
  broad range of DBMS, such as SQLite, MySQL, MariaDB, or PostgreSQL.
- `DataBackendDuckDB`: Connector to
  [duckdb](https://cran.r-project.org/package=duckdb). This includes
  support for Parquet files (see example below).
- `DataBackendPolars`: Connector to
  [polars](https://pola-rs.github.io/r-polars/).

To construct the backends, you have to establish a connection to the
DBMS yourself with the [DBI](https://cran.r-project.org/package=DBI)
package. For the serverless SQLite and DuckDB, we provide the converters
`as_sqlite_backend()` and `as_duckdb_backend()`.

## Installation

You can install the released version of mlr3db from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("mlr3db")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("mlr-org/mlr3db")
```

## Example

### DataBackendDplyr

``` r
library("mlr3db")
#> Loading required package: mlr3

# Create a classification task:
task = tsk("spam")

# Convert the task backend from a in-memory backend (DataBackendDataTable)
# to an out-of-memory SQLite backend via DataBackendDplyr.
# A temporary directory is used here to store the database files.
task$backend = as_sqlite_backend(task$backend, path = tempfile())

# Resample a classification tree using a 3-fold CV.
# The requested data will be queried and fetched from the database in the background.
resample(task, lrn("classif.rpart"), rsmp("cv", folds = 3))
#> Warning in warn_deprecated("DataBackend$data_formats"):
#> DataBackend$data_formats is deprecated and will be removed in the future.
#> 
#> ── <ResampleResult> with 3 resampling iterations ───────────────────────────────
#>  task_id    learner_id resampling_id iteration     prediction_test warnings
#>     spam classif.rpart            cv         1 <PredictionClassif>        0
#>     spam classif.rpart            cv         2 <PredictionClassif>        0
#>     spam classif.rpart            cv         3 <PredictionClassif>        0
#>  errors
#>       0
#>       0
#>       0
```

### DataBackendDuckDB

``` r
library("mlr3db")

# Get an example parquet file from the package install directory:
# spam dataset (tsk("spam")) stored as parquet file
file = system.file(file.path("extdata", "spam.parquet"), package = "mlr3db")

# Create a backend on the file
backend = as_duckdb_backend(file)

# Construct classification task on the constructed backend
task = as_task_classif(backend, target = "type")

# Resample a classification tree using a 3-fold CV.
# The requested data will be queried and fetched from the database in the background.
resample(task, lrn("classif.rpart"), rsmp("cv", folds = 3))
#> 
#> ── <ResampleResult> with 3 resampling iterations ───────────────────────────────
#>  task_id    learner_id resampling_id iteration     prediction_test warnings
#>  backend classif.rpart            cv         1 <PredictionClassif>        0
#>  backend classif.rpart            cv         2 <PredictionClassif>        0
#>  backend classif.rpart            cv         3 <PredictionClassif>        0
#>  errors
#>       0
#>       0
#>       0
```
