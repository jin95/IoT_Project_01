# RTSP Server
#### 리눅스에서 카메라 입력을 받기위한 표준 인터페이스
<li>RTP/UDP unicast</li>
<li>RTP/UDP multicast</li>
<li>RTP/TCP</li>
<li>RTP/RTSP/HTTP</li>

## Usage 1.
### VLC
<code>$ sudo raspi-config </code></br>
"Enable Camera"</br>
<code>$ sudo apt-get update </code></br>
<code>$ sudo apt-get upgrade </code></br>
<code> sudo apt-get install vlc </code></br>
<code> sudo raspivid -o - -t 0 -n | cvlc -vvv stream:///dev/stdin --sout '#rtp{sdp=rtsp://:8554/}' :demux=h264 </code></br>
<code> rtsp://IP:8554/ </code></br>
** 현재 Usage 1은 오류있음, 추후 확인할예정

## Usage 2.
### V4L2 
#### Feature:
V4L2를 설치 후 활성화 시에 사용자 프로그램이 커널을 통해 I/O요청을 확인하고 장치 드라이버로 전송이 이루어지는 것이 가능해진다. 
사용자 프로그램이 커널을 통해 시스템 하드웨어에 접근 할 수 있도록 “dev” 디렉토리 내에 “video*"라는 장치 파일이 생성이 이루어진다. 
사용자는 이러한 “/dev/video”을 통해 자료를 읽거나 기타 장치로 자료를 전송이 가능해진다.

#### Session config:
v4l2rtspserver / dev / video0 : RTP 비디오 캡처 기능이있는 하나의 RTSP 세션 V4L2 장치 / dev / video0

<code>$ sudo raspi-config </code></br>
"Enable Camera"</br>
<code>$ sudo apt-get update </code></br>
<code>$ sudo apt-get upgrade </code></br>
<code>$ sudo apt-get install liblivemedia-dev libv4l-dev cmake libasound2-dev </code></br>
<code>$ sudo apt-get install v4l-utils </code></br>
<code>$ sudo modprobe bcm2835-v4l2  //v4l2 드라이버 활성화 </code></br>
<code>$ git clone https://github.com/jin95/IoT_Project_01.git </code></br>
<code>$ cd RaspberryPi-StreamingServer </code></br>
<code>$ cd RTSP </code></br>
<code>$ tar -xvf h264_v4l2_rtspserver.tar </code></br>
<code>$ cd h264_v4l2_rtspserver </code></br>
<code>$ sudo cmake . </code></br>
<code>$ sudo make </code></br>
<code>$  sudo ./v4l2rtspserver -F 25 -W 1280 -H 720 -P 8555 /dev/video0 & </code></br>

공식 V4L2 드라이버 bcm2835-v4l2
* 주의 : <code>$ sudo modprobe bcm2835-v4l2 </code>가 안될 시 장치 인식이 제대로 안되고 있다는 소리. 재부팅 시도.

#### Reboot 후 사용법:
<code>$ sudo modprobe bcm2835-v4l2 </code></br>
<code>$ cd h264_v4l2_rtspserver </code></br>
<code>$ sudo cmake . </code></br>
<code>$ sudo make </code></br>
<code>$  sudo ./v4l2rtspserver -F 25 -W 1280 -H 720 -P 8555 /dev/video0 & </code></br>
</br>
</br>
<code>rtsp://192.168.1.100:8555/unicast</code>

## RTSP
원리: 
<p align="center"> 
<img src="/img/RTSP.JPG" width="40%"></img>
</p>
<pre>
RTSP 규약은 HTTP 규약하고 비교해 볼 때, 문법이나 동작이 비슷하다. 
하지만, HTTP 가 무상태형(stateless)인 반면에 RTSP는 상태형(stateful) 규약이다. 
임의의 세션 ID는 세션 추적할 때마다 사용되는데, 이 방법은 영구 TCP 연결을 필요로 한다. 
RTSP 메시지는 클라이언트에서 서버로 간다. 만약, 서버에서 오류가 발생한다면 서버는 오류에 대한 응답 코드를 클라이언트로 보내준다. 
기본적인 RTSP 요청 메시지는 아래와 같고, 기본 포트는 554번이다.
</pre>

### RTP
원리: 
<p align="center"> 
<img src="/img/RTP.JPG" width="40%"></img> <img src="/img/RTPPacket.JPG" width="40%"></img>
</p>
<pre>
RTP 주요 기능 (※ UDP에서 지원하지 못하는 여러 기능을 제공하게됨)
  ㅇ 시간정보 제공 기능
     - 순서번호(Sequence number)의 보장 기능
       . 패킷 손실 검출, 패킷의 순서를 재구성 등
     - 내부적인 타임스탬프(Timestamp,발송시간) 전송
       . 수신측이 데이터를 적절한 시간순서 및 시간 내에 재생할 수 있도록
         데이터 스트림에 타임스탬프 정보를 추가함
       . 단일 또는 복합 매체(다중화된 정보열)에서 동기 및 지연에 대한 계산을 할 수 있게함
  ㅇ 정보매체의 동기화 기능
     - 데이타 타입에 대한 정보 제공 (소스동기 : Payload type)
       . 비디오 또는 오디오 등에 대하여 어떤 부호화 방식을 채택했는가를 알려줌
     - 각 미디어 스트림에 식별번호 부여 (미디어소스동기 : SSRC,CSRC)
       . 여러 스트림을 단일 스트림으로 혼합할 때 각각을 식별
  ㅇ 각 프레임의 경계 구분
     - 응용프로그램(응용계층)이 응용층 데이터 단위(ADU,Application-level Data Unit)를 구분
     - RTP 계층이 이 ADU 경계를 유지하며 전달 함
</pre>


