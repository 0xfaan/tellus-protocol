# Oracle Contract Notes

This document describes the oracle contract that exists in the repository today.

## Current Contract

The oracle contract stores the latest submitted reading for a geohash and reading type.

Supported reading types:

- `Rainfall`
- `NDVI`
- `SoilMoisture`

Stored reading fields:

- `geo_cell`: geohash-like location identifier.
- `reading_type`: reading category.
- `value`: unsigned integer value.
- `timestamp`: current ledger timestamp at submission.

## Public Methods

### `initialize(admin)`

Initializes the contract with an admin address. The admin is stored, but submitter authorization is not enforced by the current implementation.

### `submit_reading(geo_cell, reading_type, value)`

Stores a latest reading for the `(geo_cell, reading_type)` pair. A later submission for the same pair replaces the previous value.

### `get_latest(geo_cell, reading_type)`

Returns the latest stored reading for the pair, or `NoReadingsAvailable` if no reading has been submitted.

## Current Limitations

- Anyone can submit a reading.
- The contract stores only the latest reading for each pair.
- There is no historical time series.
- There is no median aggregation.
- There is no signer whitelist.
- There is no signature verification.
- There is no data source verification.
- The trigger contract does not currently consume oracle readings.

## Data Units

The contract stores raw unsigned integer values and does not enforce units. Tests and integrations should document the units they use, such as millimeters for rainfall or scaled integer values for NDVI.
