# Parking-Wars-Manhattan

This is a team project with members:
* Butiong, Rafael Antonio - [rafaelantonio.butiong@duke.edu](mailto:rafaelantonio.butiong@duke.edu)
* Du, Jackie - [jacquelyn.du@duke.edu](mailto:jacquelyn.du@duke.edu)
* Huang, Congjun - [congjun.huang@duke.edu](mailto:congjun.huang@duke.edu)
* Luo, Yili - [yili.luo@duke.edu](mailto:yili.luo@duke.edu)

![ticket](nyc_parking_ticket.jpg?raw=true)

## Background

New York City is at the forefront of the open data movement among local, state and federal governments. They have made publicly available a huge amount of data ([NYC Open Data](https://nycopendata.socrata.com/)) on everything from street trees, to restaurant inspections, to parking violations. It is the last of these that we will be focusing on for this team project. 

The original data is quite large (2.01 GB uncompressed CSV file for the 2019 data) and contains more than 9 million observations. These data contains reflect parking violations from all five boroughs of New York City for fiscal years 2019. We have simplified that data significantly by removing all records not from Manhattan and only retaining features having to do with police precincts and ticket locations.

The police precincts in New York are numbered, the following 22 precincts are found in Manhattan: 1, 5, 6, 7, 9, 10, 13, 14, 17, 18, 19, 20, 22, 23, 24, 25, 26, 28, 30, 32, 33, 34.

<br/>

## Data

The violation data along with supplementary data is available on in the `data/` directory of the repository. 

* `manh_tickets_2019.csv.zip` - all parking violations for fiscal year 2019 in Manhattan. The file is zipped to make it small enough to be posted on GitHub - it can still be directly read in using `pd.read_csv()` without needing to unzip it first. 
([source](https://data.cityofnewyork.us/City-Government/Parking-Violations-Issued-Fiscal-Year-2019/faiq-9dfq))

* `manh_pluto.csv` - a csv file containing spatial locations for taxed addresses in Manhattan. Each row contains an address and the latitude and longitude of each property (the centroid of the properties boundary) and is intended to be used for geocoding the parking tickets. 
([source](http://www.nyc.gov/html/dcp/html/bytes/dwn_pluto_mappluto.shtml#mappluto))

* `manh_boundary.csv` - a csv file containing the boundary of the main island of Manhattan (other smaller islands have been removed). 

* `manh_pred.csv` - a csv file containing latitude and longitude of ~23,000 evenly spaced prediction locations within Manhattan.

<br/>

## Step 1 - Geocoding

The original parking violation data contains a large number of variables that we do not care about for this project. Using the simplified version of this data our approach is to attempt to geocode (find latitudes and longitudes) for as many of the parking tickets as possible using the given variables.

The goal is to geocode as many as possible with as much accuracy as possible to enable you to be successful with the 2nd step. This is a messy and large data set and at the best of times geocoding is a very difficult problem - we will go for the low hanging fruit first and then work on the edge cases / exceptions later as needed. The majority of this work would consistent of cleaning and fixing the address data in `manh_tickets_2019` and `manh_pluto` and then merging the two data sets based on the values in their `address` columns.

<br/>

## Step 2 - Recreating NYC's Police Precincts

The ultimate goal of this step is to reconstruct the boundaries of all 22 Manhattan New York City police precincts (numbered between 1 and 34). The parking violation data set contains the column, `precinct`, that lists the police precinct in which the violation ostensibly took place (these can be and are mis-entered in the data). Our goal is to take this data along with the geocoded locations from step 1 and build a model which is capable of predicting boundaries for all 22 precincts.

As mentioned before, the data is complex and messy so there is no guarantee that the reported precinct is correct, or the street address is correct. As such, the goal is not perfection, anything that even remotely resembles the precinct map will be considered a success. No single approach for this estimation is likely to work well, and an iterative approach to cleaning and tweaking will definitely be necessary. We will initially focus on a single precinct to develop our methods before generalizing to the entirety of Manhattan. 

We will treat this as a multivariate classification problem where the labels come from our geocoding and the features are latitude, longitude, and potentially any transformations / interactions of these. We will try multiple different modeling approaches for these data and we will discuss each approach tried and how our final model is selected. All modeling will be performed with scikit-learn.

All tuning of model hyperparameters will be principled and based on appropriate selection using cross validation. We will consider stratifying our cross-validation approach by precinct. The range of values used for searching during hyperparameter tuning will be discussed.

### Central Park

The 13th precinct in Manhattan covers central park, this precinct will be particularly challenging since there are very few commercial buildings for us to use as a basis of geocoding. As such we will likely have almost no points for this particular precinct coming from your step 1 results. 

For this and only this precinct we will impute fake ticket locations and add them to our data frame from step 1. We will do this by finding the coordinates of the four corners of Central Parking and using those to construct a simple boundary from which we sample interior points. 


### Work Product

At the end of this step we will have a fitted model / pipeline which can be used to predict the most likely precicnt for any latitude and longitude coordinate in Manhattan. Specifically, we wll be able to predict from the data contained in `manh_pred.csv`.

<br/>

## Step 3 - Visualize the boundaries

Using the model from step 2 and the prediction locations contained in `manh_pred.csv` , we will create a visualization showing our predicted boundaries across Manhattan. 

<br/>
