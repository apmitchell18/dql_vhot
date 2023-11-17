## Hands-On Exercise 4 - Additional Exercises

In this section, you will find additional exercises to practice and test your abilities querying logs.

### Exercise 1 - Logs

All log records will have a status field. This is commonly "INFO", "WARN", "ERROR", etc.. Build a DQL query that calculates the percentage of ERROR records vs the total number of records.

HINT: No filters are required, use the countIf() and toDouble() functions. Be sure to create variables to refence. Use FieldsAdd to produce the percentage.

<H4><details>
<summary>Solution</summary>

```
fetch logs
| summarize total = count(), errorTotal = countif(status == "ERROR")
| fieldsAdd errorPercent = toDouble(errorTotal) / toDouble(total) * 100
```

</details></H4>
<br>
<br>

### Exercise 2 - Entities

In the [HOT session environment](https://zcy16892.apps.dynatrace.com/), pick one host and fetch a list of all the process groups running on that host that contain "easytravel" in their name. There may be more than one possible solution.

HINT: Try to solve this exercise using the **Expand** command mentioned at the end of the previous entity exercises.

<H4><details>
<summary>Possible Solution</summary>

```
fetch dt.entity.host
| filter entity.name == "ip-10-0-0-219.ec2.internal"
| fieldsAdd runs[dt.entity.process_group], alias:process_group_id
| expand process_group_id
| lookup sourceField:process_group_id, lookupField:id, [fetch dt.entity.process_group]
| fieldsRemove lookup.id
| filter contains(lookup.entity.name, "easytravel")
```

</details></H4>
<br>
<br>


### Exercise 3 - Timeseries

Starting with the following query:

```
timeseries {
usage = avg(dt.host.cpu.usage),
system = avg(dt.host.cpu.system)
},
by:{dt.entity.host}
```

Add the ability to show the correct host names instead of the DT entity IDs shown by default. Use the lookup and fieldsRemove commands.

<H4><details>
<summary>Possible Solution</summary>

```
timeseries {
usage = avg(dt.host.cpu.usage),
system = avg(dt.host.cpu.system)
},
by:{dt.entity.host}
| lookup [fetch dt.entity.host], sourceField:dt.entity.host, lookupField:id
| fieldsRemove dt.entity.host, lookup.id
```

</details></H4>
<br>
<br>
