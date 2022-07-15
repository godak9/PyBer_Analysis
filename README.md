# PyBer_Analysis: Summarizing rideshare data using Matplotlib
## Overview
Explain the purpose of the new analysis.
Data from two csv files was provided (city_data.csv and ride_data.csv).The task was to create a summary DataFrame of the ride-sharing data by city type and create a multiple-line graph using Pandas and Matplotlib that shows the total weekly fares for each city during a period in 2019. The graph revealed insights into the different city types which are summarized in this report. 
## Results
After loading the csv files and reading them into dataframes, the two dataframes were merged on "city" so that a dataframe was created with "city", "date", "fare", "ride_id", "driver_count", and "type" as columns. 
### Summary DataFrame with grouping by city type
The first task was to create a summary dataframe that grouped data by city "type". Below is the code used to create the dataframe (found in the PyBer_Challenge.ipynb file).
```
#  1. Get the total rides for each city type
total_rides_per_city_type= pyber_data_df.groupby(["type"]).count()["ride_id"]

# 2. Get the total drivers for each city type
# Calculate the percentage of drivers for each city type.
sum_drivers_by_city_type = city_data_df.groupby(["type"]).sum()["driver_count"]

#  3. Get the total amount of fares for each city type
sum_fares_by_city_type = pyber_data_df.groupby(["type"]).sum()["fare"]

#  4. Get the average fare per ride for each city type. 
# Create the Urban city DataFrame.
average_fare_by_type = sum_fares_by_city_type / total_rides_per_city_type

# 5. Get the average fare per driver for each city type. 
average_fare_for_driver_by_city_type = sum_fares_by_city_type/sum_drivers_by_city_type

#  6. Create a PyBer summary DataFrame. 
pyber_summary_df = pd.DataFrame({"Total Rides": total_rides_per_city_type,
                                 "Total Drivers": sum_drivers_by_city_type,
                                 "Total Fares": sum_fares_by_city_type,  
                                 "Average Fare per Ride": average_fare_by_type,
                                 "Average Fare per Driver": average_fare_for_driver_by_city_type})

#  7. Cleaning up the DataFrame. Deleting the index name
pyber_summary_df.index.name = None

#  8. Format the columns.
# Format "Total Rides" to have the comma for a thousands separator, a decimal separator.
pyber_summary_df["Total Rides"] = pyber_summary_df["Total Rides"].map("{:,}".format)
# Format "Total Drivers" to have the comma for a thousands separator, a decimal separator.
pyber_summary_df["Total Drivers"] = pyber_summary_df["Total Drivers"].map("{:,}".format)
# Format "Total Fares" to have the comma for a thousands separator, a decimal separator and a "$".
pyber_summary_df["Total Fares"] = pyber_summary_df["Total Fares"].map("${:,.2f}".format)
# Format the "Average Fare per Ride" to have the comma for a thousands separator, a decimal separator and a "$".
pyber_summary_df["Average Fare per Ride"] = pyber_summary_df["Average Fare per Ride"].map("${:,.2f}".format)
# Format the "Average Fare per Driver" to have the comma for a thousands separator, a decimal separator and a "$".
pyber_summary_df["Average Fare per Driver"] = pyber_summary_df["Average Fare per Driver"].map("${:,.2f}".format)
```
With the code above, the following dataframe was generated.

![Summary_df](https://user-images.githubusercontent.com/104794100/179130119-1ea1f7b8-6e50-4461-a0a2-5fe7c5c1c8f7.png)

### Multiple-line chart of total fares for each city type
The second task was to create a multiple-line graph that showed the differences in total fares for each week between January 2019 and April 2019 by city type. A new dataframe was created where the data was grouped by city "type" and "date". The pivot() function was used to convert the dataframe so that the index was the "date" as the datetime data type. The resample() fucntion was used to resample the data into weekly bins, and, finally, the resampled dataframe was graphed using the object-oriented interface method and the df.plot() method. Below is the code used to create the dataframe and the multiple-line graph. (found in the PyBer_Challenge.ipynb file).
```
# 2. Using groupby() to create a new DataFrame showing the sum of the fares 
#  for each date where the indices are the city type and date.
total_fare_per_date_df =pyber_data_df.groupby(["type", "date"]).sum()[["fare"]]

# 3. Reset the index on the DataFrame
total_fare_per_date_df = total_fare_per_date_df.reset_index()

# 4. Create a pivot table with the 'date' as the index, the columns ='type', and values='fare' 
# to get the total fares for each type of city by the date. 
city_sum_fare = total_fare_per_date_df.groupby(["type", "date"]).sum()[["fare"]]
city_sum_fare = city_sum_fare.reset_index()
total_fare_per_date_pivot = total_fare_per_date_df.pivot(index="date", columns="type", values="fare")
total_fare_per_date_pivot.columns.name = None

# Create a new DataFrame from the pivot table DataFrame using loc on the given dates, '2019-01-01':'2019-04-29'.
filtered_fare_per_date_df = total_fare_per_date_pivot.loc['2019-01-01':'2019-04-29']

# Set the "date" index to datetime datatype. 
filtered_fare_per_date_df.index = pd.to_datetime(filtered_fare_per_date_df.index)

# Check that the datatype for the index is datetime using df.info()
filtered_fare_per_date_df.info()

# Create a new DataFrame using the "resample()" function by week 'W' and get the sum of the fares for each week.
city_fare_per_week_df = filtered_fare_per_date_df.resample("W").sum()

# Using the object-oriented interface method, plot the resample DataFrame using the df.plot() function. 
# Import the style from Matplotlib.
from matplotlib import style
# Use the graph style fivethirtyeight.
style.use('fivethirtyeight')
# Use object-oriented interface method and df.pot() method
ax = city_fare_per_week_df.plot(figsize=(16,6))
ax.plot()
ax.set_title('Total Fare by City Type')
ax.set_ylabel('Fare ($USD)')
ax.set_xlabel('Jan though Apr 2019')

plt.savefig("Analysis/PyBer_fare_summary.jpg")
```
With the code above the following graph was created. As shown, the total fares by week in urban cities was greater than total fares in suburban and rural cities, and total fares by week in suburban cities was greater than total fares in rural cities. Between Januray and April 2019, total fares peaked in urban cities in the last week of February and in the second week of March, in suburban cities in the last week of February, and in rural cities in the first week of April. 

![PyBer_fare_summary](https://user-images.githubusercontent.com/104794100/178580598-e418f8e2-7518-44cd-9753-1ff10d2d5f6f.jpg)

## Summary and Recommendations
Although the average fare per ride in urban cities is lower than it is in suburban and rural cities (by ~$6.00 and ~$10.00, respectively), the total fares in urban cities far greatly exceeded the total fares in suburban and rural cities (by ~$35,500 and ~20,300, respectively). Because of this, Pyber should target urban audiences in general advertsing because the service is most widely used in urban cities, especially during mid-February until mid-March since 2019 data showed peaks in total fares during the last week of February and in the second week of March.

The total number of rides in rural cities was far less than it was in suburban and urban cities (by ~500 and ~1,500, respectively). However, the ratio of total rides to total drivers was the greatest in rural cities at ~1.6:1 where in suburban cities it was at ~1.3:1 and in urban cities at ~ 0.7:1. This means the drivers in rural cities are receiving a larger average fare. Because of this, Pyber should allocate any bonus funds towards opportunities for drivers in urban cities since there are more drivers and less average fare per driver.

Again, the total number of rides in rural cities was far less than it was in suburban and urban cities ( by ~500 and ~1,500, respectively). PyBer should create specialized advertising in rural cities to increase its usage in these cities. Although the average fare per ride in rural cities is greater than it is in suburban and urban cities ( by ~$4.00 and ~$10.00, respectively) which may deter people from using the ride-sharing service, it also may be because people in rural cities are not aware of the ride-sharing service. Since total fares in rural cites peaked during the first week of April 2019, PyBer should put most focus on advertising in rural cities between the end of March and the beginning of April.


 
