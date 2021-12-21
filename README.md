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
## Get Unique Values
There are multiple ways of getting unique values in JMP with varying speed. The most simple one is using Associative Array, followed by Summarize, Tabulate, Distribution and Summary (you can also use Design()). If you only want to count the amount of unique values, Distribution might be fastest. 

Same techniques can be also used with lists, you just first have to create datatable with the values (except with Associative Array()).


Getting unique values from Big Class name column:
```javascript
Names Default To Here(1);
dt = Open("$SAMPLE_DATA/Big Class.jmp");

## Making Private tables visible
You can make Private datatables "visible" if you have access to datatables reference with <<new data view. 
```javascript
Names Default To Here(1);
dt = Open("$SAMPLE_DATA/Big Class.jmp", private);
dt << New Data View;
```