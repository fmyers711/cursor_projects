# Airport CIM — Amadeus AODB master mapping

Field mapping from Common Information Model (CIM) to Amadeus AODB and related sources. Generated from `airport-cim-amadeus-aodb-master.csv`.

| dataset_name | cim_field_name | cim_data_type | description | primary_system | amadeus_aodb_native_field | amadeus_export_typical_path | secondary_source_optional | pii_class | precedence_note | suggested_sourcetype |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Airfield | lastUpdated | int(10) | Record last update; candidate _time for AODB streams | Amadeus AODB | LAST_UPDATE_TS | DB_VIEW / EVENT_BUS CDC |  | internal | Use for event ordering vs operational SOBT filters per Splunk4Airports pattern | cim:airfield:aodb:amadeus |
| Airfield | flightUID | String | Unique flight leg identifier | Amadeus AODB | FLIGHT_LEG_OID | DB_VIEW.FLIGHT_LEG |  | internal | Stable per leg; confirm OID vs airline rotation key with tenant DBA | cim:airfield:aodb:amadeus |
| Airfield | flightNumber | int(4) | Marketing flight number | Amadeus AODB | MARKETING_FLIGHT_NUMBER | DB_VIEW.FLIGHT_LEG |  | none | Strip/pad per carrier convention at ingest | cim:airfield:aodb:amadeus |
| Airfield | serviceType | String(1) | Operational service type (sched/charter/etc.) | Amadeus AODB | SERVICE_TYPE_CODE | DB_VIEW.FLIGHT_LEG |  | none | Map to Splunk serviceTypes lookup | cim:airfield:aodb:amadeus |
| Airfield | airline | String | IATA operating carrier (2-char) | Amadeus AODB | CARRIER_IATA_CODE | DB_VIEW.FLIGHT_LEG |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | FQFC | String | Fully qualified flight code (carrier+number) | Amadeus AODB | derived_FQFC | CALCULATED concat(CARRIER_IATA_CODE\|\|MARKETING_FLIGHT_NUMBER) |  | none | Prefer derive at Splunk if Amadeus view omits | cim:airfield:aodb:amadeus |
| Airfield | departureOrArrival | String(1) | Leg direction D or A | Amadeus AODB | MOVEMENT_DIRECTION_IND | DB_VIEW.FLIGHT_LEG |  | none | Map ENUM to D/A | cim:airfield:aodb:amadeus |
| Airfield | aircraftParkingPosition | String | Stand / parking position | Amadeus AODB | AIRCRAFT_STAND_CODE | DB_VIEW.RESOURCE_ALLOCATION |  | none | May live allocation table vs leg core | cim:airfield:aodb:amadeus |
| Airfield | passengerGate | String | Public boarding gate | Amadeus AODB | BOARDING_GATE_CODE | DB_VIEW.GATE_ALLOCATION |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | remoteOperationalGate | String | Remote / bus gate adjunct | Amadeus AODB | REMOTE_GATE_CODE | DB_VIEW.GATE_ALLOCATION |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | runway | String | Runway-in-use for movement | Amadeus AODB | DEPARTURE_RUNWAY_ID or ARRIVAL_RUNWAY_ID | DB_VIEW.FLIGHT_MOVEMENT |  | tower_feed | Often finalized late; optional ASDE-X enrichment | cim:airfield:aodb:amadeus |
| Airfield | terminal | String | Passenger terminal | Amadeus AODB | TERMINAL_CODE | DB_VIEW.TERMINAL |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | paxBusInd | Boolean | Airside bus required | Amadeus AODB | BUS_TRANSFER_IND | DB_VIEW.FLIGHT_LEG |  | none | Normalize Y/N 1/0 to boolean | cim:airfield:aodb:amadeus |
| Airfield | registration | String | Aircraft tail registration | Amadeus AODB | AIRCRAFT_REGISTRATION | DB_VIEW.AIRCRAFT_ROTATION |  | airline_ACARS | Conflict policy: prefer post-pushback confirm source per SOCC | cim:airfield:aodb:amadeus |
| Airfield | agentInfo | String | Ground handling agent id | Amadeus AODB | HANDLING_AGENT_CODE | DB_VIEW.FLIGHT_SERVICE |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | paxCount | String or int | Final or actual pax count on leg | Amadeus AODB | PASSENGER_COUNT_FINAL | DB_VIEW.FLIGHT_LOAD |  | DCS_bridge | May arrive via load message integration | cim:airfield:aodb:amadeus |
| Airfield | destAirport | String(3) | Destination IATA (departure leg) | Amadeus AODB | ARRIVAL_STATION_IATA | DB_VIEW.FLIGHT_LEG |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | SOBT | int(10) | Scheduled off-block epoch | Amadeus AODB | SCHED_OFF_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | schedule_feed | Authority schedule baseline | cim:airfield:aodb:amadeus |
| Airfield | EOBT | int(10) | Estimated off-block epoch | Amadeus AODB | EST_OFF_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | prediction_engine |  | cim:airfield:aodb:amadeus |
| Airfield | TOBT | int(10) | Target off-block epoch | Amadeus AODB | TARGET_OFF_BLOCK_DTM | DB_VIEW.CDM_MILESTONE |  | CDM_module | After airline commitment window use CDM table if split | cim:airfield:aodb:amadeus |
| Airfield | COBT | int(10) | Calculated off-block epoch | Amadeus AODB | CALC_OFF_BLOCK_DTM | DB_VIEW.CDM_MILESTONE |  | CDM_module |  | cim:airfield:aodb:amadeus |
| Airfield | AOBT | int(10) | Actual off-block epoch | Amadeus AODB | ACT_OFF_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | ACARS | Define precedence vs airline message | cim:airfield:aodb:amadeus |
| Airfield | EGTO | int(10) | Estimated gate open epoch | Amadeus AODB | EST_GATE_OPEN_DTM | DB_VIEW.GATE_MILESTONE |  | gate_system |  | cim:airfield:aodb:amadeus |
| Airfield | AGTO | int(10) | Actual gate open epoch | Amadeus AODB | ACT_GATE_OPEN_DTM | DB_VIEW.GATE_MILESTONE |  | gate_system |  | cim:airfield:aodb:amadeus |
| Airfield | EBST | int(10) | Estimated boarding start epoch | Amadeus AODB | EST_BOARDING_START_DTM | DB_VIEW.BOARDING_MILESTONE |  | gate_agent_UI |  | cim:airfield:aodb:amadeus |
| Airfield | ABST | int(10) | Actual boarding start epoch | Amadeus AODB | ACT_BOARDING_START_DTM | DB_VIEW.BOARDING_MILESTONE |  | gate_agent_UI |  | cim:airfield:aodb:amadeus |
| Airfield | EGCL | int(10) | Estimated gate close epoch | Amadeus AODB | EST_GATE_CLOSE_DTM | DB_VIEW.GATE_MILESTONE |  | gate_agent_UI |  | cim:airfield:aodb:amadeus |
| Airfield | AGCL | int(10) | Actual gate close epoch | Amadeus AODB | ACT_GATE_CLOSE_DTM | DB_VIEW.GATE_MILESTONE |  | gate_agent_UI |  | cim:airfield:aodb:amadeus |
| Airfield | TSAT | int(10) | Target startup approval epoch | Amadeus AODB | TARGET_STARTUP_APPROVAL_DTM | DB_VIEW.CDM_MILESTONE |  | faa_surface_CDM | Native label may differ; alias to CIM TSAT | cim:airfield:aodb:amadeus |
| Airfield | ESAT | int(10) | Estimated startup approval epoch | Amadeus AODB | EST_STARTUP_APPROVAL_DTM | DB_VIEW.CDM_MILESTONE |  | faa_surface_CDM |  | cim:airfield:aodb:amadeus |
| Airfield | ASAT | int(10) | Actual startup approval epoch | Amadeus AODB | ACT_STARTUP_APPROVAL_DTM | DB_VIEW.CDM_MILESTONE |  | faa_surface_CDM |  | cim:airfield:aodb:amadeus |
| Airfield | ATOT | int(10) | Actual takeoff epoch | Amadeus AODB | ACT_TAKEOFF_DTM | DB_VIEW.FLIGHT_TIMES |  | SWIM_or_ACARS | Licensed NAS feeds require separate sourcetype | cim:airfield:aodb:amadeus |
| Airfield | CTOT | int(10) | Calculated takeoff epoch (CDM/NAS constraint slot) | Amadeus AODB | NAS_CTOT_DTM or CONSTRAINT_TAKEOFF_DTM | DB_VIEW.NAS_CONSTRAINT |  | TFMS | many US sites expose EDCT; map policy to CTOT or retain both native | cim:airfield:aodb:amadeus |
| Airfield | TTOT | int(10) | Target takeoff epoch | Amadeus AODB | TARGET_TAKEOFF_DTM | DB_VIEW.CDM_MILESTONE |  | CDM_module |  | cim:airfield:aodb:amadeus |
| Airfield | originAirport | String(3) | Origin IATA (arrival leg) | Amadeus AODB | DEPARTURE_STATION_IATA | DB_VIEW.FLIGHT_LEG |  | none |  | cim:airfield:aodb:amadeus |
| Airfield | SIBT | int(10) | Scheduled in-block epoch | Amadeus AODB | SCHED_ON_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | schedule_feed |  | cim:airfield:aodb:amadeus |
| Airfield | EIBT | int(10) | Estimated on-block epoch | Amadeus AODB | EST_ON_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | prediction_engine |  | cim:airfield:aodb:amadeus |
| Airfield | TIBT | int(10) | Target on-block epoch | Amadeus AODB | TARGET_ON_BLOCK_DTM | DB_VIEW.CDM_MILESTONE |  | CDM_module |  | cim:airfield:aodb:amadeus |
| Airfield | AIBT | int(10) | Actual on-block epoch | Amadeus AODB | ACT_ON_BLOCK_DTM | DB_VIEW.FLIGHT_TIMES |  | ACARS |  | cim:airfield:aodb:amadeus |
| Airfield | ETDN | int(10) | Estimated touchdown epoch | Amadeus AODB | EST_TOUCHDOWN_DTM | DB_VIEW.FLIGHT_MOVEMENT |  | ASDE_X_optional |  | cim:airfield:aodb:amadeus |
| Airfield | ETEN | int(10) | Estimated ten-mile epoch | Amadeus AODB | EST_10NM_DTM | DB_VIEW.FLIGHT_MOVEMENT |  | FDM_optional | Vendors use varying distance/fix definitions | cim:airfield:aodb:amadeus |
| Airfield | ATDN | int(10) | Actual touchdown epoch | Amadeus AODB | ACT_TOUCHDOWN_DTM | DB_VIEW.FLIGHT_MOVEMENT |  | ASDE_X_optional |  | cim:airfield:aodb:amadeus |
| Airfield | ATEN | int(10) | Actual ten-mile epoch | Amadeus AODB | ACT_10NM_DTM | DB_VIEW.FLIGHT_MOVEMENT |  | FDM_optional |  | cim:airfield:aodb:amadeus |
| Security | passengerName | String(<=20) | PSM format passenger name | CUPPS via Amadeus CHAIN or tenant broker | PSM_PASSENGER_NAME | INTERFACE.DCS_BOARDING |  | confidential_pii | Restrict indexes/RBAC; hash for aggregate KPIs | cim:security:cupps:amadeus |
| Security | from | String | Origin city/airport label | CUPPS via Amadeus CHAIN or tenant broker | ORIGIN_TEXT | INTERFACE.DCS_BOARDING |  | low |  | cim:security:cupps:amadeus |
| Security | to | String | Destination city/airport label | CUPPS via Amadeus CHAIN or tenant broker | DESTINATION_TEXT | INTERFACE.DCS_BOARDING |  | low |  | cim:security:cupps:amadeus |
| Security | airline | String | IATA carrier from boarding record | CUPPS via Amadeus CHAIN or tenant broker | CARRIER_IATA_CODE | INTERFACE.DCS_BOARDING |  | none | Cross-check AODB | cim:security:cupps:amadeus |
| Security | flightNumber | int(4) | Flight number from boarding record | CUPPS via Amadeus CHAIN or tenant broker | MARKETING_FLIGHT_NUMBER | INTERFACE.DCS_BOARDING |  | none |  | cim:security:cupps:amadeus |
| Security | FQFC | String | FQFC from boarding record | CUPPS via Amadeus CHAIN or tenant broker | derived_FQFC | CALCULATED |  | none |  | cim:security:cupps:amadeus |
| Security | seat | String | Seat assignment | CUPPS via Amadeus CHAIN or tenant broker | SEAT_NUMBER | INTERFACE.DCS_BOARDING |  | low |  | cim:security:cupps:amadeus |
| Security | passengerGate | String | Gate on boarding artifact | CUPPS via Amadeus CHAIN or tenant broker | GATE_CODE | INTERFACE.DCS_BOARDING |  | none | Prefer AODB gate for ops consistency | cim:security:cupps:amadeus |
| Security | boardTime | int(4) | Boarding time HHMM | CUPPS via Amadeus CHAIN or tenant broker | SCHED_BOARDING_TIME_HHMM | INTERFACE.DCS_BOARDING |  | none | Normalize to epoch if mixing types | cim:security:cupps:amadeus |
| Security | BPSOBT | String | BPS printed SOBT (no year) | CUPPS via Amadeus CHAIN or tenant broker | BPS_PRINTED_SOBT | INTERFACE.DCS_BOARDING |  | none | Join to flightUID for YoY | cim:security:cupps:amadeus |
| Security | class | int(1) | Travel class PADIS 9873 | CUPPS via Amadeus CHAIN or tenant broker | CABIN_CLASS_CODE | INTERFACE.DCS_BOARDING |  | none | Map RBD to PADIS if needed | cim:security:cupps:amadeus |
| Security | dateOfIssue | String | BSP issue date | CUPPS via Amadeus CHAIN or tenant broker | DATE_OF_ISSUE | INTERFACE.DCS_BOARDING |  | low |  | cim:security:cupps:amadeus |
| Security | WTMDID | int | Walk-through metal detector id | TSA-certified lane vendor | N/A | N/A — vendor SCADA |  | none | Separate sourcetype per OEM major version | cim:security:checkpoint:vendor |
| Security | passengerDirection | int(1) | WTMD travel direction | TSA-certified lane vendor | LANE_PASSENGER_DIRECTION | VENDOR_EVT |  | none |  | cim:security:checkpoint:vendor |
| Security | zoneAlarm | int(20) | Encoded zone alarm pattern | TSA-certified lane vendor | ZONE_ALARM_BITMAP | VENDOR_EVT |  | none | Document bit order per vendor | cim:security:checkpoint:vendor |
| Security | alarm | Boolean | Alarmed passage | TSA-certified lane vendor | ALARM_FLAG | VENDOR_EVT |  | none |  | cim:security:checkpoint:vendor |
| Security | randomAlarm | Boolean | Random alarm | TSA-certified lane vendor | RANDOM_ALARM_FLAG | VENDOR_EVT |  | none |  | cim:security:checkpoint:vendor |
| Security | lastUpdated | int(10) | Event time for WTMD record | TSA-certified lane vendor | EVENT_TIMESTAMP_UTC | VENDOR_EVT |  | internal | _time candidate | cim:security:checkpoint:vendor |
| Security | imageID | String | Cabin bag scan image id | TSA-certified lane vendor | SCAN_IMAGE_GUID | VENDOR_EVT |  | internal |  | cim:security:checkpoint:vendor |
| Security | decision | String | clear or reject | TSA-certified lane vendor | OPERATOR_DECISION | VENDOR_EVT |  | none |  | cim:security:checkpoint:vendor |
| Security | TIP | Boolean | Threat image projection flag | TSA-certified lane vendor | TIP_IND | VENDOR_EVT |  | none |  | cim:security:checkpoint:vendor |
| Security | lastUpdated | int(10) | Event time for X-ray record | TSA-certified lane vendor | EVENT_TIMESTAMP_UTC | VENDOR_EVT |  | internal | _time candidate | cim:security:checkpoint:vendor |
| Baggage | bagTag | int(10) | Bag tag number per IATA Res 751 | BHS + middleware | BAG_TAG_NUMBER | BHS.EVENT |  | none | Prefer middleware timestamped events | cim:baggage:bhs:vendor |
| Baggage | passengerName | String(<=20) | PSM name on bag message | BHS + middleware | PSM_PASSENGER_NAME | BHS.BSM |  | confidential_pii |  | cim:baggage:bhs:vendor |
| Baggage | flightUID | String | Leg join key | BHS + middleware | FLIGHT_LEG_OID | BHS.SORT_PLAN |  | internal | Must match Amadeus FLIGHT_LEG_OID mapping | cim:baggage:bhs:vendor |
| Baggage | ULDID | String | Unit load device id | BHS + middleware | ULD_IDENTIFIER | BHS.ULD |  | none |  | cim:baggage:bhs:vendor |
| Baggage | SOBT | int(10) | Scheduled off-block for sort | BHS + middleware | SCHED_OFF_BLOCK_DTM | BHS.SORT_PLAN |  | Amadeus_AODB | Duplicate semantic field; align clock with Amadeus | cim:baggage:bhs:vendor |
| Baggage | FQFC | String | FQFC for sort | BHS + middleware | derived_FQFC | BHS.SORT_PLAN |  | none |  | cim:baggage:bhs:vendor |
| Baggage | messageType | String | BSM BPM BUM etc. | Type B middleware | MESSAGE_TYPE_ID | GATEWAY.TYPE_B |  | none | IATA Type B via SITA/ARINC | cim:baggage:sita_or_arinc |
| Baggage | messageAction | String | NEW UPT DEL | Type B middleware | MESSAGE_ACTION_CD | GATEWAY.TYPE_B |  | none |  | cim:baggage:sita_or_arinc |
| Baggage | level | int(1) | Security screening level | BHS + CBIS | SCREENING_LEVEL_NBR | BHS.SECURITY_ROUTING |  | none | Airport lookup maps tier numbers | cim:baggage:bhs:vendor |
| Baggage | decision | String | clear or reject | BHS + CBIS | SCREENING_DECISION | BHS.SECURITY_ROUTING |  | none |  | cim:baggage:bhs:vendor |
| Baggage | previousLevel | int(1) | Prior screening level | BHS + CBIS | PREV_SCREENING_LEVEL_NBR | BHS.SECURITY_ROUTING |  | none |  | cim:baggage:bhs:vendor |
| Baggage | nextLevel | int(1) | Next screening level | BHS + CBIS | NEXT_SCREENING_LEVEL_NBR | BHS.SECURITY_ROUTING |  | none |  | cim:baggage:bhs:vendor |
| GroundOps | type1.start | int(10) | Type I fluid start epoch | Deice pad vendor | N/A | N/A — pad system |  | internal |  | cim:groundops:deice:vendor |
| GroundOps | type1.stop | int(10) | Type I fluid stop epoch | Deice pad vendor | N/A | N/A — pad system |  | internal |  | cim:groundops:deice:vendor |
| GroundOps | type4.start | int(10) | Type IV fluid start epoch | Deice pad vendor | N/A | N/A — pad system |  | internal |  | cim:groundops:deice:vendor |
| GroundOps | type4.stop | int(10) | Type IV fluid stop epoch | Deice pad vendor | N/A | N/A — pad system |  | internal |  | cim:groundops:deice:vendor |
| GroundOps | registration | String | Tail number linkage | Deice pad vendor | AIRCRAFT_REGISTRATION | PAD.LOG |  | none | Should match Airfield.registration policy | cim:groundops:deice:vendor |
| GroundOps | FQFC | String | Flight linkage | Deice pad vendor | FULL_FLIGHT_CODE | PAD.LOG |  | none |  | cim:groundops:deice:vendor |
| GroundOps | airportId | String | Station ICAO/IATA | Deice pad vendor | AIRPORT_IATA_OR_ICAO | PAD.LOG |  | none |  | cim:groundops:deice:vendor |
| GroundOps | padId | String | Deice pad identifier | Deice pad vendor | PAD_LOCATION_ID | PAD.LOG |  | none |  | cim:groundops:deice:vendor |
| GroundOps | eventType | String | Service type performed | Deice pad vendor | SERVICE_TYPE | PAD.LOG |  | none | e.g. De-icing Anti-icing | cim:groundops:deice:vendor |
| ParkingEvents | eventTime | int(10) | Parking event epoch | PARCS | TRANSACTION_TIMESTAMP_UTC | PARK.GATE_EVT |  | internal | _time candidate | cim:parking:parcs:vendor |
| ParkingEvents | vehicleId | String | Plate or token | PARCS | VEHICLE_TOKEN_ID | PARK.GATE_EVT |  | internal |  | cim:parking:parcs:vendor |
| ParkingEvents | eventType | String | pre-book rollup valet taxi | PARCS | EVENT_TYPE_CD | PARK.GATE_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingEvents | eventOutcome | String | success fail | PARCS | EVENT_OUTCOME_CD | PARK.GATE_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingEvents | gateId | String | Device id | PARCS | GATE_DEVICE_ID | PARK.GATE_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingEvents | gateLocation | String | Facility location text | PARCS | GATE_LOCATION_DESC | PARK.GATE_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingEvents | gateType | String | entrance exit valet | PARCS | GATE_USAGE_TYPE | PARK.GATE_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingFaults | eventTime | int(10) | Fault event epoch | PARCS | FAULT_TIMESTAMP_UTC | PARK.FAULT_EVT |  | internal |  | cim:parking:parcs:vendor |
| ParkingFaults | eventType | String | motor failure paper outage | PARCS | FAULT_TYPE_CD | PARK.FAULT_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingFaults | eventStatus | int(1) | Active fault flag | PARCS | FAULT_ACTIVE_IND | PARK.FAULT_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingFaults | gateId | String | Device id | PARCS | GATE_DEVICE_ID | PARK.FAULT_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingFaults | gateLocation | String | Facility location | PARCS | GATE_LOCATION_DESC | PARK.FAULT_EVT |  | none |  | cim:parking:parcs:vendor |
| ParkingFaults | gateType | String | entrance exit valet | PARCS | GATE_USAGE_TYPE | PARK.FAULT_EVT |  | none |  | cim:parking:parcs:vendor |
