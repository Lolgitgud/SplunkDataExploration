# SplunkDataExploration
Collection of searches and analytics to help understand data on your Splunk instance


Search 1:

Search through WALKLEX for a specific string and the indexes which contain it. Then iterate through the indexes looking for fields whose values match a specific pattern. Here, the field maybePN corresponds to the pattern of Norwegian national ID numbers.


| walklex index=[your_index] pattern=[your_pattern] type=term 
| stats count by index 
| map maxsearches=5 search="| search index=\"$index$\"
| fields - date*, time*pos, punct* 
| fieldsummary maxvals=10 
| fields field, count, distinct_count, values 
| eval indexName=\"$index$\""
| rex field=values mode=sed "s/[\[\{\}\]]+//g"
| eval values=trim(values)
| rex max_match=10 field=values "\"value\":\"?(?<value>.*?)\",\"count\":\"?(?<valueCount>\w+)\"?"
| eval valueAndValueCount=mvzip(valueCount,value)
| mvexpand valueAndValueCount
| makemv valueAndValueCount delim=","
| eval valueCount=mvindex(valueAndValueCount,0)
| eval value=mvindex(valueAndValueCount,1)
| table field, count, distinct_count, value, valueCount,indexName
| eval valueCategory=if(isint(value),"Int", if(match(value,"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$"),"IP",if(isstr(value),"String",if(isnull(value), "No Value","Other"))))
| table field, valueCategory, count, value, indexName
| eval maybePN=if(match(value,"[0-3][0-9][0-1][0-9][0-9][0-9]*"), "Yes", "No")
| eval length=len(value)
| where valueCategory="Int" and length=11
| stats sum(count) by indexName, field, valueCategory, length, maybePN


