# CartoDB / PostGIS Guide
### Michael Weaver
### Yale University

---

## What is CartoDB?

CartoDB is a web-based platform hat integrates geospatial software (PostGIS) with powerful and easy-to-use interactive mapping tools.

Key features:
 1. Hosts your geospatial and other tabular data in the cloud.
 2. Built on the powerful PostgreSQL data platform
 3. Tools for automated design and hosting of interactive maps
 4. Free entry-level accounts
 5. Highly customizable:

	* map the results of complex queries
	* tailor the design of your map using cartocss
	* build custom interactivity using the CartoDB API.

## Examples

[Trees in NYC](http://jillhubley.com/project/nyctrees/#all)

[Subway Status NYC](http://www.datapolitan.com/mta_station_repair_status/)

[British Navy in WWI](http://www.theguardian.com/news/datablog/interactive/2012/oct/01/first-world-war-royal-navy-ships-mapped)

[Evictions in SF](http://www.antievictionmappingproject.net/ellis.html)

[CartoDB Gallery](https://cartodb.com/gallery/)

[My maps](http://mdweaver.github.io/maps)

### Get an account
Free accounts for university affiliates:
https://cartodb.com/signup?plan=academy

### Load data

* csv, tsv, shapefiles, kml, geojson, etc.
* Drag and drop files to load
* Can sync tables with dropbox for dynamic updating
* Data can be points, lines, polygons, etc.

#### New Haven Wards
https://dl.dropboxusercontent.com/u/8139153/cartoDB/nh_wards.zip

Note: shapefiles must be contained in a zipped file

####New Haven SeeClickFix data
https://dl.dropboxusercontent.com/u/8139153/cartoDB/scf.csv

Download these files, and then drag & drop into CartoDB

##Mapping

###Start with polygons
Click on the dataset. See the table, then click on "Map View"

Let's go over the toolbar:

* Layer(s)
* SQL (more on this later)
* Wizards
	* Basic, Choropleth, Category
* Infowindow
* CartoCSS (more on this later)
* Legends
* Add features

###Points
Lets go back to our datasets, click on "scf"

Lets check out the 'wizards'

* Simple
* Category
* Torque
	* Category
	* Heatmap

We can also customize other parts of the map:
	* Basemap
	* Publish map
	* Add text/title
	* Add layer (ward boundaries)

##PostGIS

* Relational database: PostgreSQL
* Addition of geospatial operations

Examples:

* Intersection, union, difference, buffer
* See [this reference](http://postgis.refractions.net/documentation/manual-1.4/ch07.html)

###Basic usage:
Not enough time to introduce SQL completely

```sql
SELECT mytable.*
FROM mytable
WHERE column1 = 'value' and column2 in ('value1', 'value2')
```

```sql
SELECT scf.*
FROM scf
WHERE created_at > '2014-01-01'
```

Use the Filter tool on the toolbar to intuitively select subsets of your data. Automatically updates your SQL query.

###More advanced usage
We can merge tables in PostGIS either by columns or geometry

Common uses:

* Points in polygons
	* in which area is a point located?
	* how many points occur within a given area
* Polygon intersections
	* what areas overlap?
	* how much of one area is inside another?
* Distances between points, points within a radius of other points

Let's do an example of points in Polygon:

* How many points in a polygon
```sql
select nh_wards.*, count(*) AS issue_count
FROM nh_wards, scf
WHERE
st_contains(nh_wards.the_geom,scf.the_geom)
GROUP BY nh_wards.cartodb_id;
```
* Which polygon contains a point
```sql
select scf.*, nh_wards.wards_desc
FROM nh_wards, scf
WHERE
st_contains(nh_wards.the_geom,scf.the_geom)
```

* Which wards border each other?
```sql
SELECT wards_1.*, wards_2.wards_desc as neighboring_ward
FROM nh_wards as wards_1, nh_wards as wards_2
WHERE ST_DWithin(wards_1.the_geom::geography, wards_2.the_geom::geography, 10)
```

* Count SCF issues in a radius around an issue
```sql
SELECT points_1.*, count(*) as count_10m
FROM scf as points_1, scf as points_2
WHERE ST_DWithin(points_1.the_geom::geography, points_2.the_geom::geography, 10) and
GROUP BY points_1.cartodb_id
```

###Custom map:
[This map](http://mdweaver.github.io/scf/) lets us search for terms that appear in the SCF issue.

We can look [through the html](https://github.com/mdweaver/mdweaver.github.io/blob/master/scf/index.html) to see how we construct a new SQL query to return different subsets of the data.

##Questions?

###Other examples

[Railroad travel times](http://mdweaver.github.io/times_year)
Code is [here](https://github.com/mdweaver/mdweaver.github.io/blob/master/times_year/index.html)
