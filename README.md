FreeBike GPS — Offline Database
Overview
Each supported country has its own SQLite database file hosted on GitHub Releases.
Format: {cc}.db.gz (gzip-compressed SQLite)
URL pattern: https://github.com/freebikegps/offline-data/releases/download/{CC}/{cc}.db.gz
Schema version: 3
Supported Countries
CodeCountryFlagnlNetherlands🇳🇱plPoland🇵🇱deGermany🇩🇪beBelgium🇧🇪frFrance🇫🇷esSpain🇪🇸atAustria🇦🇹chSwitzerland🇨🇭czCzechia🇨🇿dkDenmark🇩🇰gbUnited Kingdom🇬🇧itItaly🇮🇹seSweden🇸🇪noNorway🇳🇴
Database Schema
places — Cities, towns, villages
sqlCREATE TABLE places (
    id          INTEGER PRIMARY KEY,
    name        TEXT    NOT NULL,
    lat         REAL    NOT NULL,
    lon         REAL    NOT NULL,
    place_type  TEXT,               -- city, town, village, hamlet, suburb, neighbourhood
    importance  INTEGER DEFAULT 0   -- 100=city, 80=town, 60=village, 40=hamlet
);
CREATE INDEX idx_places_latlon ON places(lat, lon);
CREATE VIRTUAL TABLE places_fts USING fts4(content='places', name);
streets — Street names
sqlCREATE TABLE streets (
    id    INTEGER PRIMARY KEY,
    name  TEXT    NOT NULL,
    city  TEXT,                     -- nearest city/town name
    lat   REAL    NOT NULL,
    lon   REAL    NOT NULL
);
CREATE INDEX idx_streets_latlon ON streets(lat, lon);
CREATE INDEX idx_streets_name ON streets(name COLLATE NOCASE);
CREATE VIRTUAL TABLE streets_fts USING fts4(content='streets', name);
poi — Points of Interest
sqlCREATE TABLE poi (
    id        INTEGER PRIMARY KEY,
    category  TEXT    NOT NULL,     -- see POI Categories below
    name      TEXT,
    lat       REAL    NOT NULL,
    lon       REAL    NOT NULL
);
CREATE INDEX idx_poi_latlon ON poi(lat, lon);
CREATE INDEX idx_poi_cat    ON poi(category);
meta — Metadata
sqlCREATE TABLE meta (
    key   TEXT PRIMARY KEY,
    value TEXT
);
-- Keys: country, built_at, schema_version (=3)
POI Categories
Categories match the PoiCategory enum in the app:
CategoryIconOSM TagsPUMP🔧amenity=bicycle_pump, amenity=compressed_airREPAIR🔧amenity=bicycle_repair_stationVIEWPOINT🏔️tourism=viewpointWATER💧amenity=drinking_water, amenity=fountainBIKE_SHOP🚲shop=bicycle, shop=e-bikeDINING🍽️amenity=cafe, amenity=restaurant, amenity=bakeryTOILETS🚻amenity=toiletsGROCERY🛒shop=supermarket, shop=convenienceEMERGENCY🆘amenity=hospital, amenity=pharmacy, amenity=police
Data Source
All data is derived from OpenStreetMap via Geofabrik extracts.
License: © OpenStreetMap contributors, ODbL 1.0
Building the Database
Use the provided Python script:
bash# Install dependencies
pip install osmium requests

# Build single country
python build_freebike_db_v4.py nl

# Build all countries
python build_freebike_db_v4.py all

# Add POI to existing databases (without rebuilding addresses)
python add_poi_to_db.py nl
python add_poi_to_db.py all
Requirements

Python 3.8+
osmium (pip install osmium)
requests (pip install requests)
~16 GB RAM for large countries (FR, DE, GB)
Disk space: ~5-50 GB per country (PBF files)

Output
Files are saved to output/ directory:

{cc}.db — SQLite database
{cc}.db.gz — Compressed database (upload to GitHub Releases)
{cc}.osm.pbf — Cached OSM extract (reused on next build)

Schema Version History
VersionChanges1Initial: places, streets2Added FTS indexes, city column in streets3Added poi table with 9 categories
