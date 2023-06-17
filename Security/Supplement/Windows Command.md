# Windows Comnmand

## dir
디렉터리의 파일 및 하위 디렉터리 목록을 표시하는 명령어
```
dir [<drive>:][<path>][<filename>] [...] [/p] [/q] [/w] [/d] [/a[[:]<attributes>]][/o[[:]<sortorder>]] [/t[[:]<timefield>]] [/s] [/b] [/l] [/n] [/x] [/c] [/4] [/r]
```
* 매개변수 없이 사용하는 경우 디스크의 볼륨 레이블 및 일련 번호와 디스크의 디렉터리 파일 목록을 표시함
* 중요 매개변수
    | 매개변수 | 설명 |
    |-----|------|
    | [\<drive\>:][\<path\>] | 목록을 보려는 드라이브 및 디렉토리를 지정함 |
    | [\<filename\>] | 목록을 보려는 특정 파일 또는 파일 그룹을 지정함 (여라 파일 이름 매개변수를 사용하기 위해서 공백, 쉼표, 또는 세미콜론으로 구분함) |
    | /q | 파일 소유권 정보를 표시함 |
    | /a[[:]\<attributes\>] | 지정된 특성이 있는 해당 디렉터리 및 파일의 이름만 표시함 <br/> &nbsp;&nbsp; - 해당 매개변수를 사용하지 않으면 숨겨진 파일과 시스템 파일을 제외한 모든 파일의 이름을 표시함 <br/> &nbsp;&nbsp; - 특성(attributes)를 지정하지 않고 사용하는 경우 숨겨진 파일곽 시스템 파일을 포함한 모든 파일의 이름을 표시함 |
    | /s | 지정된 디렉터리와 모든 하위 디렉터리 내에서 지정된 파일 이름의 모든 항목을 나열함 |
* dir 명령어의 자세한 설명은 [dir Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir)를 참고


<br/>

## type
파일의 내용을 표시하는 명령어
```
type [<drive>:][<path>]<filename>
```
* 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | [\<drive>\:][\<path\>]\<filename\> | 보려는 파일 또는 파일의 위치와 이름을 지정함 <br/> &nbsp;&nbsp; - \<filename\>에 공백이 포함된 경우 따옴표(")로 묶어야 함 <br/> &nbsp;&nbsp; - 여러 파일 이름 사이에 공백을 추가하여 나열할 수 있음 |
* type 명령어의 자세한 설명은 [type Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/type)를 참고

<br/>

## cd
현재 디렉터리의 이름을 표시하거나 현재 디렉터리를 변경하는 명령어 (chdir 명령어와 동일함)
```
cd [/d] [<drive>:][<path>]
cd [..]
chdir [/d] [<drive>:][<path>]
chdir [..]
```
* 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | /d | 드라이브에 대한 현재 디렉터리와 현재 드라이브를 변경함 |
    | \<drive\>: | 드라이브 간 이동을 위해 사용함 |
    | \<path\> | 이동하려는 디렉터리의 경로를 지정하면 지정한 경로로 현재 디렉터리를 변경함 |
    | .. | 상위 디렉터리로 이동함 |
    | \ | 루트로 이동 |
* cd 명령어의 자세한 설명은 [cd Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cd)를 참고

<br/>

## del
하나 이상의 파일을 삭제하는 명령어(지우기 명령어와 동일한 작업을 수행함) <br/> ⚠️ *Warning: del을 사용하여 디스크에서 파일을 삭제하는 경우 파일을 검색할 수 없음*
```
del [/p] [/f] [/s] [/q] [/a[:]<attributes>] <names>
```
* 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | \<names\> | 하나 이상의 파일 또는 디렉터리 목록을 지정함 <br/> &nbsp;&nbsp; - 여러 파일을 삭제하려면 와일드카드(```*```)를 사용할 수 있음 <br/> &nbsp;&nbsp; - 디렉터리를 지정하면 디렉터리 내의 모든 팡리이 삭제됨
    | /p | 지정된 파일을 삭제하기 전에 확인 메시지를 표시함 |
    | /f | 읽기 전용 파일을 강제로 삭제함(force) |
    | /s | 지정된 현재 디렉터리와 모든 하위 디렉터리에서 파일을 삭제함 (삭제될 파일의 이름을 표시함) |
    | /q | 자동 모드를 지정함 (삭제 확인 불필요) |
    | /a[:]\<attributes\> | 파일 특성(attributes)에 따라 파일을 삭제함 |
* del 명령어의 자세한 설명은 [del Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/del)를 참고

<br/>

## move
한 디렉터리에서 하나 이상의 파일을 다른 디렉터리로 이동시키는 명령어
```
move [{/y|-y}] [<source>] [<target>]
```
* 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | /y | 기존 대상 파일을 덮어쓸지 확인하는 메시지를 중지함 |
    | -y | 기존 대상 파일을 덮어쓸 것인지 확인하는 메시지를 표시하기 시작함 |
    | \<source\> | 이동한 파일의 경로와 이름을 지정함 <br/> &nbsp;&nbsp; - 디렉터리를 이동하거나 이름을 바꾸려면 source가 현재 디렉터리 경로 및 이름이어야 함 |
    | \<target\> | 이동할 파일의 경로와 이름을 지정함 <br/> &nbsp;&nbsp; - 디렉터리를 이동하거나 이름을 바꾸기 위해서는 target이 대상 디렉터리 경로 및 이름이어야 함 |
* move 명령어의 자세한 설명은 [move Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/move)를 참고

<br/>

## copy
다른 한 위치에서 하나 이상의 파일을 복사하는 명령어
```
copy [/d] [/v] [/n] [/y | /-y] [/z] [/a | /b] <source> [/a | /b] [+<source> [/a | /b] [+ ...]] [<destination> [/a | /b]]
```
* 중요 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | \<source\> | [**필수**] 복사할 파일 또는 파일 목록을 지정함 (드라이브 문자 및 콜론, 디렉터리 이름, 파일 이름 등으로 구성될 수 있음) |
    | \<destination\> | [**필수**] 파일 또는 파일 목록이 복사될 위치를 지정함 (드라이브 문자 및 콜론, 디렉터리 이름, 파일 이름 등으로 구성될 수 있음) |
* copy 명령어의 자세한 설명은 [copy Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/copy)를 참고 

<br/>

## ipconfig
모든 현재 TCP/IP 네트워크 구성 값을 표시하고 DHCP(Dynamic Host Configuration Protocol) 및 DNS(Domain Name System) 설정을 수정하는 명령어
```
ipconfig [/allcompartments] [/all] [/renew [<adapter>]] [/release [<adapter>]] [/renew6[<adapter>]] [/release6 [<adapter>]] [/flushdns] [/displaydns] [/registerdns] [/showclassid <adapter>] [/setclassid <adapter> [<classID>]]
```
* 중요 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | /all | 모든 어댑터의 전체 TCP/IP 구성을 표시함 |
    | /displaydns | DNS client resolver cache 내용을 표시함 (DNS client 서비스가 DNS Server에 쿼리하기 전에 이 정보를 사용하여 자주 쿼리되는 Domain Name을 빠르게 확인함) |
    | /release [\<adapter\>] | DHCP 서버에 DHCPRELEASE 메시지를 전송하여 현재 DHCP 구성을 해제하고 어댑터에 대한 IP 주소 구성을 삭제함 (\<adapter\>가 포함된 경우 해당 어댑터만 삭제, 아니면 전체 어댑터를 삭제함) <br/> &nbsp;&nbsp; - IP 주소를 자동으로 얻도록 구성된 어댑터에 대해 TCP/IP를 비활성화함 |
    | /renew [\<adapter\] | 어댑터에 대한 DHCP 구성을 갱신함 (\<adapter\>가 포함된 경우 해당 어댑터만 갱신, 아니면 전체 어댑터를 갱신함) <br/> &nbsp;&nbsp; - IP 주소를 자동으로 얻도록 구성된 어댑터가 존재하는 컴퓨터에서만 사용할 수 있음 |
    - 매개변수 없이 사용하면 모든 어댑터에 대한 IPv4 및 IPv6 주소, 서브넷 마스크 및 기본 게이트웨이를 표시함
* ipconfig 명령어의 자세한 설명은 [ipconfig Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig)를 참고

<br/>

## set
cmd.exe 환경 변수를 표시, 설정 및 제거하는데 사용하는 명령어
```
set [<variable>=[<string>]]
set [/p] <variable>=[<promptString>]
set /a <variable>=<expression>
```
* 매개변수
    | 매개변수 | 설명 |
    |---|------|
    | \<variable\> | 설정하거나 수정할 환경 변수를 지정함 |
    | \<string\> | 특정 환경 변수와 연결할 문자열을 지정함 |
    | /p | \<variable\>의 값을 사용자의 입력값으로 설정함 |
    | \<promptString\> | 사용자에게 입력하라는 메시지를 지정함 (/p 매개변수와 함께 사용함) |
    | /a | \<string\>을 평가하는 숫자 표현식(numerical expression)으로 설정함 |
    | \<expression\> | 숫자 표현식을 지정함 |
    - 매개변수 없이 사용하는 경우 현재 환경 변수 설정을 표시함
* set 명령어의 자세한 설명은 [set (environment variable) Documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/set_1)을 참고


<br/><br/><br/><br/>
### 🔖 출처
* 📚 [Windows Server Documentation - windows command](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir)
