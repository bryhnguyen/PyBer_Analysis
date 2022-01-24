# PyBer_Analysis
## Overview of the PyBer_Analysis
The purpose of this PyBer analysis was to demonstrate proficiency and knowledge of Python, Pandas, and Matplotlib by: 
* Creating a summary DataFrame of the ride-sharing data by city type
* Creating a multiple-line graph that shows the total weekly fares for each city type
* Writing a summary about how the data differs by city type and how those differences can be used by decision-makers at PyBer
By completing these tasks, we are able to identify information describing performance of PyBer's services based on the city type (Urban, Suburban, Rural), average fare per driver, and number of drivers per city type. These key metrics will be used in decision-making in business strategy changes at PyBer. 

## Results
After merging dataframes from the supplied city_data.csv and ride_data.csv files, we had to create a Summary DataFrame. 
The PyBer summary DataFrame is created by using the groupby() functions with the count() and sum() methods on PyBer DataFrame columns to:

```
#  1. Get the total rides for each city type
total_rides_by_type = pyber_data_df.groupby(["type"]).count()["ride_id"]
# 2. Get the total drivers for each city type
total_drivers_by_type = city_data_df.groupby(["type"]).sum()["driver_count"]
#  3. Get the total amount of fares for each city type
sum_fares_by_type = pyber_data_df.groupby(["type"]).sum()["fare"]
#  4. Get the average fare per ride for each city type. 
avg_fare_per_ride_by_type = pyber_data_df.groupby(["type"]).mean()["fare"]
```
Then, average fare per driver for each city type was calculated by dividing sum_fares_by_type by total_drivers_by_type.
```
# 5. Get the average fare per driver for each city type. 
avg_fare_per_driver_by_type = sum_fares_by_type / total_drivers_by_type
```
To create the dataframe, we used this code to organize the created data series into a dataframe.
```
pyber_summary_df = pd.DataFrame({
          "Total Rides" : total_rides_by_type ,
          "Total Drivers": total_drivers_by_type ,
          "Total Fares": sum_fares_by_type,
          "Average Fare per Ride": avg_fare_per_ride_by_type,
          "Average Fare per Driver": avg_fare_per_driver_by_type})
pyber_summary_df
```
The dataframe is will still need to be formatted to display fare as currency (USD) to two decimal points.
```
pyber_summary_df.index.name = None
pyber_summary_df["Total Rides"] = pyber_summary_df["Total Rides"].map("{:.0f}".format)

pyber_summary_df["Total Drivers"] = pyber_summary_df["Total Drivers"].map("{:.0f}".format)

pyber_summary_df["Total Fares"] = pyber_summary_df["Total Fares"].map('${:,.2f}'.format)

pyber_summary_df["Average Fare per Ride"] = pyber_summary_df["Average Fare per Ride"].map('${:,.2f}'.format)

pyber_summary_df["Average Fare per Driver"] = pyber_summary_df["Average Fare per Driver"].map('${:,.2f}'.format)

pyber_summary_df
```
![PyBer_Fare_Dataframe](/analysis/Pyber_Fare_Dataframe.png)


Our end result is the above dataframe where we can see key metrics like Total Rides, Average Fare per Ride, Average Fare per Driver, and the City type!

Now the line plot must be created by creating a new dataframe. Using the groupby() function again, a dataframe is created on the "type" and "date" columns, and the sum() method is applied on the "fare" column to show the total fare amount for each date and time.
```
multiple_line_plot_df = pyber_data_df.groupby(["type","date"]).sum()["fare"]
```
Before we can move on to create a pivot table with the pivot() function, the index must be reset on the dataframe using:
```
multiple_line_plot_df = multiple_line_plot_df.reset_index()
```

Now that the dataframe's index has been reset, a pivot table dataframe is created with the pivot() function where the index is the "date," the columns are the city "type," and the values are the "fare." to get the total fares for each type of city by the date.
```
pivot_table = multiple_line_plot_df.pivot(index ='date',columns = 'type', values = 'fare')
```
Using this pivot table dataframe, we are creating another dataframe for a range of dates between 2019-01-01 and 2019-04-29 using the loc method. Then, we set the "date" index to datetime datatype. This is necessary to use the resample() method later.
```
new_df = pivot_table.loc["2019-01-01":"2019-04-29"] 
new_df.index = pd.to_datetime(new_df.index)
```
Before we plot the multiple line graph, we created a new DataFrame using the aforementioned "resample()" function by week 'W' and got the sum of the fares for each week.
```
resampled_df = new_df.resample('W').sum()
```

Finally, we plot the multiple line graph used this dataframe and saved it as PyBer_fare_summary.png in our analysis folder.
```
resampled_df.plot(figsize = (15,5))

from matplotlib import style

style.use('fivethirtyeight')

plt.title("Total Fare By City Type")
plt.ylabel("Fare ($USD)")
plt.xlabel("Months")

lgnd = plt.legend(loc="center", title="type")

plt.savefig("analysis/PyBer_fare_summary.png")
plt.show()
```

![PyBer_Fare_Summary](/analysis/PyBer_fare_summary.png)

## Summary

Now that we are able to look at the key metrics surrounding city types, fares, rides, and drivers, we can identify any trends to plan and make decisions at PyBer. 
Looking at the pivot table, we see that the city type: Urban has the lowest average fare per driver and that the other two city types have a higher average fare per driver than average fare per ride. This could be a result of shorter trips and too low of a flat charge. A conclusion that can be drawn from this observation is that drivers would benefit from increasing the base fee per trip to compensate for lower mileage and shorter trips. 

  From what can be observed on the multiple line graph, the highest increase for all city types occurs towards the end of February from the middle of February. We would want to make sure that there are enough drivers to meet the increase in demand. Since all city types saw a fare increase at that time, hiring more drivers would be the solution.
  
  A final observation from the multiple line graph shows Suburban fares begin to increase as the month of April escalates, while the other two city types steadily decline. We want to also make sure there are enough drivers to meet the increased demand, perhaps transitioning drivers from the Urban and Rural cities. Transitioning drivers from other city types might prove to be most effective as the decrease in fares may be a result of a decreased demand. 
