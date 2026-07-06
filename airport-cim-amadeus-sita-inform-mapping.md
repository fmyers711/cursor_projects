# Airport CIM field mapping: Amadeus, SITA, and INFORM

This document is generated from [`airport-cim-amadeus-sita-inform-mapping.csv`](airport-cim-amadeus-sita-inform-mapping.csv). It maps vendor-native identifiers to a **common canonical naming convention** aligned with the [Airports Common Information Model for Splunk](https://splunk.github.io/airport_cim_for_splunk/) style: camelCase with acronyms in capitals (`SOBT`, `FQFC`, `TSAT`, etc.).

**Disclaimer:** Native field names are representative placeholders. Validate every mapping against your tenant’s Amadeus, SITA, and INFORM data dictionaries and interface contracts before production ingest.

---

## Convention summary

| Column (CSV) | Meaning |
|--------------|---------|
| **cim_canonical_field** | Target Splunk/CIM field name after normalization. |
| **amadeus_native_field** | Typical Amadeus Airport IT / AODB-style export name. |
| **sita_native_field** | Typical SITA airport operations / flight API style name. |
| **inform_native_field** | Typical INFORM GroundStar-style export name. |
| **harmonization_notes** | Timezone, precedence, joins, and edge-case handling. |
| **pii_class** | Data sensitivity hint (`none`, `low`, `internal`, `confidential_pii`, etc.). |

---

## Airfield — Core

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| lastUpdated | int(10) | Record last update epoch UTC | LAST_UPDATE_TS | flightLegLastModifiedUtc | GS_EVT_LAST_UPDATE_TS | Normalize all vendors to epoch UTC at ingest; use for Splunk `_time` when policy allows. | internal |
| flightUID | String | Stable flight leg identifier | FLIGHT_LEG_OID | flightLegUuid | GS_FLIGHT_LEG_KEY | Establish cross-vendor correlation table keyed by airline-date-flight-sequence if IDs differ. | internal |
| flightNumber | int(4) | Marketing flight number | MARKETING_FLIGHT_NUMBER | flightNumber | FLT_NBR | Treat alphanumeric codes per airline rules before casting int where unsafe. | none |
| serviceType | String(1) | Service type sched charter etc | SERVICE_TYPE_CODE | serviceTypeCode | SVC_TYP_CD | Map via lookup to CIM serviceTypes enumeration. | none |
| airline | String | IATA carrier 2-char | CARRIER_IATA_CODE | airlineDesignator | CARRIER_IATA | Uppercase strip spaces. | none |
| FQFC | String | Carrier plus flight number | derived_FQFC | derived_fullFlightDesignator | derived_FQFC | Prefer derive at normalization layer for consistency across vendors. | none |
| departureOrArrival | String(1) | D or A leg direction | MOVEMENT_DIRECTION_IND | movementIndicator | ARR_DEP_IND | Map DEP ARR enums to single-char D A. | none |
| aircraftParkingPosition | String | Stand hardstand code | AIRCRAFT_STAND_CODE | standPosition.currentCode | PARK_POS_CD | INFORM allocation may lag Amadeus publish; define precedence. | none |
| passengerGate | String | Boarding gate | BOARDING_GATE_CODE | gateAssignment.boardingGate | GATE_ID | Remote gates may use suffix conventions document mapping. | none |
| remoteOperationalGate | String | Bus remote gate adjunct | REMOTE_GATE_CODE | gateAssignment.remoteGate | REMOTE_GATE_CD | Often null when not remote operation. | none |
| runway | String | Runway designation | DEPARTURE_RUNWAY_ID or ARRIVAL_RUNWAY_ID | runwayInUse.designator | RWY_DESIG | Tower or surveillance feed may override static AODB publish document precedence. | tower_feed |
| terminal | String | Terminal code | TERMINAL_CODE | terminalCode | TERM_CD | Map numeric vs alpha terminal ids via lookup. | none |
| paxBusInd | Boolean | Airside bus indicator | BUS_TRANSFER_IND | busTransferRequired | pax_bus_ind | Normalize Y N 1 0 true false to boolean. | none |
| registration | String | Tail number | AIRCRAFT_REGISTRATION | aircraftRegistration | AC_REG | Align tail updates with ACARS policy table. | none |
| agentInfo | String | Handling agent code | HANDLING_AGENT_CODE | handlingAgentCode | GH_AGENT_CD | Multi-agent flights may need array collapse rule at ingest. | none |
| paxCount | String or int | Passenger count final | PASSENGER_COUNT_FINAL | passengerCount.final | pax_cnt_final | May arrive only after gate close; type coerce consistently. | none |

---

## Airfield — Departures

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| destAirport | String(3) | Destination IATA | ARRIVAL_STATION_IATA | destinationAirport.iataCode | DEST_AP_IATA | Departure-leg semantics only. | none |
| SOBT | int(10) | Scheduled off-block epoch | SCHED_OFF_BLOCK_DTM | scheduledOffBlockTimeUtc | SOBT_UTC | Parse ISO-8601 from SITA APIs to epoch. | none |
| EOBT | int(10) | Estimated off-block epoch | EST_OFF_BLOCK_DTM | estimatedOffBlockTimeUtc | EOBT_UTC | Vendor prediction refresh cadence differs document staleness. | none |
| TOBT | int(10) | Target off-block epoch | TARGET_OFF_BLOCK_DTM | targetOffBlockTimeUtc | TOBT_UTC | CDM commitment windows may live only on one system define master. | CDM_module |
| COBT | int(10) | Calculated off-block epoch | CALC_OFF_BLOCK_DTM | calculatedOffBlockTimeUtc | COBT_UTC | Alias COBT from CDM milestone export not prediction table. | CDM_module |
| AOBT | int(10) | Actual off-block epoch | ACT_OFF_BLOCK_DTM | actualOffBlockTimeUtc | AOBT_UTC | Precedence ACARS vs AODB vs INFORM mobile capture. | ACARS |
| EGTO | int(10) | Estimated gate open epoch | EST_GATE_OPEN_DTM | estimatedGateOpenUtc | gate_open_est_utc | Gate milestones may be INFORM-only if airport uses GS Gate Bridge. | none |
| AGTO | int(10) | Actual gate open epoch | ACT_GATE_OPEN_DTM | actualGateOpenUtc | gate_open_act_utc | — | none |
| EBST | int(10) | Estimated boarding start epoch | EST_BOARDING_START_DTM | estimatedBoardingStartUtc | brd_start_est_utc | — | none |
| ABST | int(10) | Actual boarding start epoch | ACT_BOARDING_START_DTM | actualBoardingStartUtc | brd_start_act_utc | — | none |
| EGCL | int(10) | Estimated gate close epoch | EST_GATE_CLOSE_DTM | estimatedGateCloseUtc | gate_close_est_utc | — | none |
| AGCL | int(10) | Actual gate close epoch | ACT_GATE_CLOSE_DTM | actualGateCloseUtc | gate_close_act_utc | — | none |
| TSAT | int(10) | Target startup approval epoch | TARGET_STARTUP_APPROVAL_DTM | targetStartupApprovalUtc | TSAT_UTC | US Surface CDM naming differs map vendor startup approval fields here. | faa_surface_CDM |
| ESAT | int(10) | Estimated startup approval epoch | EST_STARTUP_APPROVAL_DTM | estimatedStartupApprovalUtc | ESAT_UTC | — | faa_surface_CDM |
| ASAT | int(10) | Actual startup approval epoch | ACT_STARTUP_APPROVAL_DTM | actualStartupApprovalUtc | ASAT_UTC | — | faa_surface_CDM |
| ATOT | int(10) | Actual takeoff epoch | ACT_TAKEOFF_DTM | actualTakeOffTimeUtc | ATOT_UTC | Licensed NAS feeds may supply authoritative ATOT retain native sourcetype. | SWIM_or_ACARS |
| CTOT | int(10) | Calculated constraint takeoff epoch | NAS_CTOT_DTM or CONSTRAINT_TAKEOFF_DTM | calculatedTakeOffTimeUtc | CTOT_UTC | US EDCT vs CTOT retain native nas_edct_utc if policy differs. | TFMS |
| TTOT | int(10) | Target takeoff epoch | TARGET_TAKEOFF_DTM | targetTakeOffTimeUtc | TTOT_UTC | — | CDM_module |

---

## Airfield — Arrivals

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| originAirport | String(3) | Origin IATA | DEPARTURE_STATION_IATA | originAirport.iataCode | ORIG_AP_IATA | Arrival-leg semantics only. | none |
| SIBT | int(10) | Scheduled on-block epoch | SCHED_ON_BLOCK_DTM | scheduledOnBlockTimeUtc | SIBT_UTC | — | none |
| EIBT | int(10) | Estimated on-block epoch | EST_ON_BLOCK_DTM | estimatedOnBlockTimeUtc | EIBT_UTC | — | none |
| TIBT | int(10) | Target on-block epoch | TARGET_ON_BLOCK_DTM | targetOnBlockTimeUtc | TIBT_UTC | — | CDM_module |
| AIBT | int(10) | Actual on-block epoch | ACT_ON_BLOCK_DTM | actualOnBlockTimeUtc | AIBT_UTC | — | ACARS |
| ETDN | int(10) | Estimated touchdown epoch | EST_TOUCHDOWN_DTM | estimatedTouchdownUtc | tdn_est_utc | Surveillance enrichment optional. | ASDE_X_optional |
| ETEN | int(10) | Estimated ten-mile epoch | EST_10NM_DTM | estimatedTenMileUtc | eten_est_utc | Define fix distance consistently across feeds. | FDM_optional |
| ATDN | int(10) | Actual touchdown epoch | ACT_TOUCHDOWN_DTM | actualTouchdownUtc | tdn_act_utc | — | ASDE_X_optional |
| ATEN | int(10) | Actual ten-mile epoch | ACT_10NM_DTM | actualTenMileUtc | aten_act_utc | — | FDM_optional |

---

## Security — Passenger PSRM

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| passengerName | String(<=20) | PSM passenger name | PSM_PASSENGER_NAME | passengerNamePsm | PSM_NAME | CUPPS vs SITA Airport Connect vs INFORM pax modules tenant dependent. | confidential_pii |
| from | String | Origin label | ORIGIN_TEXT | originDescription | ORIG_TXT | — | low |
| to | String | Destination label | DESTINATION_TEXT | destinationDescription | DEST_TXT | — | low |
| airline | String | IATA carrier | CARRIER_IATA_CODE | carrierIata | CARRIER_IATA | Cross-check against Airfield.airline. | none |
| flightNumber | int(4) | Flight number | MARKETING_FLIGHT_NUMBER | flightNumber | FLT_NBR | — | none |
| FQFC | String | FQFC | derived_FQFC | derived_fullFlightDesignator | derived_FQFC | — | none |
| seat | String | Seat | SEAT_NUMBER | seatDesignator | SEAT_CD | — | low |
| passengerGate | String | Gate on BP | GATE_CODE | gateCode | GATE_CD | Prefer Airfield.passengerGate for ops KPI unless BP scan is source of truth. | none |
| boardTime | int(4) | Boarding HHMM | SCHED_BOARDING_TIME_HHMM | scheduledBoardingLocal | BRD_TM_HHMM | Normalize HHMM int vs epoch per single convention. | none |
| BPSOBT | String | BPS printed SOBT | BPS_PRINTED_SOBT | boardingPassPrintedSobt | BPS_SOBT_STR | Join via flightUID year absent in print artifact. | none |
| class | int(1) | PADIS class | CABIN_CLASS_CODE | cabinClassPadis | CLS_CD_PADIS | RBD to PADIS lookup may be required. | none |
| dateOfIssue | String | BSP issue date | DATE_OF_ISSUE | dateOfIssue | dOI | — | low |

---

## Security — Checkpoint WTMD

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| WTMDID | int | Detector id | N/A | deviceLaneId | GS_SEC_DEVICE_ID | Vendor-specific checkpoint telemetry not from AODB vendors. | none |
| passengerDirection | int(1) | Pass direction | N/A | passengerWalkDirection | walk_dir_ind | — | none |
| zoneAlarm | int(20) | Zone bitmap | N/A | alarmZoneBitmap | zone_alarm_mask | Document bit-endian per OEM. | none |
| alarm | Boolean | Alarm flag | N/A | isAlarm | alarm_flg | — | none |
| randomAlarm | Boolean | Random alarm | N/A | isRandomAlarm | random_alarm_flg | — | none |
| lastUpdated | int(10) | Event epoch | N/A | eventTimestampUtc | evt_ts_utc | `_time` candidate for checkpoint stream. | internal |

---

## Security — Checkpoint X-ray

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| imageID | String | Scan image id | N/A | scanImageGuid | xray_img_guid | — | internal |
| decision | String | clear reject | N/A | operatorDecision | xray_decision_cd | — | none |
| TIP | Boolean | TIP flag | N/A | threatImageProjection | tip_ind | — | none |
| lastUpdated | int(10) | Event epoch | N/A | eventTimestampUtc | evt_ts_utc | `_time` candidate for checkpoint stream. | internal |

---

## Baggage — Hold

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| bagTag | int(10) | Bag tag number | BAG_TAG_NUMBER | bagTagNumber | LPN_TAG_NBR | BHS vendor export names vary normalize leading zeros. | none |
| passengerName | String(<=20) | PSM on bag msg | PSM_PASSENGER_NAME | passengerNamePsm | BPM_PSM_NAME | SITA Type B gateway element names vary by parser version. | confidential_pii |
| flightUID | String | Leg join to Airfield | FLIGHT_LEG_OID | flightLegRefUuid | GS_FLIGHT_LEG_KEY | Must align with Airfield.flightUID correlation table. | internal |
| ULDID | String | ULD id | ULD_IDENTIFIER | uldReference | ULD_REF | — | none |
| SOBT | int(10) | SOBT for sort context | SCHED_OFF_BLOCK_DTM | scheduledOffBlockTimeUtc | SOBT_UTC | Clock sync between BHS and AODB required. | none |
| FQFC | String | FQFC for sort | derived_FQFC | derived_fullFlightDesignator | derived_FQFC | — | none |
| messageType | String | BSM BPM BUM | MESSAGE_TYPE_ID | typeBMessageType | MSG_TYP_CD | SITA gateway primary path for Type B at many hubs. | none |
| messageAction | String | NEW UPT DEL | MESSAGE_ACTION_CD | typeBAction | MSG_ACT_CD | — | none |
| level | int(1) | Screening level | SCREENING_LEVEL_NBR | securityScreeningLevel | SCR_LVL_NBR | Airport tier lookup required. | none |
| decision | String | clear reject | SCREENING_DECISION | securityDecision | SCR_DEC_CD | — | none |
| previousLevel | int(1) | Prior level | PREV_SCREENING_LEVEL_NBR | previousScreeningLevel | PREV_SCR_LVL | — | none |
| nextLevel | int(1) | Next level | NEXT_SCREENING_LEVEL_NBR | nextScreeningLevel | NEXT_SCR_LVL | — | none |

---

## GroundOps — Deice

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| type1.start | int(10) | Type I start epoch | N/A | N/A | N/A | Pad vendor systems map separately to CIM GroundOps. | internal |
| type1.stop | int(10) | Type I stop epoch | N/A | N/A | N/A | — | internal |
| type4.start | int(10) | Type IV start epoch | N/A | N/A | N/A | — | internal |
| type4.stop | int(10) | Type IV stop epoch | N/A | N/A | N/A | — | internal |
| registration | String | Tail | AIRCRAFT_REGISTRATION | aircraftRegistration | AC_REG | Same harmonization as Airfield.registration. | none |
| FQFC | String | Flight code | FULL_FLIGHT_CODE | derived_fullFlightDesignator | derived_FQFC | — | none |
| airportId | String | Station | IATA_OR_ICAO_AT_PAD | airportIataOrIcao | ARPT_CD | — | none |
| padId | String | Deice pad | N/A | N/A | GS_DEICE_PAD_ID | INFORM pad resource id when airport uses GS Ground Ops module for deice. | none |
| eventType | String | Service type | N/A | N/A | GS_DEICE_SVC_TYP | — | none |

---

## Parking — Events (PARCS)

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| eventTime | int(10) | Event epoch | TRANSACTION_TIMESTAMP_UTC | N/A | N/A | PARCS vendor neutral fields align to ParkingEvents CIM. | internal |
| vehicleId | String | Token plate | VEHICLE_TOKEN_ID | N/A | N/A | — | internal |
| eventType | String | Event kind | EVENT_TYPE_CD | N/A | N/A | — | none |
| eventOutcome | String | success fail | EVENT_OUTCOME_CD | N/A | N/A | — | none |
| gateId | String | Device id | GATE_DEVICE_ID | N/A | N/A | — | none |
| gateLocation | String | Location text | GATE_LOCATION_DESC | N/A | N/A | — | none |
| gateType | String | Usage type | GATE_USAGE_TYPE | N/A | N/A | — | none |

---

## Parking — Faults (PARCS)

| Canonical field | Type | Description | Amadeus | SITA | INFORM | Harmonization notes | PII |
|-----------------|------|-------------|---------|------|--------|---------------------|-----|
| eventTime | int(10) | Fault epoch | FAULT_TIMESTAMP_UTC | N/A | N/A | — | internal |
| eventType | String | Fault type | FAULT_TYPE_CD | N/A | N/A | — | none |
| eventStatus | int(1) | Active flag | FAULT_ACTIVE_IND | N/A | N/A | — | none |
| gateId | String | Device id | GATE_DEVICE_ID | N/A | N/A | — | none |
| gateLocation | String | Location | GATE_LOCATION_DESC | N/A | N/A | — | none |
| gateType | String | Usage type | GATE_USAGE_TYPE | N/A | N/A | — | none |

---

## Source file

Row-level edits should be made in **`airport-cim-amadeus-sita-inform-mapping.csv`** and this Markdown file regenerated or merged as part of your documentation pipeline.
