본 데이터셋은 [한국 해양 교통 안전 공단 여객선 항로](https://www.data.go.kr/data/15126739/fileData.do)에서 제공되는 csv 형태의 데이터를 sqlite 형태의 데이터로 가공한 데이터 셋입니다.

원본 데이터는 {분류(ID),항로명,변침점순번,위도(LAT),경도(LON),변침점명,기항지명}의 Column 정보를 가지고 있으며,이를 가공하여, ports, routes, waypoints, route_ports의 테이블과, 위의 테이블들을 활용한 v_route_details 뷰를 가진 dataset을 생성하였습니다.

dataset은 sqlite, spatialite 두가지 포맷을 사용하고 있습니다.

![image](diagram.png)

# Table Schema

## ports(항만)

| 컬럼명      | 설명                                        |
| ----------- | ------------------------------------------- |
| port_id(PK) | 항만을 고유하게 식별하는 코드 (Primary Key) |
| port_name   | 항만의 공식 명칭                            |
| latitude    | 항만 위치의 위도 좌표                       |
| longitude   | 항만 위치의 경도 좌표                       |

## routes(항로)

| 컬럼명       | 설명                                        |
| ------------ | ------------------------------------------- |
| route_id(PK) | 항로를 고유하게 식별하는 코드 (Primary Key) |
| route_name   | 항로의 명칭(예: 부산-제주 1항로)            |

## waypoints(변침점)

| 컬럼명          | 설명                                        |
| --------------- | ------------------------------------------- |
| waypoint_id(PK) | 변침점을 고유하게 식별하는 코드             |
| route_id(FK)    | 해당 변침점이 속한 항로의 ID                |
| sequence_number | 항로 내 변침점의 순서(경유 순서)            |
| latitude        | 변침점의 위도 좌표                          |
| longitude       | 변침점의 경도 좌표                          |
| waypoint_name   | 변침점의 명칭(예: No.9 부이, 조도방파제 등) |
| port_name       | 변침점이 항만일 경우 해당 항만명            |

## route_ports(항로->항만 연관 테이블)

| 컬럼명       | 설명                                                |
| ------------ | --------------------------------------------------- |
| route_id(FK) | 항로를 고유하게 식별하는 코드                       |
| port_id(FK)  | 항만을 고유하게 식별하는 코드                       |
| port_order   | 항로 내 항만의 역할(출발지, 도착지, 중간 기항지 등) |

- 출발지와 도착지가 같은 순환 항로의 경우, 하나의 port_id만 존재할 수 있음.
- port_order를 통해 출발지/도착지/중간 기항지를 구분.

# View Schema

## v_route_details

| 컬럼명            | 설명                                          |
| ----------------- | --------------------------------------------- |
| route_id(FK)      | 항로를 고유하게 식별하는 코드                 |
| route_name        | 노선명                                        |
| departure_port    | 출발 항구명                                   |
| arrival_port      | 도착 항구명                                   |
| linestring_wkt    | 항로 전체를 GIS 좌표로 표현한 WKT(LineString) |
| waypoint_sequence | 항로 내 변침점들을 순서대로 나열한 문자열     |
| waypoint_count    | 항로에 포함된 변침점의 총 개수                |

- 항로 내 변침점 중 waypoint_name과 port_name이 없는 변침점은 제외되었습니다.
