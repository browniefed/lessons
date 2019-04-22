## Introduction

Knex is a popular library for migrating and querying databases. It's very flexible and has plugins allowing you to extend the abilities of the library. One such plugin is `knex-postgis`. We'll get there eventually but to start by setting up a table to hold onto coordinates, and then get into querying them for information.

We'll end with querying for distance in a radius. This is useful if your app has GPS coordinates of something and you need to search if anything is with in a particular radius away.

## Knex

To setup a knex and migrations you can follow the official docs here [https://knexjs.org/#Migrations](https://knexjs.org/#Migrations).

Once you have installed the installed the global CLI `npm install knex -g` in your project directory you can then run `knex init`. This will create specific files that you'll modify to specify your connection to your database.

You'll need modify those with the location/username/password to be able to connect.

To then create a migration you run `knex migration:make NAME_OF_YOUR_MIGRATION`. This name is nothing special, it's just to help you identify what the migration is doing.

This will create a file in a `migrations` folder that has functions for how to setup the tables/columns, and then commands for undoing the creation. This is essential for rolling back the changes and also re-applying them.

## Enable PostGIS

Before we move on the database needs to have PostGIS enabled. Running on RDS or many other cloud services the plugin is present but not enabled. To enable it you need to run this command for the database you're connecting to

```sql
create extension postgis;
```

## Setup PostGIS and a Column

Once we have our up/down, in our up we can define a new table. Knex doesn't have defaults for defining schema that are PostGIS enabled but with it's `specificType` schema creator you can define anything you want.

```js
exports.up = function(knex) {
  return knex.schema.createTable("locations", table => {
    table.increments("id").primary();
    table.specificType("coordinates", "geometry(point, 4326)");
  });
};
```

The specific type we want to create is a geometry point with the 4326 SRID (spatial reference identifier). For a more in depth explanation check this post on the [GIS StackExchange](https://gis.stackexchange.com/questions/131363/choosing-srid-and-what-is-its-meaning).

```
Basically, PostGIS opens up the ability to store your data in a single coordinate system such as WGS84 (SRID 4326).
```

## Save Data

To insert data into this `coordinate` column we need to construct a query to insert a point.

```sql
INSERT INTO "locations" ("coordinates") VALUES (ST_SetSRID(ST_MakePoint(LONGITUDE,LATITUDE), 4326));
```

With data:

```sql
INSERT INTO "locations" ("coordinates") VALUES (ST_SetSRID(ST_MakePoint(-122.079513,45.607703), 4326));
```

You can see the structure of the insert matches the column we created. We are creating a point with the `long/lat` then setting the SRID to `4326`. In GIS longitude is defined before the latitude. There are other ways to save and build out the data but this is by far the quickest.

To update we execute the same but with the update command

```sql
UPDATE locations SET coordinates = ST_SetSRID(ST_MakePoint(LONGITUDE,LATITUDE), 4326);
```

With data:

```sql
UPDATE locations SET coordinates = ST_SetSRID(ST_MakePoint(-122.079513,45.607703), 4326);
```

## Query Data

Now we want to query for data. This is

```sql
SELECT *
FROM locations
WHERE ST_DWithin(coordinates, ST_MakePoint(-122.079513,45.607703)::geography, 1000);
```

The 1000 unit is in meters. So if you're needing a dynamic query and you're input is in miles/feet you will need to convert those values to meters. Also we don't have any data to select in our table so we can just return the coordinates. However the underyling data structure won't return the longitude/latitude without using PostGIS functions.

```sql
SELECT ST_X(coordinates) as longitude, ST_Y(coordinates) as latitude FROM locations;
```

This will return the actual longitude/latitude from the `coordinates` column.

# Knex PostGIS

Knex allows you to write chained commands that can construct safe queries to execute your database. So rather than writing the queries above you can use the `knex-postgis` plugin. You can find the github repo here [https://github.com/jfgodoy/knex-postgis#readme](https://github.com/jfgodoy/knex-postgis#readme)

You'll need to setup the plugin to hook into connect like so

```js
const knex = require("knex");
const knexPostgis = require("knex-postgis");

const db = knex({
  dialect: "postgres"
});

// install postgis functions in knex.postgis;
const st = knexPostgis(db);
```

or if you need access somewhere else and don't have access to `st` variable.

```js
const knex = require("knex");
const knexPostgis = require("knex-postgis");

const db = knex({
  dialect: "postgres"
});

// install postgis functions in knex.postgis;
const st = db.postgis;
```

This will give you the ability to use `st` to construct queries.

Lets walk through inserting and updating with knex.

Inserting data

```js
knex("locations").insert({
  coordinates: st.setSRID(st.makePoint(longitude, latitude, 4326)
});
```

Updating data

```js
knex("locations").update({
  coordinates: st.setSRID(st.makePoint(longitude, latitude, 4326)
});
```

Querying and returning data

```js
knex("locations").select(st.x("coordinates").as("longitude"), st.y("coordinates").as("latitude"));
```

Querying radius

```js
knex("locations")
  .select(
    st.distance("coordinates", st.geography(st.makePoint(longitude, latitude))).as("distanceAway")
  )
  .where(st.dwithin("coordinates", st.geography(st.makePoint(longitude, latitude)), 64373.8));
```

This will return the distance away in meters for saved locations in the `coordinates` table that are within a 64373.8 meters (50 miles) radius.

# Ending

Knex and Postgres are a powerful combination for creating/managing your database and safely querying for data. You're now able to leverage PostGIS and query for latitude/longitude that are with in a meter radius.
