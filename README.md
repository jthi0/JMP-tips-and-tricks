# JMP-tips-and-tricks
JMP tips and tricks collection


## Enable filter for Filter Col Selector always
Filter col selectors filter is hidden in IfBox. When there are enough lines to show it will come visible, but you can force set the IfBox to one whenever you want to by using XPath
```javascript
Names Default To Here(1);
New Window("Filter Col Selector",
    flc = Filter Col Selector(),
    ((flc << Parent) << Xpath("//IfBox")) << set(1)
)
```

## Use Capture Log to capture problems with database queries
When querying for data JMP doesn't necessarily throw an error when problem occurs and it might be difficult to detect. You can check if result is a datatable and if it has any rows, but that might not catch all errors and you won't get errormessages. So you can try using Capture Log()
```javascript
Names Default To Here(1);
dbc = Create Database Connection("DSN=dBASE Files;DBQ=C:/Program Files/SAS/JMPPRO/16/Samples/Import Data/;");
ora_error = Log Capture(dt = Execute SQL(databaseConnectionHandle, "SELECT HEIGHT, WEIGHT FROM Bigclass", "NewTable"));
If(Contains(ora_error, "ORA"),
	Throw(ora_error);
);
Close Database Connection(databaseConnectionHandle);
```


## Get Unique Values
There are multiple ways of getting unique values in JMP with varying speed. The most simple one is using Associative Array, followed by Summarize and Summary (you can also use Design(), Tabulate, Distribution...).

Same techniques can be also used with lists, you just first have to create datatable with the values (except with Associative Array()).

Getting unique values from Big Class name column:
```javascript
Names Default To Here(1);
dt = Open("$SAMPLE_DATA/Big Class.jmp");

//Associative Array
//Three examples how to get values to Associative Array. << get keys to get unique names
unique_names = Associative Array(Column(dt, "name") << get keys);
unique_names = Associative Array(dt[0, "name"]) << get keys;
unique_names = Associative Array(dt:name) << get keys;

//Summarize
//Will save value to unique_names variable
Summarize(dt, unique_names = by(:name));

//Summary() table
dt_summary = dt << Summary(
	Group(:name),Freq("None"),Weight("None"),
	Link to original data table(0),private
);
unique_names = dt_summary[0,1]; //[0,2] contains count of unique values in N Rows column
Close(dt_summary, no save);

//create table from list
list = {"a", "b", "a", "c"};
dt = New table("getuniques",
	New column("1", character, values(list)),
	private
);
```

## Re-order list using Ranking
You can re-order your values in associative array back to your original order by using Ranking. This also allows you ranking for example columns from best to worst based on their Gage R&R results
reorder list using Ranking.

I have used this technique to keep values saved in Associative Array for easier access and then when I want to report them in original order, I can do it with Ranking. Similarly I have calculated some specific values over multiple tables and wanted to sort datatables based on that. That can also be done with same technique.
```javascript
Names Default To Here(1);

//re-ordering associative array
order_list = {"ThisFirst", "Second", "AA_Third", "Fourth"};
values = {"first", "second", "third", "fourth"};
aa = Associative Array(Eval List(order_list), Eval List(values));
aa["ThisFirst"] = "AAA"; aa["Second"] = "BBB";
//in alphabetical order
keys = aa << get keys; 
values = aa << get values;

orig_order_order_list = keys[Ranking(order_list)];
orig_order_values = values[Ranking(order_list)];
```


## Making Private tables visible
You can make Private datatables "visible" if you have access to datatables reference with <<new data view. 
```javascript
Names Default To Here(1);
dt = Open("$SAMPLE_DATA/Big Class.jmp", private);
dt << New Data View;
```

## << Set Each Value instead of Formula()
Usually when you want to calculate something in JMP datatable you use formula() and then delete the formula, if you don't want the values to change. You can skip the delete part and make creation of the column faster by using << Set Each Value

```javascript
Names Default To Here(1);
dt = Open("$SAMPLE_DATA/Big Class.jmp");
new_col = dt << New Column("Formula", Numeric, Continuous, Formula(ColSum(:height, :age)));
new_col << Delete Formula;
new_col = dt << New Column("SetEachValue", Numeric, Continuous, <<Set Each Value(ColSum(:height, :age)));
