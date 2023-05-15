---
title: "OCPI 2.1.1 implementation as eMSP"

date: 2022-07-11 09:55:28 -0400

categories: work

tags: ocpi security encryption


# 목차
toc: true  
toc_sticky: true 
---

# OCPI 2.1.1 implementation as eMSP

## implementation scope
- location module
- CPO : SHELL - Greenlot

## 1. FLOW

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

## 2. OCPI SPECs

### 2.1. location module
- endpoint structure : /ocpi/cpo/211/locations
- Flow and 구현방법 
  - Locations 모듈에는 기본 Object로 Location이 있고, Location에는 EVSE가 있고, EVSE에는 커넥터가 있습니다. eMSP 인터페이스의 방법으로 Locations 정보/상태를 eMSP와 공유할 수 있습니다. Locations 업데이트는 EVSE 또는 커넥터에만 수행할 수 있습니다. 
  - CPO는 Locations Object를 생성할 때 eMSP Locations Endpoint의 PUT를 호출하여 eMSP에 푸시합니다. **푸시 모드를 지원하지 않는 공급자(eMSP?)는 새 Object를 수신하기 위해 CPO Locations Endpoint의 GET을 호출해야 합니다.** 
  - CPO가 Locations 관련 Object를 교체하려는 경우 Locations Endpoint의 PUT을 호출하여 eMSP 시스템에 푸시합니다. 
  - Locations 관련 Object에 대한 모든 변경 사항은 eMSP Locations Endpoint의 PATCH를 호출하여 eMSP에 푸시할 수도 있습니다. **푸시 모드를 지원하지 않는 공급자(eMSP?)는 업데이트를 수신하기 위해 CPO Locations Endpoint의 GET을 호출해야 합니다.** 
  - CPO가 EVSE를 삭제하려면 상태 필드를 REMOVED로 설정하여 업데이트하고 eMSP 시스템에서 PUT 또는 PATCH를 호출해야 합니다. **유효한 EVSE Object가 없는 Location는 만료된 것으로 간주되어 더 이상 표시되지 않아야 합니다.** Location를 삭제하는 직접적인 방법은 없습니다. 
  - CPO가 eMSP 시스템의 Location, EVSE 또는 커넥터 Object의 상태 또는 존재에 대해 확신할 수 없는 경우 CPO는 GET를 호출하여 eMSP 시스템의 Object를 확인할 수 있습니다.


#### 2.1.1. method : GET
- 제공된 URL 세그먼트에 따라 GET 요청을 사용하여 이 CPO에서 사용 가능한 Locations 및 EVSE 목록(GET List)에 대한 정보를 검색하거나 특정 Location, EVSE 또는 커넥터에 대한 정보(GET Object)를 가져올 수 있습니다.
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

## 3. Object description
### 3.0 common 
#### 3.0.1 Requst
#### 3.0.2 Response 
##### 3.0.2.1 Response Format
##### 3.0.2.1.1 Response Result Code 
###### - 1xxx: 성공
| 코드 | 설명 |
| :--- | :---------------------: |
| 1000 | 일반 성공 코드 |

###### - 2xxx: 클라이언트 오류
클라이언트가 보낸 메시지에서 서버가 감지한 오류: 클라이언트가 뭔가 잘못했습니다.

| 코드   | 설명                                                           |
| :----- | :------------------------------------------------------------- |
| 2000년 | 일반 클라이언트 오류                                           |
| 2001년 | 유효하지 않거나 누락된 매개변수                                |
| 2002년 | 정보가 충분하지 않습니다. 예: 정보가 너무 적은 권한 부여 요청. |
| 2003년 | 알 수 없는 위치(예: 명령: 알 수 없는 위치의 START_SESSION).    |


###### - 3xxx: 서버 오류
서버에서 OCPI 페이로드를 처리하는 동안 오류가 발생했습니다. 메시지는 구문상 올바르지만 올바르지 않습니다.
서버에서 처리합니다.

| 코드 | 설명 |
| :--- | :--------------------------------------------------------------------- |
| 3000 | 일반 서버 오류 |
| 3001 | 클라이언트의 API를 사용할 수 없습니다. 예를 들어 자격 증명 등록 중: 자격 증명 endpoint에 대한 공개 POST 호출 중에 초기화 당사자가 상대방에게 데이터를 요청할 때. GET 중 하나를 처리할 수 없는 경우 당사자는 POST 응답에서 이 오류를 반환해야 합니다. |
| 3002 | 지원되지 않는 버전입니다. |
| 3003 | 당사자 간에 일치하는 endpoint 또는 예상되는 endpoint가 없습니다. 두 당사자가 사용할 수 있는 상호 모듈 또는 endpoint가 없거나 상대방 구현에서 예상하는 최소값이 없는 경우 등록 프로세스 중에 사용됩니다. |








---

### 3.1 Location Object
- Location Object는 함께 속한 EVSE 그룹이 설치된 Location와 속성을 설명합니다. 일반적으로 Location Object는 EVSE 그룹의 정확한 Location이지만 이러한 EVSE가 포함된 주차장 입구일 수도 있습니다. 각 EVSE에 도달하는 정확한 방법은 자체 속성에 의해 추가로 지정할 수 있습니다.
  
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

### 3.2 EVSE Object
- EVSE 객체는 단일 세션에서 단일 EV에 대한 전원 공급을 제어하는 부분을 설명합니다. 항상 Location Object에 속합니다. Location에서 EVSE로 가는 길찾기(예: 층, 물리적 참조 또는 길찾기)만 포함됩니다. 이러한 속성이 Locations 지점에서 EVSE에 도달하는 데 충분하지 않은 경우 일반적으로 이 EVSE를 다른 Locations Object에 배치해야 함을 나타냅니다(때로는 주소는 같지만 좌표/방향이 다름). EVSE Object에는 동시에 사용할 수 없는 커넥터 목록이 있습니다. 한 번에 EVSE당 하나의 커넥터만 사용할 수 있습니다.

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

### 3.3 Connector Object
- 커넥터는 EV에서 사용할 수 있는 소켓 또는 케이블입니다. 단일 EVSE는 여러 커넥터를 제공할 수 있지만 그 중 하나만 동시에 사용할 수 있습니다. 커넥터는 항상 EVSE Object에 속합니다.

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


## 4. version information module
- 이 endpoint은 사용 가능한 모든 OCPI 버전과 지원되는 endpoint과 같은 버전별 세부 정보를 찾을 수 있는 해당 URL을 나열합니다. 
endpoint 구조의 예: /ocpi/cpo/versions 및 /ocpi/emsp/versions 
구현된 버전 endpoint에 대한 정확한 URL은 OCPI 구현과 인터페이스하는 당사자에게 (오프라인) 제공되어야 합니다. 이 endpoint는 구현된 OCPI의 다른 모듈 및 버전. CPO와 eMSP에는 모두 이 endpoint가 있어야 합니다.

```json
[
    {
        "version": "2.1.1",
        "url": "https://example.com/ocpi/cpo/2.1.1/"
    },
    {
        "version": "2.0",
        "url": "https://example.com/ocpi/cpo/2.0/"
    }
]
```
## 5. version detail module

- 이 endpoint는 사용 가능한 모든 OCPI 버전과 지원되는 endpoint와 같은 버전별 세부 정보를 찾을 수 있는 해당 URL을 나열합니다.
endpoint 구조의 예: /ocpi/cpo/versions 및 /ocpi/emsp/versions
구현된 버전 endpoint에 대한 정확한 URL은 OCPI 구현과 인터페이스하는 당사자에게 (오프라인) 제공되어야 합니다. 이 endpoint은 구현된 OCPI의 다른 모듈 및 버전의 위치를 검색하기 위한 시작점입니다.
CPO와 eMSP에는 모두 이 endpoint이 있어야 합니다.

```json
{
    "version": "2.0",
    "endpoints": [
        {
            "identifier": "credentials",
            "url": "https://example.com/ocpi/cpo/2.0/credentials/"
        },
        {
            "identifier": "locations",
            "url": "https://example.com/ocpi/cpo/2.0/locations/"
        }
    ]
}
```

## 6. crudentials endpoint
- Example: /ocpi/cpo/2.0/credentials and /ocpi/emsp/2.0/credentials