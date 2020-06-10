
## Query Execution Flow
Following is the algorithm for executing a user query:

 1. User inputs the query of the form
 
 ```
 bin/run-query --num-objs 2 --pool tpchdata --oid-prefix "public" --table-name "TABLE_NAME" --select "col1,op1,val1;...;col2,op2,val2;" --project "col1,col2,...,coln" --groupby "col1,col2,...,coln" --orderby "col1,asc/desc;...;col2,asc/desc;"
 ```
 
 2. Query predicates are processed and classified as `AggPreds` (MIN, MAX, SUM, COUNT) and `NonAggPreds`  (others). These are stored differently.
 
 
 3. Firstly, all rows are processed through `NonAggPreds`and the passing rows are stored for further processing.

 4. If there is a `GROUP BY` query, `split-apply-combine` method of implementing GROUP BY is used. (Read more about the method [here](https://pandas.pydata.org/pandas-docs/stable/user_guide/groupby.html)).
 
	 4.1. **[STEP-1] Split:** Each row is mapped to a key based on the values of the GROUP BY columns. 
This step essentially *groups* the data based on some criteria.

	4.2.  **[STEP-2] Apply:** `AggPreds` are applied independently to each *group of rows* to obtain one consolidated result for each row group.

	4.3. **[STEP-3] Combine:** The processed rows are hence combined and packed into a data structure (in case of Skyhook, `bufferlist`) to return to the caller.
	
	4.4. Before packing into the final data structure, check if `ORDER BY` is present.  If yes, sort the returning rows `ASC/DESC` as per the query before packing.

5. If `GROUP BY` is not present, we skip Step 4.1 and continue from Step 4.2 by treating individual rows as groups.

## TL, DR:
Following flowchart summarises the above algorithm:

![Query Execution Flow](https://github.com/aditigupta17/skyhookdm-notes/blob/master/Query-Execution-Flow/exec_query_flowchart.png "Query Execution Flow")
