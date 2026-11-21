---
title: Map tiles
description: Where OpenCTI takes the tiles and country boundaries of its maps, and how to replace them with your own files, with no external map server.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/map/
---

OpenCTI draws its maps, such as the locations of an entity or the map widgets of dashboards, from map data it serves itself. This page explains where that data comes from, and how to replace it with a more detailed or regional map. Since version 7.260811, maps work with no configuration, air-gapped deployments included; before, the browsers load the tiles from a tile server.

::::only{opencti=">=7.260811"}

## How OpenCTI serves its maps

The browser renders the maps from vector tiles in a [PMTiles](https://pmtiles.io/) file. OpenCTI serves it at `/maps/world.pmtiles` and answers range requests, so the browser downloads only the tiles of the area and zoom level it shows. No external map server is involved, and the map follows the light or dark theme of the interface.

::diagram{id=deploy-configure-maps}

The tiles come from one of two files:

- the bundled file, in the image of OpenCTI, at the path `APP__MAP_BUNDLED_FILE_PATH` (`./static/maps/world.pmtiles` by default);
- a custom file, uploaded by an administrator and stored in the S3 bucket of OpenCTI.

:::only{opencti=">=7.261030"}
The country boundaries, the polygons that color countries in map widgets, are a second file, served at `/maps/countries.json`. They come the same way from a bundled file, at `APP__MAP_COUNTRIES_BUNDLED_FILE_PATH` (`./static/maps/countries.json` by default), or from a custom file. Replacing one file does not replace the other.
:::

A custom file, when there is one, wins over the bundled one.

:::only{opencti=">=7.261030"}
Browsers keep each file for up to 5 minutes, so an upload or a deletion reaches every browser within that time.
:::

:::only{opencti="<7.261030"}
Browsers keep the tiles for up to 24 hours: after an upload or a deletion, a browser that already showed a map keeps the previous tiles until then, unless its cache is emptied.
:::

The `APP__MAP_*_BUNDLED_FILE_PATH` settings only move the bundled files, for a custom image of OpenCTI: you do not need them to use your own map.

## Get a PMTiles file

[Protomaps](https://protomaps.com/) publishes a daily build of the whole planet, about 137 GB. A low-zoom extract, from zoom 0 to 6, is about 30 to 80 MB and enough for OpenCTI. The [go-pmtiles](https://github.com/protomaps/go-pmtiles) command line extracts it with range requests, without downloading the whole planet:

```bash
pmtiles extract https://build.protomaps.com/20260722.pmtiles world.pmtiles --maxzoom=6
```

Replace the date with that of a recent build. For a regional map, the [Protomaps command line](https://docs.protomaps.com/guide/getting-started) extracts an area with `--bbox`; [planetiler](https://github.com/onthegomap/planetiler) builds tiles from OpenStreetMap data.

## Upload your map

:::only{opencti=">=7.261030"}
:::steps
1. Go to :menu[Settings > Parameters > Map configuration].
2. Open the menu of **Custom map** and choose **Upload**, then select your `.pmtiles` file. When a custom map is already in place, **Replace** uploads a new one in its place.

   :::verify
   The **Custom map** row shows your file, with its name and size. Within 5 minutes, the maps of OpenCTI show your tiles.
   :::
:::

To go back to the bundled map, open the menu of **Custom map**, choose **Delete** and confirm.
:::

:::only{opencti="<7.261030"}
:::steps
1. Go to :menu[Settings > Parameters > Map configuration].
2. In the **Custom map** row, click **Upload**, then select your `.pmtiles` file. A new upload replaces the custom map in place.

   :::verify
   The **Custom map** row shows your file, with its name and size. A browser that already showed a map shows your tiles once its copy of the previous file expires, within 24 hours.
   :::
:::

To go back to the bundled map, click **Delete** in the **Custom map** row: it takes effect at once, with no confirmation.
:::

:::only{opencti=">=7.261030"}

## Upload your country boundaries

The file is a GeoJSON `FeatureCollection`, plain or gzipped. Every feature carries an `ISO3` property: OpenCTI colors a country with the polygons whose `ISO3` matches one of its aliases, and ignores the other properties. The bundled file comes from the `ne_10m_admin_0_countries` layer of [Natural Earth](https://www.naturalearthdata.com/downloads/10m-cultural-vectors/); the script `scripts/generate-countries-geojson.sh` of the OpenCTI repository builds it, and can start your own.

:::steps
1. Go to :menu[Settings > Parameters > Map configuration].
2. Open the menu of **Custom country boundaries** and choose **Upload**, or **Replace** when a custom file is in place, then select your file.

   :::verify
   The row shows your file. In the browser tab you uploaded from, the next map uses the new boundaries at once.
   :::
:::

OpenCTI checks the file and keeps the previous one when the new file is an unreadable gzip archive, is not valid JSON, is not a `FeatureCollection` with at least one feature, or has a feature whose `ISO3` property is missing, empty or not a string. **Download**, in the same menu, returns the stored file as plain GeoJSON. **Delete** goes back to the bundled boundaries.
:::

## Coming from an external tile server

Earlier versions loaded raster tiles from a tile server, set with `APP__MAP_TILE_SERVER_DARK` and `APP__MAP_TILE_SERVER_LIGHT`. OpenCTI now ignores these settings: remove them, and the tile server if you ran one.

::::

::::only{opencti="<7.260811"}

## Tile servers

This version of OpenCTI loads raster map tiles in the browser, from one tile server for the dark theme and one for the light theme:

| Variable | Default |
| --- | --- |
| `APP__MAP_TILE_SERVER_DARK` | `https://map.opencti.io/styles/filigran-dark3/{z}/{x}/{y}.png` |
| `APP__MAP_TILE_SERVER_LIGHT` | `https://map.opencti.io/styles/filigran-light3/{z}/{x}/{y}.png` |

The browsers of your users must reach these servers. In an air-gapped network, run a tile server of your own and set both variables, on the `opencti` service in `docker-compose.override.yml`, to its URL patterns with `{z}`, `{x}` and `{y}`.

From version 7.260811, OpenCTI serves the map itself and no longer needs a tile server.

::::

## What's next

- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/).
- [Configure the shared services](/deploy/configure/shared-services/), for the S3 bucket that holds the custom files.
