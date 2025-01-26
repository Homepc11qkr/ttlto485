# ttlto485 소스
# https://github.com/greays/esphome
substitutions :
  name: 485-ttl-esp32-ip109
  device_description: "esp32dev ew11 485 ttl ip109"
  friendly_name:  "bolier_right_485-ttl-ip109"

esphome:
  name: ${name}
  comment: ${device_description}
  friendly_name: ${friendly_name}

  on_boot:
    priority: -100
    then:
      
      - logger.log: "Step 4: Checking WiFi connection..."
      - if:
          condition:
            wifi.connected:        # WiFi 연결 여부 확인
          then:
            - logger.log: "WiFi connected!"
          else:
            - logger.log: "WiFi not connected, restarting..."
            - delay: 10s          # 10초 대기 후 재부팅
            - lambda: |-
                #include "esp_system.h"
                esp_restart();


esp32:
  board: esp32dev
  framework:
    type: arduino
# esp32:
# 플래시 사이즈 및 파티션 재 설정  
  flash_size: 8MB
  partitions: "default_8MB.csv"      
  # partitions: "default_8MB.csv"      
  
api:
  encryption:
    key: "narKDp7FsU+WEXAKcWcXINe/CDOaAEHgP9LEbVhyc7o="
ota:
  - platform: esphome
    # safe_mode: true
    password: !secret ota_password_485


 

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: light  
  manual_ip:
    static_ip: 192.168.0.109
    gateway: 192.168.0.1
    subnet: 255.255.255.0
    dns1: 8.8.8.8
    dns2: 8.8.4.4
  # Enable fAllback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "C-Esp32 FAllback Hotspot"
    password: "LeCzah3Xwdyf"

  
captive_portal:

time:
  - platform: homeassistant
    id: homeassistant_time     
    timezone: Asia/Seoul
 
  
logger:
  level: INFO # DEBUG # INFO # 
  baud_rate: 0
 
debug:
  update_interval: 5s

# ========================= 485 wAllpad connection ========================== 
# ========================= 485 wAllpad connection ========================== 
# ========================= 485 wAllpad connection ========================== 
#  여기서부터 github 485 연결에 깡스님의 자료연결 및 스위치 보일러 센서 구성입니다
# https://cafe.naver.com/koreassistant/15454
# https://cafe.naver.com/stsmarthome/12973

external_components:
  - source: github://greays/esphome@rs485
    components: [ rs485 ]
#  485 연결에 깡스님의 자료연결 및 스위치 보일러 센서 구성
# ---- > ok 

# 9999999999999999999999999999999999999999999
# uart:
#   id: uart_bus
#   tx_pin: GPIO17
#   rx_pin: GPIO16
#   baud_rate: 9600
# # #  tx rx단자 확안 구성
# # # RS485 Component (for ttl to rs485 module)
# # #  - esp8266: UART0 (TX: GPIO1, RX: GPIO3)
# # #  - esp32: UART2 (TX: GPIO17, RX: GPIO16)
# # #  눈에 보이는 rx tx 단자 연결은 정확히 연결 합니다.

uart:
  id: uart_bus
  tx_pin: GPIO17
  rx_pin: GPIO16
  baud_rate: 9600
  # stop_bits: 1
  # parity: NONE
  # data_bits: 8

# ============================
# 초기 설치후 연결 작업
# captive_portal: 다음에 붙여넣습니다.
# ==========================================
# web_server:
# web_server:
#   port: 80
#   version: 2
#   include_internal: true
  

# mqtt:
#   broker: 192.168.0.35
#   username: mqtt_user
#   password: mqtt_pass  
#   discovery: false

binary_sensor:

  - platform: status
    name: "Connection Status ${name}"


text_sensor:
  - platform: version
    name: "Version"
 
# ===========================
  - platform: homeassistant
    id: wifi_signal_percent2
    entity_id: sensor.esp32_485_wifi_signal_percent
    internal: true         
# text_sensor: # esp32 information
  - platform: debug
    device:
      name: "Device Info"
    reset_reason:
      name: "Reset Reason"
# ===========================
# text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"
      id: ip
    ssid:
      name: "SSID"
    mac_address:
      name: "Mac" 
# =================     
# text_sensor:
  # Heap Memory 상태 (남은 메모리 / 총 메모리)
  - platform: template
    name: "1_Heap Memory Status"
    lambda: |-
      float total_heap = 327680.0;  // 총 힙 메모리 크기 (327680 bytes)
      float free_heap = ESP.getFreeHeap();  // 남은 힙 메모리
      char status[50];
      sprintf(status, "%.0f B / %.0f B", free_heap, total_heap);  // 남은 메모리 / 총 메모리 포맷
      return {status};  // 문자열 반환
    update_interval: 10s  # 10초마다 업데이트

sensor:
# sensor:

  # RAM 사용량 (%)
  - platform: template
    name: "1_RAM Usage (%)"
    lambda: |-
      float total_ram = 327680.0;  // 총 RAM 용량 (327680 bytes)
      float used_ram = total_ram - ESP.getFreeHeap();  // 사용된 RAM 계산
      return (used_ram / total_ram) * 100.0;  // RAM 사용률 계산 (퍼센트)
    unit_of_measurement: "%"
    accuracy_decimals: 1

  # 남은 RAM 메모리 (바이트 단위)
  - platform: template
    name: "1_Free RAM Memory"
    lambda: |-
      return ESP.getFreeHeap();  // 남은 힙 메모리 반환
    unit_of_measurement: "B"
    accuracy_decimals: 0

  - platform: uptime
    name: 1_Uptime _Sensor
    update_interval: 1s       
 
  - platform: wifi_signal # Reports the WiFi signal strength/RSSI in dB
    name: "WiFi_Signal_dB"
    id: wifi_signal_db
    update_interval: 30s
    entity_category: "diagnostic"

  - platform: copy # Reports the WiFi signal strength in %
    source_id: wifi_signal_db
    name: "WiFi Signal Percent"
    filters:
      - lambda: return min(max(2 * (x + 100.0), 0.0), 100.0);
    unit_of_measurement: "Signal %"
    entity_category: "diagnostic"
    device_class: ""
# =======================================    #  485 통신 sensor 추가  
# [거실온도] 0x11 7번째 장소 
  - platform: rs485
    name: "temperature_living_room_°C"
    unit_of_measurement: "°C"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x00] 
    # # 1~6 off ack 마지막 패킷 0x00 추가
    #  8번째(온도정보) 
    data:
      offset: 8
      length: 1
      precision: 0
# 이하 동일하게 복사후 방 패킷/이름만 변경 

# [안방온도] 0x12 7번째 장소
  - platform: rs485
    name: "temperature_big_room_°C"
    unit_of_measurement: "°C"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x00] 
    # # 1~6 on/off ack 마지막 패킷 0x00
    data:
      offset: 8
      length: 1
      precision: 0
# [작은방온도] 0x13 7번째 장소
  - platform: rs485
    name: "temperature_small_room_°C"
    unit_of_measurement: "°C"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x00]
     # # 1~6 on/off ack 마지막 패킷 0x00
    data:
      offset: 8
      length: 1
      precision: 0
# [서재방온도] 0x14 7번째 장소      
  - platform: rs485
    name: "temperature_book_room_°C"
    unit_of_measurement: "°C"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x00] 
    # # 1~6 on/off ack 마지막 패킷 0x00
    data:
      offset: 8
      length: 1
#  이하 수정없이 사용

rs485:
  # uart_id: uart_bus
  # update_interval: 5s  # 5초마다 상태 요청
    
  baud_rate: 9600
  #Required
  data_bits: 8
  #Option(default: 8)
  parity: 0
  #Option(default: 0)
  stop_bits: 1
  #Option(default: 1)
  
  rx_wait: 10     
  #Option(default: 10ms) -> 수신 메시지 대기시간 (10ms 미만으로 수신된 메시지만 한 패킷으로 판단)
  tx_interval: 50
  #Option(default: 50ms) -> 발신 메시지 전송 간격 (패킷 수신 후 50ms 대기 후 전송)
  tx_wait: 50
      #Option(default: 50ms) -> 발신 메시지 Ack 대기시간
  tx_retry_cnt: 5 # 3
  #Option(default: 3)    -> 발신 메시지 Ack 없을 경우 재시도 횟수
  # ctrl_pin: GPIO2 #Option -> 수동 제어 모듈 사용시 셋팅 (MAX485모듈의 DE,RE에 연결된 PIN)
  prefix: [0xF7]
  #Option -> 값 세팅시 모든 수신 패킷 Check, 발신 패킷에 Append
  suffix: [0xEE]
  #Option -> 값 세팅시 모든 수신 패킷 Check, 발신 패킷에 Append

  checksum: true
  # Option(default: False) -> 체크섬 사용 여부 (lambda 사용 시 세팅 불필요)
  # checksum: !lambda |-
  #   uint8_t crc = 0xF7; // data 변수에는 prefix 제외되어 있음
  #   for (num_t i = 0; i < len; i++) {
  #     crc ^= data[i];
  #   }
  #   return crc;

  
  checksum2: False
   #Option(default: False) -> ADD 체크섬 사용여부
  # checksum2: !lambda |-  
  #   uint8_t crc = 0xF7; // data 변수에는 prefix 제외되어 있음
  #   for(num_t i=0; i<len; i++)
  #     crc += data[i];
  #   crc += checksum1; // 첫번째 체크섬 계산 결과
  #   return crc;

  state_response:
     #Option -> 값 세팅시 response 패킷 수신 후에 명령 패킷 송신
    data: [0x04]
     #비트연산 시 위치 값 참고 => 1: 0x01, 2: 0x02, 3: 0x04, 4: 0x08, 5: 0x10, 6: 0x20, 7: 0x40, 8: 0x80
    offset: 3
    and_operator: False
     #Option(default: False) 수신패킷의 offset 위치와 data[0]를 And 비트연산
    inverted: False
     #Option(default: False) 결과 값 반전(일치하지 않을 경우 참)



light:
# 99999999999999999999999999999999999999
# -------------------------------------------------------------------------
  - platform: rs485
    name: "Light_living_room" # 거실불
    id: living_room_a
    device: [0x0b, 0x01, 0x19, 0x04] # 4자리
    sub_device:
      offset: 5  # 5번다음 data값
      data: [0x11]  # 5번다음 data값
    state_on:
      offset: 7 
      data: [0x01] # 7번째 값
      and_operator: true
      inverted: false
    state_off:
      offset: 7
      data: [0x01]  # state_on data 동일 7번째 값
      and_operator: true
      inverted: true
    command_on:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x11, 0x01, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x11, 0x01, 0x01] 
    command_off:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x11, 0x02, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x11, 0x02, 0x02]
    command_state: [0x0b, 0x01, 0x19, 0x01, 0x40, 0x11, 0x00, 0x00]
    # 1줄로 command_state 추가 
    update_interval: 11s # 적단한 숫자
# -------------------------------------------------------------------------
# [W][rs485:293]: [Read] Checksum error: 0xF7 0x0B 0x01 0x19 0x01 0x40 0x11 0x00 0x00 0x00 0xEE (11 byte)


  - platform: rs485
    name: "Light_living_room2" # 거실불2
    id: livingroom2_a
    device: [0x0b, 0x01, 0x19, 0x04]
    sub_device:
      offset: 5
      data: [0x12]
    state_on:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: false
    state_off:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: true
    command_on:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x12, 0x01, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x12, 0x01, 0x01]
    command_off:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x12, 0x02, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x12, 0x02, 0x02]
    command_state: [0x0b, 0x01, 0x19, 0x01, 0x40, 0x12, 0x00, 0x00]
    update_interval: 11s
# -------------------------------------------------------------------------
  - platform: rs485
    name: "Light_sofa" # 소파불
    id: living_room_sofa_a
    device: [0x0b, 0x01, 0x19, 0x04]
    sub_device:
      offset: 5
      data: [0x13] # 전등
    state_on:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: false
    state_off:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: true
    command_on:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x13, 0x01, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x13, 0x01, 0x01]
    command_off:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x13, 0x02, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x13, 0x02, 0x02]
    command_state: [0x0b, 0x01, 0x19, 0x01, 0x40, 0x13, 0x00, 0x00]
    update_interval: 11s
# -------------------------------------------------------------------------
  - platform: rs485
    name: "Light_front" # 앞불
    id: living_room_front_a
    device: [0x0b, 0x01, 0x19, 0x04]
    sub_device:
      offset: 5
      data: [0x14]
    state_on:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: false
    state_off:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: true
    command_on:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x14, 0x01, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x14, 0x01, 0x01]
    command_off:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x14, 0x02, 0x00]
      ack:  [0x0b, 0x01, 0x19, 0x04, 0x40, 0x14, 0x02, 0x02]
    command_state: [0x0b, 0x01, 0x19, 0x01, 0x40, 0x14, 0x00, 0x00]
    update_interval: 11s
# -------------------------------------------------------------------------
  - platform: rs485
    name: "Light_bogdo" # 복도불
    id: living_room_bogdo_a
    device: [0x0b, 0x01, 0x19, 0x04]
    sub_device:
      offset: 5
      data: [0x15]
    state_on:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: false
    state_off:
      offset: 7
      data: [0x01]
      and_operator: true
      inverted: true
    command_on:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x15, 0x01, 0x00]
      ack: [0x0b, 0x01, 0x19, 0x04, 0x40, 0x15, 0x01, 0x01]
    command_off:
      data: [0x0b, 0x01, 0x19, 0x02, 0x40, 0x15, 0x02, 0x00]
      ack: [0x0b, 0x01, 0x19, 0x04, 0x40, 0x15, 0x02, 0x02]
    command_state: [0x0b, 0x01, 0x19, 0x01, 0x40, 0x15, 0x00, 0x00]
    update_interval: 11s
# ------------------------------------------------------------------------- 
# =========================================================================
# ===================== RS485 Climate  ====================================
#--------------------------------------------------------------------------
climate:
#----------------------------------climate---------------------------------------  
  - platform: rs485
    name: "Boiler_living_room"
    id: Boiler_living_room_a
    icon: "mdi:water-thermometer"
    visual:
      min_temperature: 5 °C
      max_temperature: 40 °C
      temperature_step: 1 °C
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x00]
    state_current:
      offset: 8
      length: 1
      precision: 0

    state_target:
      offset: 9
      length: 1
      precision: 0
    
    state_off:  #
      offset: 7
      data: [0x04]

    state_heat:
      offset: 7
      data: [0x01]

    state_away:
      offset: 7
      data: [0x07]

      # 전등은  data: [0x0b ,ack: [0x0b,  보일러는 data: [0x0b ,ack: [0x0d,,,소문자
      # 보일러 온도조절은 0x45[5] 고정한다
    command_off: #Required (끄기 명령 )
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x04, 0x04]

    command_heat: #Option (난방모드 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x01, 0x01]   

    command_away: #Option (외출 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x07, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x07, 0x01]

    command_home: #Option (재실모드)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x01, 0x01]

    command_temperature: !lambda |-  
      return {
                {0x0b, 0x01, 0x18, 0x02, 0x45, 0x11, (uint8_t)x, 0x00},
                {0x0d, 0x01, 0x18, 0x04, 0x45, 0x11, (uint8_t)x, 0x01}
             };

    command_state:  [0x0b, 0x01, 0x18, 0x01, 0x46, 0x11, 0x00, 0x00] 
    update_interval: 8s
# ---------------------------------------climate---------------------------------------------
# [안방] 0x12  7번째 장소 
  - platform: rs485
    name: "Boiler_big_room"
    id: Boiler_big_room_a
    icon: "mdi:water-thermometer"
    visual:
      min_temperature: 5 °C
      max_temperature: 40 °C
      temperature_step: 1 °C
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x00]
    # 0x45  ,,,,,0x12 및 7자리
    state_current:
      offset: 8
      length: 1
      precision: 0

    state_target:
      offset: 9
      length: 1
      precision: 0

    state_off:
      offset: 7
      data: [0x04]

    state_heat:
      offset: 7
      data: [0x01]

    state_away:
      offset: 7
      data: [0x07]

    command_off: #Required (끄기 명령)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x04, 0x04]

    command_heat: #Option (난방모드 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x01, 0x01]     

    command_away: #Option (외출 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x07, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x07, 0x01]

    command_home: #Option (재실모드)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x01, 0x01]

    command_temperature: !lambda |-  
       return {
                {0x0b, 0x01, 0x18, 0x02, 0x45, 0x12, (uint8_t)x, 0x00},
                {0x0d, 0x01, 0x18, 0x04, 0x45, 0x12, (uint8_t)x, 0x01}
             };
    # 0x45  온도에 5번째값
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x12, 0x00, 0x00] 
    update_interval: 8s
#-----------------------------------climate---------------------------------------
# [작은방] 0x13  7번째 장소
  - platform: rs485
    name: "Boiler_small_room"
    id: Boiler_small_room_a
    icon: "mdi:water-thermometer"
    visual:
      min_temperature: 5 °C
      max_temperature: 40 °C
      temperature_step: 1 °C
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x00]
    state_current:
      offset: 8
      length: 1
      precision: 0

    state_target:
      offset: 9
      length: 1
      precision: 0

    state_off:
      offset: 7
      data: [0x04]

    state_heat:
      offset: 7
      data: [0x01]

    state_away:
      offset: 7
      data: [0x07]

    command_off: #Required (끄기 명령)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x04, 0x04]

    command_heat: #Option (난방모드 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x01, 0x01]

    command_away: #Option (외출 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x07, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x07, 0x01]

    command_home: #Option (재실모드)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x01, 0x01]

    command_temperature: !lambda |- 
      return {
                {0x0b, 0x01, 0x18, 0x02, 0x45, 0x13, (uint8_t)x, 0x00},
                {0x0d, 0x01, 0x18, 0x04, 0x45, 0x13, (uint8_t)x, 0x01}
             };
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x13, 0x00, 0x00] 
    update_interval: 8s
#----------------------------------climate----------------------------------------
# [서재방] 0x14 7번째 장소
  - platform: rs485
    name: "Boiler_book_room"
    id: Boiler_book_room_a
    icon: "mdi:water-thermometer"
    visual:
      min_temperature: 5 °C
      max_temperature: 40 °C
      temperature_step: 1 °C
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x00]
    state_current:
      offset: 8
      length: 1
      precision: 0

    state_target:
      offset: 9
      length: 1
      precision: 0

    state_off:
      offset: 7
      data: [0x04]

    state_heat:
      offset: 7
      data: [0x01]

    state_away:
      offset: 7
      data: [0x07]

    command_off: #Required (끄기 명령)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x04, 0x04]

    command_heat: #Option (난방모드 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x01, 0x01]

    command_away: #Option (외출 켜기)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x07, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x07, 0x01]

    command_home: #Option (재실모드)
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x01, 0x01]

    command_temperature: !lambda |- 
      return {
                {0x0b, 0x01, 0x18, 0x02, 0x45, 0x14, (uint8_t)x, 0x00},
                {0x0d, 0x01, 0x18, 0x04, 0x45, 0x14, (uint8_t)x, 0x01}
             };
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x14, 0x00, 0x00] 
    update_interval: 8s

# ===============================================================================================
# 🔎 RS485 패킷 규칙 검토  https://github.com/greays/esphome
# 0x46은 난방 모드 또는 특정 온도 제어 기능에 사용될 수 있습니다.
# 온도조정부분은 # 0x45 입니다 ( haos에서조정시 실제 온도조절기 온도 조정 되나 확인 합니다 )
# ===============================================================================================
 
# switch:
switch:
  - platform: gpio
    pin:
      number: GPIO2
      mode: INPUT_PULLUP
      inverted: true
    name: "Restart_Node Switch"
    on_turn_on:
      then:
        - switch.turn_on: Restart_Node_id
    restore_mode: ALWAYS_OFF  # 부팅 시 OFF 상태로 시작

  - platform: restart
    id: Restart_Node_id
    name: "Restart Node"
    restore_mode: ALWAYS_OFF  # 부팅 시 항상 OFF 상태로 유지

# ===============================================================================================
# 보일러  climate 스위치 구성으로  All switch 구성 준비
# =====================================switch==========================================================
# Automation 사용 예제 (rs485.write) - with ACK
  - platform: rs485
    name: "Boiler_living_room_switch"
    icon: "mdi:power-socket-eu"
    id: "Boiler_living_room_switch"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x00]
    state_off:
      offset: 7
      data: [0x04]
    command_off:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x04, 0x04]
    state_on:
      offset: 7
      data: [0x01]
    command_on:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x01, 0x01]
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x11, 0x00, 0x00] 
    update_interval: 8s

# -----------------------------switch-----------------------------------------
# Boiler_big_room 12
  - platform: rs485
    name: "Boiler_big_room_switch"
    icon: "mdi:power-socket-eu"
    id: "Boiler_big_room_switch"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x00]
     # 2024_0924_2347_27 에서 11 to 12수정
    state_off:
      offset: 7
      data: [0x04]
    command_off:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x04, 0x04]
    state_on:
      offset: 7
      data: [0x01]
    command_on:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x01, 0x01]
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x12, 0x00, 0x00] 
    update_interval: 8s
# -------------------------------switch---------------------------------------
  # Boiler_small_room_sw 13
  - platform: rs485
    name: "Boiler_small_room_switch"
    icon: "mdi:power-socket-eu"
    id: "Boiler_small_room_switch"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x00]
    state_off:
      offset: 7
      data: [0x04]
    command_off:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x04, 0x04]
    state_on:
      offset: 7
      data: [0x01]
    command_on:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x01, 0x01]
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x13, 0x00, 0x00] 
    update_interval: 8s
# -----------------------------switch-----------------------------------------
# Boiler_book_room_sw 14 서재보일러 스위치동작
  - platform: rs485
    name: "Boiler_book_room_switch"
    icon: "mdi:power-socket-eu"
    id: "Boiler_book_room_switch"
    device: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x00]
    # # 1~6 on/off ack 마지막 패킷 0x00
    state_off:
      offset: 7
      data: [0x04]
    command_off:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x04, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x04, 0x04]   
    state_on:
      offset: 7
      data: [0x01]
    command_on:
      data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x01, 0x00]
      ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x01, 0x01]
    command_state: [0x0b, 0x01, 0x18, 0x01, 0x46, 0x14, 0x00, 0x00] 
    update_interval: 8s
# ===============================================================================================
button:


  - platform: template
    name: "Light_All_on"
    on_press:
      then:
        - light.turn_on: living_room_a
        - delay: 1s
        - light.turn_on: livingroom2_a
        - delay: 1s
        - light.turn_on: living_room_sofa_a
        - delay: 1s
        - light.turn_on: living_room_front_a
        - delay: 1s
        - light.turn_on: living_room_bogdo_a
        - delay: 1s
        - light.turn_on: living_room_a
        - delay: 1s
        - light.turn_on: livingroom2_a
        - delay: 1s
        - light.turn_on: living_room_sofa_a
        - delay: 1s
        - light.turn_on: living_room_front_a
        - delay: 1s
        - light.turn_on: living_room_bogdo_a
        - delay: 1s
  - platform: template
    name: "Light_All_off"
    on_press:
      then:
        - light.turn_off: living_room_a
        - delay: 1s
        - light.turn_off: livingroom2_a
        - delay: 1s
        - light.turn_off: living_room_sofa_a
        - delay: 1s
        - light.turn_off: living_room_front_a
        - delay: 1s
        - light.turn_off: living_room_bogdo_a
        - delay: 1s
        - light.turn_off: living_room_a
        - delay: 1s
        - light.turn_off: livingroom2_a
        - delay: 1s
        - light.turn_off: living_room_sofa_a
        - delay: 1s
        - light.turn_off: living_room_front_a
        - delay: 1s
        - light.turn_off: living_room_bogdo_a
        - delay: 1s
# ======================================================================
# 보일러 All off
# --------------------------------  
# [모든 보일러 ON 자동화]
# switch:# 모든 보일러 HEAT 모드 설정 스위치
# switch:
  - platform: template
    name: "Boiler_All_on_rs485"
    on_press:
      then:
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x01, 0x00]  # 거실 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x11, 0x01, 0x00]  # 거실 보일러 On
            
        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x01, 0x00]  # 큰방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x12, 0x01, 0x00]  # 큰방 보일러 On

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x01, 0x00]  # 작은방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x13, 0x01, 0x00]  # 작은방 보일러 On

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x01, 0x00]  # 서재방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x14, 0x01, 0x00]  # 서재방 보일러 On

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x01, 0x00]  # 거실 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x11, 0x01, 0x00]  # 거실 보일러 On
            
        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x01, 0x00]  # 큰방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x12, 0x01, 0x00]  # 큰방 보일러 On

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x01, 0x00]  # 작은방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x13, 0x01, 0x00]  # 작은방 보일러 On

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x01, 0x00]  # 서재방 보일러 On
            ack:  [0x0b, 0x01, 0x18, 0x04, 0x46, 0x14, 0x01, 0x00]  # 서재방 보일러 On
# 모든 보일러 OFF 모드 설정 스위치
# 보일러를 꺼야 하는 경우에는 climate.control의 IDLE 모드 대신 RS485 패킷을 직접 보내는 방식이 더 적합합니다.
# 아래는 모든 보일러를 OFF 상태로 설정하는 구성을 제공해 드립니다.
# switch: -------------------------------------------------------------

  - platform: template
    name: "Boiler_All_off_rs485"
    on_press:
      then:
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x04, 0x00]  # 거실 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x04, 0x04]   
            
        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x04, 0x00]  # 큰방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x04, 0x04]   

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x04, 0x00]  # 작은방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x04, 0x04]   

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x04, 0x00]  # 서재방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x04, 0x04]   
            #  한번더 명령을 2회실행하면 빠지는부분 해결
        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x11, 0x04, 0x00]  # 거실 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x11, 0x04, 0x04]   
            
        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x12, 0x04, 0x00]  # 큰방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x12, 0x04, 0x04]   

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x13, 0x04, 0x00]  # 작은방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x13, 0x04, 0x04]   

        - delay: 1s
        - rs485.write:
            data: [0x0b, 0x01, 0x18, 0x02, 0x46, 0x14, 0x04, 0x00]  # 서재방 보일러 OFF
            ack: [0x0d, 0x01, 0x18, 0x04, 0x46, 0x14, 0x04, 0x04]           
# switch: -------------------------------------------------------------
  - platform: template
    name: "Boiler_All_off_switch"
    on_press:
      then:
        - switch.turn_off: Boiler_living_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_big_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_small_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_book_room_switch
        - delay: 1s
        # 한번더 명령을 2회실행하면 빠지는부분 해결
        - switch.turn_off: Boiler_living_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_big_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_small_room_switch
        - delay: 1s
        - switch.turn_off: Boiler_book_room_switch
        - delay: 1s        
# switch: -------------------------------------------------------------
# 보일러 All on
# --------------------------------  
  - platform: template
    name: "Boiler_All_on_switch"
    on_press:
      then:
        - switch.turn_on: Boiler_living_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_big_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_small_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_book_room_switch
         # 한번더 명령을 2회실행하면 빠지는부분 해결
        - delay: 1s
        - switch.turn_on: Boiler_living_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_big_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_small_room_switch
        - delay: 1s
        - switch.turn_on: Boiler_book_room_switch
        - delay: 1s        
# switch: -------------------------------------------------------------
# 보일러 온도설정
# --------------------------------  
# button:
  - platform: template
    name: "23_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 23°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 23°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 23°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 23°C
        - delay: 1s    
        # 한번더 명령을 2회실행하면 빠지는부분 해결
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 23°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 23°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 23°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 23°C
        - delay: 1s            
# switch: -------------------------------------------------------------
  - platform: template
    name: "22_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 22°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 22°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 22°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 22°C
        - delay: 1s    
         # 한번더 명령을 2회실행하면 빠지는부분 해결
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 22°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 22°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 22°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 22°C
        - delay: 1s            
# switch: -------------------------------------------------------------
  - platform: template
    name: "20_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 20°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 20°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 20°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 20°C
             # 한번더 명령을 2회실행하면 빠지는부분 해결
        - delay: 1s    
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 20°C

        - delay: 1s            
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 20°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 20°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 20°C
        - delay: 1s    

# switch: -------------------------------------------------------------
  - platform: template
    name: "18_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 18°C
        - delay: 1s    
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 18°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 18°C
        - delay: 1s           
# switch: -------------------------------------------------------------
  - platform: template
    name: "10_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 10°C
        - delay: 1s    
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 10°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 10°C
        - delay: 1s    
              
# switch: -------------------------------------------------------------
  - platform: template
    name: "0_°C_All_Setting"
    on_press:
      then:
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 0°C
        - delay: 1s    
        - climate.control:
            id: Boiler_living_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_big_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_small_room_a
            target_temperature: 0°C
        - delay: 1s                
        - climate.control:
            id: Boiler_book_room_a
            target_temperature: 0°C
        - delay: 1s    

# switch: -------------------------------------------------------------
# 
# ============================================
# sensor:  # 20231226_1230_55
# backlight   20240110_1154_28
# ===============================
# 20231225_0022_52
# (구 현대 월패드 정보)
# ----전등--------
# 거실1 전등
# 켜기 요청 F7 0B 01 19 02 40 11 01 00 B6 EE
# 켜기 응답 F7 0B 01 19 04 40 11 01 01 B1 EE
# 끄기 요청 F7 0B 01 19 02 40 11 02 00 B5 EE
# 끄기 응답 F7 0B 01 19 04 40 11 02 02 B1 EE
# ​
# 거실2 전등 
# 켜기 요청 F7 0B 01 19 02 40 12 01 00 B5 EE
# 켜기 응답 F7 0B 01 19 04 40 12 01 01 B2 EE
# 끄기 요청 F7 0B 01 19 02 40 12 02 00 B6 EE
# 끄기 응답 F7 0B 01 19 04 40 12 02 02 B2 EE
# ​
# 소파3 전등 
# 켜기 요청 F7 0B 01 19 02 40 13 01 00 B4 EE
# 켜기 응답 F7 0B 01 19 04 40 13 01 01 B3 EE
# 끄기 요청 F7 0B 01 19 02 40 13 02 00 B7 EE
# 끄기 응답 F7 0B 01 19 04 40 13 02 02 B3 EE
# ​
# 앞불4 전등 
# 켜기 요청 F7 0B 01 19 02 40 14 01 00 B3 EE
# 켜기 응답 F7 0B 01 19 04 40 14 01 01 B4 EE
# 끄기 요청 F7 0B 01 19 02 40 14 02 00 B0 EE
# 끄기 응답 F7 0B 01 19 04 40 14 02 02 B4 EE
# ​
# 복도5 전등
# 켜기 요청 F7 0B 01 19 02 40 15 01 00 B2 EE
# 켜기 응답 F7 0B 01 19 04 40 15 01 01 B5 EE
# 끄기 요청 F7 0B 01 19 02 40 15 02 00 B1 EE
# 끄기 응답 F7 0B 01 19 04 40 15 02 02 B5 EE
# --------------------------------------------------
# ---------  보일러 -------------
# 거실 보일러
# 켜기 요청 F7 0B 01 18 02 46 11 01 00 B1 EE
# 켜기 응답 F7 0D 01 18 04 46 11 01 01 17 15 B2 EE
# 끄기 요청 F7 0B 01 18 02 46 11 04 00 B4 EE
# 끄기 응답 F7 0D 01 18 04 46 11 04 04 17 15 B2 EE
# 7도  요청 F7 0B 01 18 02 45 11 07 00 B4 EE
# 7도  응답 F7 0E 01 00 00 00 10 01 02 00 60 E9 FF
# ---------------
# 안방 보일러
# 켜기 요청 F7 0B 01 18 02 46 12 01 00 B2 EE
# 켜기 응답 F7 0D 01 18 04 46 12 01 01 17 0F AB EE
# 끄기 요청 F7 0B 01 18 02 46 12 04 00 B7 EE
# 끄기 응답 F7 0D 01 18 04 46 12 04 04 17 0F AB EE
# 7도  요청 F7 0B 01 18 02 45 12 07 00 B7 EE
# 7도  응답 F7 0D 01 18 04 45 12 07 01 17 07 A6 EE
# ---------
# 작은방 보일러
# 켜기 요청 F7 0B 01 18 02 46 13 01 00 B3 EE
# 켜기 응답 F7 0D 01 18 04 46 13 01 01 16 07 A3 EE
# 끄기 요청 F7 0B 01 18 02 46 13 04 00 B6 EE
# 끄기 응답 F7 0D 01 18 04 46 13 04 04 16 07 A3 EE
# 7도  요청 F7 0B 01 18 02 45 13 07 00 B6 EE
# 7도  응답 F7 0D 01 18 04 45 13 07 01 16 07 A6 EE
# ---------
# 서재 보일러
# 켜기 요청 F7 0B 01 18 02 46 14 01 00 B4 EE
# 켜기 응답 F7 0D 01 18 04 46 14 01 01 16 07 A4 EE
# 끄기 요청 F7 0B 01 18 02 46 14 04 00 B1 EE
# 끄기 응답 0D 01 18 04 46 14 04 04 16 07 A4 EE
# 7도  요청 F7 0B 01 18 02 45 14 07 00 B1 EE
# 7도  응답 F7 0D 01 18 04 45 14 07 01 16 07 A1 EE
# =========================== 변환 ==========================
# 거실1전등	on	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x11,	0x01,	0x00,	0xB6,	0xEE,	 	 
# 거실1전등	on	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x11,	0x01,	0x01,	0xB1,	0xEE,	 	 
# 거실1전등	off	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x11,	0x02,	0x00,	0xB5,	0xEE,	 	 
# 거실1전등	off	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x11,	0x02,	0x02,	0xB1,	0xEE,	 	 

# 거실2전등	on	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x12,	0x01,	0x00,	0xB5,	0xEE,	 	 
# 거실2전등	on	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x12,	0x01,	0x01,	0xB2,	0xEE,	 	 
# 거실2전등	off	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x12,	0x02,	0x00,	0xB6,	0xEE,	 	 
# 거실2전등	off	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x12,	0x02,	0x02,	0xB2,	0xEE,	 	 

# 소파3전등	on	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x13,	0x01,	0x00,	0xB4,	0xEE,	 	 
# 소파3전등	on	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x13,	0x01,	0x01,	0xB3,	0xEE,	 	 
# 소파3전등	off	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x13,	0x02,	0x00,	0xB7,	0xEE,	 	 
# 소파3전등	off	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x13,	0x02,	0x02,	0xB3,	0xEE,	 	 

# 앞불4전등	on	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x14,	0x01,	0x00,	0xB3,	0xEE,	 	 
# 앞불4전등	on	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x14,	0x01,	0x01,	0xB4,	0xEE,	 	 
# 앞불4전등	off	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x14,	0x02,	0x00,	0xB0,	0xEE,	 	 
# 앞불4전등	off	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x14,	0x02,	0x02,	0xB4,	0xEE,	 	 

# 복도등	on	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x15,	0x01,	0x00,	0xB2,	0xEE,	 	 
# 복도등	on	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x15,	0x01,	0x01,	0xB5,	0xEE,	 	 
# 복도등	off	요청		0xF7,	0x0b,	0x01,	0x19,	0x02,	0x40,	0x15,	0x02,	0x00,	0xB1,	0xEE,	 	 
# 복도등	off	응답		0xF7,	0x0b,	0x01,	0x19,	0x04,	0x40,	0x15,	0x02,	0x02,	0xB5,	0xEE,	 	 
#---------------------------------------------
# 거실보일러	off요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x11,	0x04,	0x00,	0xB4,	0xEE,	 	 
# 거실보일러	off응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x11,	0x04,	0x04,	0x14,	0x16,	0xB2,	0xEE,
# 거실보일러	7	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x11,	0x07,	0x00,	0xB4,	0xEE,	 	 
# 거실보일러	7	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x11,	0x07,	0x01,	0x14,	0x7,	0xA6,	0xEE,
# 거실보일러	8	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x11,	0x08,	0x00,	0xBB,	0xEE,	 	 
# 거실보일러	8	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x11,	0x08,	0x01,	0x14,	0x8,	0xA6,	0xEE,

# 안방보일러	on요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x12,	0x01,	0x00,	0xB2,	0xEE,	 	 
# 안방보일러	on응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x12,	0x01,	0x01,	0x15,	0x16,	0xB0,	0xEE,
# 안방보일러	off요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x12,	0x04,	0x00,	0xB7,	0xEE,	 	 
# 안방보일러	off응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x12,	0x04,	0x04,	0x15,	0x16,	0xB0,	0xEE,
# 안방보일러	7	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x12,	0x07,	0x00,	0xB7,	0xEE,	 	 
# 안방보일러	7	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x12,	0x07,	0x01,	0x15,	0x7,	0xA4,	0xEE,
# 안방보일러	8	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x12,	0x08,	0x00,	0xB8,	0xEE,	 	 
# 안방보일러	8	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x12,	0x08,	0x01,	0x15,	0x8,	0xA4,	0xEE,

# 작은방보일러	on요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x13,	0x01,	0x00,	0xB3,	0xEE,	 	 
# 작은방보일러	on응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x13,	0x01,	0x01,	0x13,	0x16,	0xB7,	0xEE,
# 작은방보일러	off요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x13,	0x04,	0x00,	0xB6,	0xEE,	 	 
# 작은방보일러	off응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x13,	0x04,	0x04,	0x13,	0x16,	0xB7,	0xEE,
# 작은방보일러	7	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x13,	0x07,	0x00,	0xB6,	0xEE,	 	 
# 작은방보일러	7	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x13,	0x07,	0x01,	0x13,	0x7,	0xA3,	0xEE,
# 작은방보일러	8	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x13,	0x08,	0x00,	0xB9,	0xEE,	 	 
# 작은방보일러	8	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x13,	0x08,	0x01,	0x13,	0x8,	0xA3,	0xEE,

# 서재보일러	on요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x14,	0x01,	0x00,	0xB4,	0xEE,	 	 
# 서재보일러	on응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x14,	0x01,	0x01,	0x17,	0x17,	0xB5,	0xEE,
# 서재보일러	off요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x46,	0x14,	0x04,	0x00,	0xB1,	0xEE,	 	 
# 서재보일러	off응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x46,	0x14,	0x04,	0x04,	0x17,	0x17,	0xB5,	0xEE,
# 서재보일러	7	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x14,	0x07,	0x00,	0xB1,	0xEE,	 	 
# 서재보일러	7	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x14,	0x07,	0x01,	0x17,	0x7,	0xA0,	0xEE,
# 서재보일러	8	요청		0xF7,	0x0b,	0x01,	0x18,	0x02,	0x45,	0x14,	0x08,	0x00,	0xBE,	0xEE,	 	 
# 서재보일러	8	응답		0xF7,	0x0d,	0x01,	0x18,	0x04,	0x45,	0x14,	0x08,	0x01,	0x17,	0x8,	0xA0,	0xEE,
#---------------------------------------------
 
# 패킷정보 1,2,3,4,5,6,,7,8,9,10,11,12,13 의 정보
# 1,시작 패킷
# 2,조명 패킷은 명령/응답 모두 0b와 보일러 명령0b 응답0d 
# 3,패킷의길이(설치된 조명개수 +10)
# 4, 패킷의길이(설치된 조명개수 +10)
# 5,명령패킷 2,응답패킷4
# 6,전등40,보일러45 46
# 7,방번호/장소(방+조명위치) 11,12,13,14,15
# 8,on/off(01:on,02:off) or 온도
# 9,요청(00),켜기응답(01),끄기응답(02)
# 10,xor의 checksum
# 11,마지막 종료
# 12,xor의 checksum
# 13,마지막 종료
# 2025_0108_1535_24
# =======================================    # 
# file:///G:/ew11/information_%EC%A0%95%EB%B3%B4/Thermostat%20Protocol_(Thermostat)_231110_example.pdf
# https://github.com/greays/esphome?tab=readme-ov-file
# 기준으로 작성 횔용 수정 합니다
# =====================================  2025_0112_0033_00

