# TimeZone Map

Get timezone by latitude and longitude coordinates, similar Google TimeZone API.


### Required

* Mysql: >= 5.6


### Use

Execute a sql query for find timezone:

```sql
SELECT `Name` FROM `zone` WHERE ST_Contains(`Location`, POINT(37.620393, 55.75396));
```
Query must retrun string "Europe/Moscow"

*Function POINT has arguments (Longitude, Latitude)*


### Manual assembly timezone table

1) Download and unzip file tz_world.zip in http://efele.net/maps/tz/world/

```bash
wget http://efele.net/maps/tz/world/tz_world.zip -O tz_world.zip &&
unzip tz_world.zip && 
rm -f tz_world.zip
```

2) Install [ogr2ogr](http://www.osgeo.org) tools included GDAL/OGR

[Install manual for CentOS](https://github.com/wavded/ogre/wiki/Compiling-a-recent-ogr2ogr-from-source-on-CentOS-(RHEL))

3) Convert tz_world.shp to CSV foramt use ogr2ogr
```bash
/usr/local/bin/ogr2ogr -f "CSV" tz_world world/tz_world.shp -lco GEOMETRY=AS_WKT && 
mv tz_world/tz_world.csv tz_world.csv && 
rm -rf tz_world world
```

4) Create table structure
```sql
CREATE TABLE `zone` (
  `Id` int(11) NOT NULL AUTO_INCREMENT,
  `Name` varchar(255) NOT NULL,
  `Location` geometry NOT NULL,
  PRIMARY KEY (`Id`),
  SPATIAL KEY `Location` (`Location`)
) ENGINE=MyISAM;
```

5) Import tz_world.csv in DB
```sql
LOAD DATA INFILE '/path/to/tz_world.csv' 
INTO TABLE zone 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(@Location, @Name)
SET 
id := null,
Name := @Name,
Location := GeomFromText(@Location);
```
