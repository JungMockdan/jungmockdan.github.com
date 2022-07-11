---
title: "OCPI 2.1.1 implementation as eMSP"
date: 2022-07-11 09:55:28 -0400
categories: work
tags: ocpi ev


# 목차
toc: true  
toc_sticky: true 
---

# OCPI 2.1.1 implementation as eMSP

## implementation scope
- location module
- CPO : SHELL - Greenlot

## 1. implementations

### 1.1. location module
- endpoint structure : /ocpi/cpo/211/locations
- 제공된 URL 세그먼트에 따라 GET 요청을 사용하여 이 CPO에서 사용 가능한 위치 및 EVSE 목록(GET List)에 대한 정보를 검색하거나 특정 위치, EVSE 또는 커넥터에 대한 정보(GET Object)를 가져올 수 있습니다.
#### 1.1.1. method : GET
##### GET List
###### Example endpoints
> /ocpi/cpo/2.0/locations/?date_from=xxx&date_to=yyy
> /ocpi/cpo/2.0/locations/?offset=50
> /ocpi/cpo/2.0/locations/?limit=100
> /ocpi/cpo/2.0/locations/?offset=50&limit=100

- List Request Parameters : **QueryString**

    | param     |     type | mandatory | desc                                                                |
    | :-------- | -------: | :-------: | :------------------------------------------------------------------ |
    | date_from | DateTime |    no     | Only return Locations that have last_updated after this Date/Time.  |
    | date_to   | DateTime |    no     | Only return Locations that have last_updated before this Date/Time. |
    | offset    |      int |    no     | The offset of the first object returned. Default is 0.              |
    | limit     |      int |    no     | Maximum number of objects to GET.                                   |

- List Response Data :

    |                      |             | *Choice: one of three                                 |
    | :------------------- | :---------: | :---------------------------------------------------- |
    | Type                 | Cardinality | Description                                           |
    | -------------------- | ----------- | ----------------------------------------------------- |
    | Location             |      1      | New Location object, or Location object to replace.   |    

##### GET Object
###### Example endpoints
> /ocpi/cpo/2.0/locations/{location_id}
> /ocpi/cpo/2.0/locations/{location_id}/{evse_uid}
> /ocpi/cpo/2.0/locations/{location_id}/{evse_uid}/{connector_id}

- Object Request Parameters : **PathVariable**

    | param     |     type | mandatory | desc                                                                |
    | :-------- | -------: | :-------: | :------------------------------------------------------------------ |
    | location_id | string(39) |    yes     | Only return Locations that have last_updated after this Date/Time.  |
    | evse_uid   | string(39) |    no     | Only return Locations that have last_updated before this Date/Time. |
    | connector_id    |      string(36) |    no     | The offset of the first object returned. Default is 0.              |

- Object Response Data :
  
      |                      |             | *Choice: one of three                                 |
      | :------------------- | :---------: | :---------------------------------------------------- |
      | Type                 | Cardinality | Description                                           |
      | -------------------- | ----------- | ----------------------------------------------------- |
      | Location             |      1      | New Location object, or Location object to replace.   |
      | EVSE                 |      1      | New EVSE object, or EVSE object to replace.           |
      | Connector            |      1      | New Connector object, or Connector object to replace. |
    
    - sample response
        ```json
        {
        "id": "LOC1",
        "type": "ON_STREET",
        "name": "Gent Zuid",
        "address": "F.Rooseveltlaan 3A",
        "city": "Gent",
        "postal_code": "9000",
        "country": "BEL",
        "coordinates": {
            "latitude": "51.04759",
            "longitude": "3.72994"
        },
        "evses": [
            {
            "uid": "3256",
            "id": "BE-BEC-E041503001",
            "status": "AVAILABLE",
            "status_schedule": [],
            "capabilities": [
                "RESERVABLE"
            ],
            "connectors": [
                {
                "id": "1",
                "standard": "IEC_62196_T2",
                "format": "CABLE",
                "power_type": "AC_3_PHASE",
                "voltage": 220,
                "amperage": 16,
                "tariff_id": "11",
                "last_updated": "2015-03-16T10:10:02Z"
                },
                {
                "id": "2",
                "standard": "IEC_62196_T2",
                "format": "SOCKET",
                "power_type": "AC_3_PHASE",
                "voltage": 220,
                "amperage": 16,
                "tariff_id": "11",
                "last_updated": "2015-03-18T08:12:01Z"
                }
            ],
            "physical_reference": "1",
            "floor_level": "-1",
            "last_updated": "2015-06-28T08:12:01Z"
            },
            {
            "uid": "3257",
            "id": "BE-BEC-E041503002",
            "status": "RESERVED",
            "capabilities": [
                "RESERVABLE"
            ],
            "connectors": [
                {
                "id": "1",
                "status": "RESERVED",
                "standard": "IEC_62196_T2",
                "format": "SOCKET",
                "power_type": "AC_3_PHASE",
                "voltage": 220,
                "amperage": 16,
                "tariff_id": "12"
                }
            ],
            "physical_reference": "2",
            "floor_level": "-2",
            "last_updated": "2015-06-29T20:39:09Z"
            }
        ],
        "operator": {
            "name": "BeCharged"
        },
        "last_updated": "2015-06-29T20:39:09Z"
        }

        ```

----

## 2. Object description
### 2.1 Location Object
- Location 개체는 함께 속한 EVSE 그룹이 설치된 위치와 속성을 설명합니다. 일반적으로 Location 개체는 EVSE 그룹의 정확한 위치이지만 이러한 EVSE가 포함된 주차장 입구일 수도 있습니다. 각 EVSE에 도달하는 정확한 방법은 자체 속성에 의해 추가로 지정할 수 있습니다.
  
 | Property             | Type                  | Cardinality | Description                                                                                                                                                                                                    |
 | :------------------- | :-------------------- | :---------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
 | id                   | string_39             |      1      | Uniquely identifies the location within the CPOs platform (and suboperator platforms). This field can never be changed, modified or renamed.                                                                   |
 | type                 | LocationType          |      1      | The general type of the charge pointlocation.                                                                                                                                                                  |
 | name                 | string_255            |      ?      | Display name of the location.                                                                                                                                                                                  |
 | address              | string_45             |      1      | address string(45) 1 Street/block name and house number if available.                                                                                                                                          |
 | city                 | string_45             |             | City or town.                                                                                                                                                                                                  |
 | postal_code          | string_10             |      1      | Postal code of the location.                                                                                                                                                                                   |
 | country              | string_3              |      1      | ISO 3166-1 alpha-3 code for the country of this location.                                                                                                                                                      |
 | coordinates          | GeoLocation           |      1      | Coordinates of the location.                                                                                                                                                                                   |
 | related_locations    | AdditionalGeoLocation |      *      | Geographical location of related points relevant to the user.                                                                                                                                                  |
 | evses                | EVSE                  |      *      | List of EVSEs that belong to this Location.                                                                                                                                                                    |
 | directions           | DisplayText           |      *      | Human-readable directions on how to reach the location.                                                                                                                                                        |
 | operator             | BusinessDetails       |      ?      | Information of the operator. When not specified, the information retrieved from the api_info endpoint should be used instead.                                                                                  |
 | suboperator          | BusinessDetails       |      ?      | Information of the suboperator if available.                                                                                                                                                                   |
 | owner                | BusinessDetails       |      ?      | Information of the owner if available.                                                                                                                                                                         |
 | facilities           | Facility              |      *      | Optional list of facilities this charge location directly belongs to.                                                                                                                                          |
 | time_zone            | string_255            |      ?      | One of IANA tzdata’s TZ-values representing the time zone of the location. Examples: “Europe/Oslo”, “Europe/Zurich”. (http://www.iana.org/time-zones)                                                          |
 | opening_times        | Hours                 |      ?      | The times when the EVSEs at the location can be accessed for charging.                                                                                                                                         |
 | charging_when_closed | boolean               |      ?      | Indicates if the EVSEs are still charging outside the opening hours of the location. E.g. when the parking garage closes its barriers over night, is it allowed to charge till the next morning? Default: true |
 | images               | Image                 |      *      | Links to images related to the location such as photos or logos.                                                                                                                                               |
 | energy_mix           | EnergyMix             |      ?      | Details on the energy supplied at this location.                                                                                                                                                               |
 | last_updated         | DateTime              |      1      | Timestamp when this Location or one of its EVSEs or Connectors were last updated (or created).                                                                                                                 |

### 2.2 EVSE Object
- EVSE 객체는 단일 세션에서 단일 EV에 대한 전원 공급을 제어하는 부분을 설명합니다. 항상 Location 개체에 속합니다. 위치에서 EVSE로 가는 길찾기(예: 층, 물리적 참조 또는 길찾기)만 포함됩니다. 이러한 속성이 위치 지점에서 EVSE에 도달하는 데 충분하지 않은 경우 일반적으로 이 EVSE를 다른 위치 개체에 배치해야 함을 나타냅니다(때로는 주소는 같지만 좌표/방향이 다름). EVSE 개체에는 동시에 사용할 수 없는 커넥터 목록이 있습니다. 한 번에 EVSE당 하나의 커넥터만 사용할 수 있습니다.

| Property             | Type               | Cardinality | Description                                                                                                                                                                                                                                                                                                                         |
| :------------------- | :----------------- | :---------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| uid                  | string_39          |      1      | Uniquely identifies the EVSE within the CPOs platform (and suboperator platforms). For example a database unique ID or the “EVSE ID”. This field can never be changed, modified or renamed. This is the ‘technical’ identification of the EVSE, not to be used as ‘human readable’ identification, use the field: evse_id for that. |
| evse_id              | string_48          |      ?      | Compliant with the following specification for EVSE ID from “eMI3 standard version V1.0” (http: //emi3group.com/documents-links/) “Part 2: business objects.” Optional because: if an EVSE ID is to be re-used the EVSE ID can be removed from an EVSE that is removed (status: REMOVED)                                            |
| status               | Status             |      1      | Indicates the current status of the EVSE.                                                                                                                                                                                                                                                                                           |
| status_schedule      | StatusSchedule     |      *      | Indicates a planned status in the future of the EVSE.                                                                                                                                                                                                                                                                               |
| capabilities         | Capability         |      *      | List of functionalities that the EVSE is capable of.                                                                                                                                                                                                                                                                                |
| connectors           | Connector          |      +      | List of available connectors on the EVSE.                                                                                                                                                                                                                                                                                           |
| floor_level          | string_4           |      ?      | Level on which the charging station is located (in garage buildings) in the locally displayed numbering scheme.                                                                                                                                                                                                                     |
| coordinates          | GeoLocation        |      ?      | Coordinates of the EVSE.                                                                                                                                                                                                                                                                                                            |
| physical_reference   | string_16          |      ?      | A number/string printed_o the outside of the EVSE for visual identification.                                                                                                                                                                                                                                                        |
| directions           | DisplayText        |      *      | Multi-language human-readable directions when more detailed information on how to reach the EVSE from the Location is required.                                                                                                                                                                                                     |
| parking_restrictions | ParkingRestriction |      *      | The restrictions that apply to the parking spot.                                                                                                                                                                                                                                                                                    |
| images               | Image              |      *      | Links to images related to the EVSE such as photos or logos.                                                                                                                                                                                                                                                                        |
| last_updated         | DateTime           |      1      | Timestamp when this EVSE or one of its Connectors was last updated (or created).                                                                                                                                                                                                                                                    |

### 2.3 Connector Object
- 커넥터는 EV에서 사용할 수 있는 소켓 또는 케이블입니다. 단일 EVSE는 여러 커넥터를 제공할 수 있지만 그 중 하나만 동시에 사용할 수 있습니다. 커넥터는 항상 EVSE 개체에 속합니다.

| Property             | Type            | Cardinality | Description                                                                                                                                                  |
| :------------------- | :-------------- | :---------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                   | string_36       |      1      | Identifier of the connector within the EVSE. Two connectors may have the same id as long as they do not belong to the same EVSE object.                      |
| standard             | ConnectorType   |      1      | The standard of the installed connector.                                                                                                                     |
| format               | ConnectorFormat |      1      | The format (socket/cable) of the installed connector.                                                                                                        |
| power_type           | PowerType       |      1      |                                                                                                                                                              |
| voltage              | int             |      1      | Voltage of the connector (line to neutral for AC_3_PHASE), in volt [V].                                                                                      |
| amperage             | int             |      1      | maximum amperage of the connector, in ampere [A].                                                                                                            |
| tariff_id            | string_36       |      ?      | Identifier of the current charging tariff structure. For a “Free of Charge” tariff this field should be set, and point to a defined “Free of Charge” tariff. |
| terms_and_conditions | URL             |      ?      | URL to the operator’s terms and conditions.                                                                                                                  |
| last_updated         | DateTime        |      1      | Timestamp when this Connectors was last updated (or created).                                                                                                |
