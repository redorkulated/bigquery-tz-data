# BigQuery Public Dataset for Timezone Data
An public mirror of the [tzdb](https://www.iana.org/time-zones) (timezone database) from IANA and the [timezone boundaries](https://github.com/evansiroky/timezone-boundary-builder) within BigQuery.

This is being offered as a "best effort" replication to support anybody who would need this information within [BigQuery](https://cloud.google.com/bigquery) .


## Releases
The data will be mirrored into the BigQuery on a best-effort basis so might lag behind the releases found from both sources.

## What is BigQuery?
Bigquery is a managed data warehouse from the Google Cloud platform; the official documentation can be found [here](https://cloud.google.com/bigquery) . There can be costs from Google Cloud when consuming this data; it is advisable to checkout the [pricing page](https://cloud.google.com/bigquery/pricing) before consuming.

## Contents
### Release Version Identifiers
The release version identifiers are based on the IANA timezone release identifiers, today this is in the format of `YYYYx`; for example `2023d`.

### EU vs US
All data is replicated into both the EU and US [Multi Regions](https://cloud.google.com/bigquery/docs/locations#multi-regions). The datasets all have a suffix to represent which region the data is stored within. Both regions contain all global data and are provided as a convenience to the consumer.

### Latest Version
Two datasets which will contain the most recent version for each table. It could be that the tables from each source has a different latest version.

There are some more technical tables (such as raw JSON tables) that are not available within the latest version and need to be sourced per version; this is to ensure that the latest tables and content are stable (where-as source JSON *might* change.

| Dataset Project | EU Dataset | US Dataset | tzdb version | Boundary version |
|-----------------|------------|------------|--------------|------------------|
| tz-data         | [latest_EU](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=latest_EU&t=timezones) | [latest_US](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=latest_US&t=timezones) | 2023d | 2023d |

### Version Datasets
Alternatively to the latest version, previous versions are kept as a historical record and to ensure you can migrate to the next version at your own pace.
| Version | Dataset Project | EU Dataset | US Dataset | Has tzdb | Has Boundary |
|---------|-----------------|------------|------------|----------|--------------|
| 2023d   | tz-data         | [release_2023d_EU](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=release_2023d_EU&t=timezones) | [release_2023d_US](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=release_2023d_US&t=timezones) |     Y    |       Y      |
| 2023b   | tz-data         | [release_2023b_EU](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=release_2023b_EU&t=timezones) | [release_2023b_US](https://console.cloud.google.com/bigquery?page=table&p=tz-data&d=release_2023b_US&t=timezones) |     Y    |       Y      |

### Detailed Contents
Here is a list of the tables currently available. Unless otherwise mentioned the schema of these tables is the same across versions. For more detailed descriptions of the contents I would suggest reading the documentation from the source directly.
| Table Name                      | Source                    | In Latest? | Description                                                             | Notes       |
|---------------------------------|---------------------------|------------|-------------------------------------------------------------------------|-------------|
| timezones                       | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | Comprehensive list of timezones (excluding oceans)                      | See examples below |
| timezones-1970                  | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | List of timezones valid since 1970 (excluding oceans)                   | Since 2023d |
| timezones-now                   | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | Timezones currently valid (excluding oceans)                            | Since 2023d |
| timezones-with-oceans           | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | Comprehensive list of timezones                                         |             |
| timezones-with-oceans-1970      | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | List of timezones valid since 1970                                      | Since 2023d |
| timezones-with-oceans-now       | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |      Y     | Timezones currently valid                                               | Since 2023d |
| timezones_json                  | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, Comprehensive list of timezones (excluding oceans)    |             |
| timezones-1970_json             | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, List of timezones valid since 1970 (excluding oceans) | Since 2023d |
| timezones-now_json              | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, Timezones currently valid (excluding oceans)          | Since 2023d |
| timezones-with-oceans_json      | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, Comprehensive list of timezones                       |             |
| timezones-with-oceans-1970_json | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, List of timezones valid since 1970                    | Since 2023d |
| timezones-with-oceans-now_json  | [timezone-boundary-builder](https://github.com/evansiroky/timezone-boundary-builder) |            | JSON import data, Timezones currently valid                             | Since 2023d |
| tzdb_links                      | [IANA tzdb](https://www.iana.org/time-zones) |      Y     | Timezone links (aliases)                                                |             |
| tzdb_rules                      | [IANA tzdb](https://www.iana.org/time-zones) |      Y     | Timezone rules                                                          |             |
| tzdb_zones                      | [IANA tzdb](https://www.iana.org/time-zones) |      Y     | Timezones                                                               |             |


## Examples

### Lookup a timezone from a set of coordinates
```
SELECT
    timezone_id
FROM
    `tz-data.latest_EU.timezones`
WHERE
    ST_WITHIN(
        ST_GEOGPOINT(
            /* longitude */ -0.13948559859956436,
            /* latitude */ 51.5029659891868
        ),
        geometry
    );
LIMIT 1
```
**Return Value:**
|  timezone_id  |
| ------------- |
| Europe/London |

### Get timezones for multiple places
Using the standard list, if a place is within the ocean it will not return a timezone.

```
WITH places AS (
  SELECT "Buckingham Palace" AS place,51.500833 AS lat,-0.141944 AS long,
  UNION ALL SELECT "Null Island" AS place,0.0 AS lat,0.0 AS long,
  UNION ALL SELECT "Statue of Liberty" AS place,40.689167 AS lat,-74.044444 AS long,
  UNION ALL SELECT "Sydney Opera House" AS place,-33.858611 AS lat,151.214167 AS long,
  UNION ALL SELECT "Taj Mahal" AS place,27.175 AS lat,78.041944 AS long
)
SELECT
  p.place,
  p.lat,
  p.long,
  t.timezone_id,
  CURRENT_TIMESTAMP AS utc_time,
  DATETIME(CURRENT_TIMESTAMP,t.timezone_id) AS local_time
FROM
  places p
  LEFT JOIN `tz-data.latest_EU.timezones` t
    ON ST_WITHIN(
        ST_GEOGPOINT(
          p.long,
          p.lat
        ),
        t.geometry
    )
ORDER BY
  p.place ASC
```
**Return Value:**
| place              | lat        | long       | timezone_id                    | utc_time                       | local_time                 |
|--------------------|------------|------------|--------------------------------|--------------------------------|----------------------------|
| Buckingham Palace  | 51.500833  | -0.141944  | Europe/London                  | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T08:21:33.747150 |
| Null Island        | 0.0        | 0.0        |                                | 2023-12-31 08:21:33.747150 UTC |                                |
| Statue of Liberty  | 40.689167  | -74.044444 | America/New_York               | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T03:21:33.747150 |
| Sydney Opera House | -33.858611 | 151.214167 | Australia/Sydney               | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T19:21:33.747150 |
| Taj Mahal          | 27.175     | 78.041944  | Asia/Kolkata                   | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T13:51:33.747150 |

### Get timezones for multiple places, including the oceans
Here NULL island returns a timezone, as it within the ocean.

```
WITH places AS (
  SELECT "Buckingham Palace" AS place,51.500833 AS lat,-0.141944 AS long,
  UNION ALL SELECT "Null Island" AS place,0.0 AS lat,0.0 AS long,
  UNION ALL SELECT "Statue of Liberty" AS place,40.689167 AS lat,-74.044444 AS long,
  UNION ALL SELECT "Sydney Opera House" AS place,-33.858611 AS lat,151.214167 AS long,
  UNION ALL SELECT "Taj Mahal" AS place,27.175 AS lat,78.041944 AS long
)
SELECT
  p.place,
  p.lat,
  p.long,
  t.timezone_id,
  CURRENT_TIMESTAMP AS utc_time,
  DATETIME(CURRENT_TIMESTAMP,t.timezone_id) AS local_time
FROM
  places p
  LEFT JOIN `tz-data.latest_EU.timezones` t
    ON ST_WITHIN(
        ST_GEOGPOINT(
          p.long,
          p.lat
        ),
        t.geometry
    )
ORDER BY
  p.place ASC
```
**Return Value:**
| place              | lat        | long       | timezone_id                    | utc_time                       | local_time                 |
|--------------------|------------|------------|--------------------------------|--------------------------------|----------------------------|
| Buckingham Palace  | 51.500833  | -0.141944  | Europe/London                  | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T08:21:33.747150 |
| Null Island        | 0.0        | 0.0        | Etc/GMT                        | 2023-12-31 08:21:33.747150 UTC | 2023-12-31 08:21:33.747150 UTC |
| Statue of Liberty  | 40.689167  | -74.044444 | America/New_York               | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T03:21:33.747150 |
| Sydney Opera House | -33.858611 | 151.214167 | Australia/Sydney               | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T19:21:33.747150 |
| Taj Mahal          | 27.175     | 78.041944  | Asia/Kolkata                   | 2023-12-31 08:21:33.747150 UTC | 2023-12-31T13:51:33.747150 |


## Licenses

- timezone-boundary data is licensed under the [Open Data Commons Open Database License (ODbL)](http://opendatacommons.org/licenses/odbl/).

- IANA timezone database license can be found [here](https://www.iana.org/help/licensing-terms)
