## Hands-On Exercise 2 - Log Queries in Practice

In this section, you will work through a practical use case using log queries in DQL. This exercise will focus on find log sources, appropriately filtering your queries for log sources, and evaluating conditions within logs.

---

### Step 1: Finding Logs - Log Sources

Let's find all sources in an environment. Create a Notebook and run a DQL Query.

**Run the following query and review the results:**
```
fetch logs
| summarize count(), by: {log.source}
```

How many unique logs sources do you see? Which logs have the most activity?

**Let's try to sort the logs from biggest to smallest.**
```
fetch logs
| summarize count(), by: {log.source}
| sort `count()` desc
```

**Can you make the query a little cleaner, avoiding the need to use backticks for count()?**

<H3><details>
    <summary>Click to Expand Solution</summary>

```
fetch logs
| summarize LogCount=count(), by: {log.source}
| sort LogCount desc
```

</H3></details>


### Step 2: Finding Logs - Log Sources

Now let's try to locate a log by a piece of text. For example, you may just be given an example of a log to start with, but not be given the log source. Think about why you can't just search for this exact log line.

**Write a query to find logs that look like this example line:**
```
2023-08-21 11:25:13 uemload    INFO  [HeadlessStatistics] Aug 21 11:20:00: Visits: started [2] Completed [2] Skipped [0] Exceptions [2] Average [2] started/minute. from:[Aug 21 11:24:00] to:[Aug 21 11:24:00]
```

Since most of these attributes look like they will vary, pick something that looks consistent.


<H3><details>
    <summary>Click to Expand Solution</summary>

```
fetch logs
| filter matchesPhrase(content, "HeadlessStatistics")
```

Now you should see an output of logs that look like the given example.

</H3></details>

But simply using this as a filter may not be specific enough. How can we make sure we're narrowing down the results sufficiently?

**Use what we learned from Step 1 to see what log sources matching this filter**

```
fetch logs
| filter matchesPhrase(content, "HeadlessStatistics")
| summarize count(), by: {dt.source_entity}
```

You'll see that lots of different processes have this style of log. 

Any log queries you write need to account for the right scope.

**Find the underlying process group for these logs**

```
fetch logs
| filter matchesPhrase(content, "HeadlessStatistics")
| summarize count(), by: {dt.process.name, dt.entity.process_group}
```

Now you can work with the requestor to narrow down the process.

**Write a query that additionally filters the log query by process group.**

<H3><details>
    <summary>Click to Expand Solution</summary>

Example Process Group - use a real Process Group from your environment. 
```
fetch logs
| filter matchesPhrase(content, "HeadlessStatistics") and in(dt.entity.process_group, "PROCESS_GROUP-A6BBB46BDAADE0F2")
```

You should see an output of logs filtered down by the desired process group.

</H3></details>

**Best Practices:**
- Don’t assume the user has given you all the details to scope appropriately
- They might be focusing on a single host or process, and not considering they’re also interested everything coming from a process group, or a host group
- On the flipside, they may also give you a scope that is too broad and you must narrow it down
- You may have to use a less precise text search to find possible elements
- When creating log metrics, log events, log dashboards, etc –make sure you’re using filters like this, vs a simple text search
- Doing so will ensure the accurate results


### Step 3: Evaluate Conditions

Let’s use that same log example and try some interesting things.

For example, this log has a “started” count and a “Completed” count. Perhaps you’re only interested in seeing when these two numbers do not match.

First, we need to parse out those numbers so we can use them. 

**Write a query with a parse that pulls out the "started" and "Completed" numbers as variables.**

<H3><details>
    <summary>Click to Expand Solution</summary>

```
fetch logs
| filter matchesPhrase(content, "[HeadlessStatistics]") and dt.entity.process_group=="PROCESS_GROUP-A6BBB46BDAADE0F2"
| parse content, "LD 'started [' INT:Started '] Completed [' INT:Completed ']'"
| fields Started, Completed
| fieldsAdd Matching=if(Started==Completed, true, else:false)
| filter Matching==false and isNotNull(Started)

```

</H3></details>

Next, add a field that evaluates if the "started" count matches the "Completed" count. The filter down the results to the cases when they don't match.

<H3><details>
    <summary>Click to Expand Solution</summary>

```
fetch logs
| filter matchesPhrase(content, "[HeadlessStatistics]") and dt.entity.process_group=="PROCESS_GROUP-A6BBB46BDAADE0F2"
| parse content, "LD 'started [' INT:Started '] Completed [' INT:Completed ']'"
| fields Started, Completed
| fieldsAdd Matching=if(Started==Completed, true, else:false)
| filter Matching==false and isNotNull(Started)

```

</H3></details>
