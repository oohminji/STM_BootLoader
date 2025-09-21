# STM32기반 BootLoader
**세미나 발표 자료** : [stm부트로더](https://docs.google.com/presentation/d/1aQPwUHy2htVaIc0hbfxVewVQ1QZWxDCkgHesAlucqaY/edit?slide=id.p1&pli=1#slide=id.p1)
## BootLoader 란?
> 1. 컴퓨터 시스템이 부팅될 때 실행되는 프로그램<br>
> 2. 부트로더가 운영체제를 로드하고 실행하기 전, 하드웨어 초기화, 시스템 구성, 에러 처리, 운영체제 로딩 담당

## STM의 BootLoader 모드 종류
<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/eb05af26-796d-4262-813c-07c6fd947826" /><br>
<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/be1e37b7-9529-4527-b096-a77bdd82b49f" /><br>
> - **Main Flash memory** : 사용자가 직접 작성하고 업로드한 코드가 실행, Main Flash에서 부팅 됨<br>
> - **System memory** : UART, I2C, SPI, USB등을 통해 칩에 프로그램을 다운로드할 때 사용<br>
> - **Embedded SRAM** : 내장 STRAM에서 부팅,  휘발성 메모리로 주로 개발 및 디버깅 목적으로 사용<br>

## 핀 설정
<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/17a7bf49-abeb-4af8-beeb-be09c7b9808b" /><br>

> **BOOT0핀** <br>
> - LOW(0) : 기본적으로 GND(LOW)에 연결되어 있어 따로 연결하지 X<br>
> - HIGHT(1) : boot0핀을 3.3v핀과 연결(short)<br>

> **BOOT1(PB2)** <br>
> - LOW(0) : 기본적으로 GND(LOW)에 연결되어 있어 따로 연결하지 X<br>
> - HIGHT(1) : PB2핀을 3.3v핀과 연결(short)<br>

## 일반 CubeIDE 프로젝트
> - 메모리 구조 : 유저 어플리케이션<br>
> - 시작 주소 : 전원 인가 or Reset버튼 누를 시, 유저 어플리케이션<br>
> - 목적 : 단일 기능 수행 <br>

## Main Flash Memory 모드
> - 메모리 구조 : 플레시 메모리 공간을 두 영역으로 분할 (부트로더 + 유저 어플리케이션)<br>
> - 시작 주소 : 전원 인가 or Reset버튼 누를 시, 부트로더<br>
> - 목적 : 펌웨어 업데이트 ( OTA : Over The Air) <br>
> 즉, 유저 어플리케이션을 실행시켜주는 역할을 하는 ‘소프트웨어 부트로더’<br>

### **구현** <br>
> 1. 부트로더와 유저 어플리케이션 총 2개의 프로젝트 생성<br>
> 2. 각 프로젝트의 링커 스크립트 (.ld)파일의 Memory의 FLASH origin 주소과 length 크기 수정 EX) 부트로더 origin = 0x08000000, length = 32K 유저어플리케이션 origin = 0x08008000, length = 480K<br>
> 3. 부트로더와 유저 어플리케이션을 일반 CubeIDE 프로젝트 개발했던 것과 동일하게 GPIO, 인터럽트, 타이머 등 구현 가능<br>
> 4. 코드 작성할 때 주의할 점: 부트로더 프로젝트에 꼭 JUMP 코드 작성( 유저 어플리케이션에 점프하도록), 유저 어플리케이션영역보다 부트로더의 영역이 작기 때문에 간소화 하여 작성, main함수에 작성하되 while문엔 작성하지 않는 것이 좋음<br>
> 5. 두 프로젝트를 .hex 파일 or .bin파일로 디버깅하고 STM32CubeProgrammer 툴을 이용하여 업로드(단, .bin파일은 Start address를 정확히 적어 업로드)<br>

## System Memory 모드 
<img width="600" height="350" alt="image" src="https://github.com/user-attachments/assets/4c956575-cf68-47dd-b035-4f77bd665dd8" /> <br>

> - ST가 기본적으로 Rom 부트로더를 제공(별도의 플래싱 없이 칩에서 지원)
> - USART, USB, DFU, I2C, SPI, CAN같은 인터페이스를 통해 펌웨어를 다운로드 가능
> - BOOT0핀을 Hight(3.3V)로 설정하면 전원이 켜질 때 시스템 메모리에 들어가면서 내장 부트로더 실행
> - BOOT0핀을 LOW(0V)로 설정하면 Flash User code가 실행

### **구현** <br>
> 1. PL2303TA USB to TTL SERIAL CABLE을 이용하여 PC와 stm과 연결(Uart통신)
    
    검: GND
    
    흰: RXD(수신)
    
    초: TXD(전송)
    
    빨: 5V
    
> - 흰 → stm의 uart2 txd(PA9)와 연결
> - 초 → stm의 uart2 rxd(PA10)와 연결
> 2. 업로드할 프로젝트의 hex파일 or bin 파일을 업로드하고 Start Programming
> 3. 전원을 빼고 연결한 Boot0을 LOW로 빼고 전원을 인가해 Reset버튼을 눌러주면Flash User code가 실행
