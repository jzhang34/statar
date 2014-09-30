statar
======

A set of R commands for Stata users built on dplyr and data.table. 

1. The package adds the following vector functions: partition and lead/lag
	````R
	library(dplyr)
	library(data.table)
	library(statar)
	
	# partition creates quantile categories (corresponds to Stata xtile)
	v2 <-   sample(1e6, 1e6, TRUE)                   
	v2_categorized <- partition(v2, nq = 3) # 3 groups based on terciles
	v2_categorized <- partition(v2, cutpoints = c(1e5, 5e5)) # 3 groups based on two cutpoints
	
	# lag (corresponds to Stata L. F.)
	## unbalanced panel
	DT <- data.frame(
	 date  = c(1992, 1989, 1991, 1990, 1994, 1992, 1991),
	 value = c(4.1, 4.5, 3.3, 5.3, 3.0, 3.2, 5.2)
	)
	DT %>% mutate(lag(value, 1, order_by = date)) # wrong
	DT %>% mutate(lag(value, 1, along_with = date)) # right
	## lubridate periods can be used instead of integers
	library(lubridate)
    df <- data.frame(     
       id = c("id1", "id1", "id1", "id1"),
     date = mdy(c("03/01/1992", "04/03/1992", "07/15/1992", "08/21/1992")),
    value = c(4.1, 4.5, 3.3, 5.3)
    )
    df <- df %>% mutate(date = floor_date(date, "month"))
	df %>% group_by(id) %>% mutate(lag(value, months(1), along_with = date)) 
	````

2. The package adds the following verbs built on dplyr syntax for data.tables: `colorder`, `sum_up` and `expand`.

	````R
	
	N=1e6; K=100
    DT <- data.table(
	  id = 1:N,
	  v1 = sample(5, N, TRUE),
	  v2 = sample(1e6, N, TRUE),
	  v3 = sample(round(runif(100,max=100), 4), N, TRUE)
	  )
	
	# colorder (= Stata order)
	DT  %>% colorder(starts_with("v"))
	DT  %>% colorder(starts_with("v"), inplace = TRUE)
	
	# sum_up (= Stata summarize)
	DT  %>% sum_up
	DT  %>% sum_up(v3, d=T)
	DT  %>% filter(v1==1) %>% sum_up(starts_with("v"))
	
	# expand (= Stata tsfill)
	DT <- data.table(
	    id = c(1, 1, 1, 1, 1, 2, 2),
	  date = c(1992, 1989, 1991, 1990, 1994, 1992, 1991),
	 value = c(4.1, 4.5, 3.3, 5.3, 3.0, 3.2, 5.2)
	)
	DT %>% expand(date)
	DT %>% group_by(id) %>% expand(date, type = "within")
	DT %>% group_by(id) %>% expand(date, type = "across")
	````

3. The package adds a wrapper for `merge` on data.tables based on Stata syntax
	
	````R
	## left join
	ejoin(DTm, DTu, type = m:m, keep = c("master","matched"), gen = FALSE)
	## inner join
	ejoin(DTm, DTu, type = m:m, keep = "matched", gen = FALSE)
	## full outer join
	ejoin(DTm, DTu, type = m:m, keep = c("master","matched","using"), gen = FALSE)
	````

3. Other commands
	- A `tidyr::spread`  method for data.tables that relies on `dcast.data.table`. This makes the command faster.
	- `floor_date`, originally from the package `lubridate`, is rewritten to accept "quarter" as an argument 
	- `tempname` creates a name not assigned in the environment specified by the second variable

		````R
		tempvar <- tempname("temp", DT)
		tempname <- tempname("temp", globalenv())
		````

The package can be installed via the package `devtools`

````R
devtools::install_github("matthieugomez/tidyr")
devtools::install_github("matthieugomez/lazyeval")
devtools::install_github("matthieugomez/dplyr")
devtools::install_github("matthieugomez/statar")
````
