[toc]

파일 및 디렉토리 탐색 기능을 가진 프로그램을 제작하는 프로젝트 2주의 제한기간
---
MFC, C++, Sqlite3 를 활용한 프로젝트이며 처음으로 MFC 및 C++ 를 활용한 프로젝트이다.
NTFS file system에 직접 접근하여 모든 Data를 가져오는것을 중점으로 둔 프로젝트이다.
***
# NTFS
## Data Read
```
NTFS 의 MFT 을 읽어와 각각의 Entry 내부의 Attributes를 확인해 정보를 불러온다.
이는 구조체를 통해 읽어오며 각각의 구조체는 하단의 링크 정보를 기반으로 구축하였다.
file:///C:/Users/TestKim/Desktop/NTFS%20Cheat%20Sheet.pdf
```
```
현 프로그램은 Data Read 할 때 직접 Record no 를 계산하는것이 아닌, 저장된 정보를 읽어오는 방식으로 동작하기에
NTFS version 3.1 이상부터 동작한다.
이하버전에서 동작하려면 Record no 를 직접 계산하는 방식이 필요하다($MFT 가 Record 0번, 순차적으로 증가)
```
---
# WinApi 와 비교
## Data Read
```
WinApi를 통해서도 MFT 를 읽는것 처럼 파일,디렉토리 정보를 읽어올 수 있다.
하지만 이 방법은 각 파일, 디렉토리별로 디스크 read 를 직접 진행해야되기 때문에 일정 수치 이상의 최적화가 될 수 없다.
또한 일반적으로 재귀 함수 방식으로 구현되기 때문에 메모리 리스크또한 높으며 전체적인 실행 속도가 느리다.

실제 직접 구현해서 비교한 결과
NTFS 프로그렘 데이터 Read 시간 : 5초
WinApi 프로그렘 데이터 Read 시간 : 1분 7초

프로그램 구축 실력이 떨어진다 하더라도 두 프로그램 제작자가 본인으로 동일함을 감안하면
실제 엄청난 차이가 존재했으며, 이 차이는 최적화를 하면 NTFS Read 방식은 더 빨라지나 WinApi는 일정 이상 줄어들지 않는다.

이러한 결과를 본다면, File Read 를 할 때 Window이고, NTFS 임을 보장받는다면 MFT를 활용하는것이 가장 최선의 방식이라 확신한다.
```
---
# Others Algorithm & MFC

## Data Sort
```
Sort 알고리즘은 병합 정렬을 사용하였다.
평균적으로 가장 빠른 속도를 내는 알고리즘이기 때문이다.
```

---

## Data Search
```
Search 알고리즘은 KMP 알고리즘을 사용하였다.
KMP, Trie, Rabin-Karp, Turbo Boyer-morere 를 각각 비교하였고

KMP
O(N + M), 실패함수 배열 사용, 메모리 추가사용

Trie
생성 : O(N * M), 가장 빠른 실행속도, 중간문자열 탐색에 사용할 수 없음
탐색 : O(L) 

Rabin-Karp
최상 : O(N + M), 해시함수 사용, 해시 충돌이 발생할 경우 성능이 낮아짐, 파일 시스템 특성상 중복허용
최악 : O(N * M) 

Turbo boyer-morore
평균 : O(n / m), 검색대상의 배열 추가, 실패함수 추가
최악 : O(N + M)
검색 대상의 배열을 추가로 가지고 있어야되기 때문에 200만개 이상의 추가 데이터를 지니기가 부담스러움
DB에서 사용하는 방식


모든 상황에서 O(N + M) 의 실행속도를 보이며, 사용 가능한 KMP 를 선택하기로 함
```
---
## Synchronize 
```
동기화는 USN journal를 활용하기로 결정
하지만 시간상의 문제로 제대로 분석하지는 못하였고, 동작 및 사용만 가능하도록 구현

Thread 를 통해 Main 과 분리하여 동작하도록 결정
```
---
## Database
```
현재 프로그램에서 Database의 필요한 기능은 따로 존재하지 않는다. 따라서 Database의 장점이 없지만,
공부를 위한 프로젝트이기 때문에 Sqlite3를 활용하도록 한다.
Data를 저장할 때 깨지는 글자가 있어 insert 가 되지 않는경우가 발생해 bind를 통해 구현하였다.

Serialization을 활용해 구현을 고민하였고, 단일 프로젝트로 확장성을 띄지 않는다면
Serialization 을 활용하는것이 더 좋다고 판단하였으나 공부상의 이유로 Database Sqlite3를 활용하기로함
```
---
## MFC
```
현 프로젝트에서 사용한 List는 OwnerData 옵션의 CListCtrl 을 제작하여 진행하였으며
이로인해 화면 출력속도가 빠르다.

아직 MFC에 익숙하지 않아 Document 에 제어를 담아두지 않고 각각의 파일에 흩어져있다.
```

---

## 실행방법
```
다운받아서 실행한다.
64bit 기준으로 작성되었고, 32bit로 돌아가는것은 보장하지 않는다.
Sqlite3 Database가 설치되어있어야하며, 직접 path 를 설정에서 수정해야 한다ㅇ
```
---
## 성능
```
CPU : i7-7700 기준 평균 14% 사용
Memory : 52만개 데이터 기준 307MB

개선점
현재 동기화를 위한 방식으로 USN 저널을 무한루프로 읽어오고있다.
이를 변경시점마다 호출 및 FileRead부분을 없애고, User가 필요로하는 디렉토리로 제한한다면 cpu 점유가 낮아지리라 예상

쓰레드를 활용해 data read 를 효과적으로 구현가능
```
---
# 맺으며
```
현 프로젝트를 진행하며 C++에 친숙해지며, Disk에서 받아오는 구조체를 직접 설정해보는 경험을 해보았다.
```

### 프로젝트 진행하며 메모했던 사항

정리하지 않고 그냥 올려두도록 한다.
***
```
# Everything 초기정보

첫 로딩시간 약 5초

첫 초기화는 무조건 5초정도인가?
무조건 5초정도 이다 시작 프로그램에 등록해
첫 로딩자체를 컴퓨터 부팅과 동시에 진행하기 때문에 유저가 느낄 수 없다.
단, x 버튼을 눌러도 종료되지 않는다. 이는 로딩 시간을 최소화하기 위함이다.

또한, sqliteDB를 활용하여 첫 db 생성 이후에는 속도가 빠르다.
또한 프로그램 종료 후 변경사항이 유지되는것으로 보아
아주 빠르게 변경사항을 입력하는것을 확인할 수 있고, 이는 아마 폴더 해쉬를 사용하는것 같다.
폴더 변화를 각 폴더 해시를 사용해 판단한 뒤 
필요한 폴더만 변경사항 탐색을 진행하자.
-> ntfs usn journal 값으로 변경사항 탐색 결정


설치시 설정파일 위치와 설치파일 위치의 모든 파일을 삭제해도 동작에 문제생기지 않는다.
이는 메모리에 있는 데이터를 어디 저장하지 않고 계속 실시간으로 들고 있다는 의미
->프로그램 종료시에 DB에 현재 상태를 저장한다.


정리
에브리띵은 모든 데이터를 메모리에 저장하고 사용한다.
이는 동작 안하는 상태의 메모리 점유율 53.2mb로 저장 공간의 효율이 좋다.

매번 실행할때 마다 프로그램 정보를 다 긁어온다.
받아오는 순간 cpu를 20퍼까지 먹는것을 확인했음
76메가의 메모리를 먹는것을 확인했음
76->53
메모리를 먹으며 리스트를 생성하고, 과정에 생긴 다른 공간들을 할당 해제해준것으로 확인
일정 수준 이상의 길이를 검색했을 때 메모리가 줄어드는것으로 보아
문자열 비교 이전에 길이를 가지고 비교 하는것을 생각해볼 수 있음

기본적인 설계상 원래는 자료구조와 ui를 분리하는게 맞으나, (강의내용 분리해라)
이번 프로그램의 특징상 굳이 분리하지 않고, listview 구조에 속하도록 작성하도록 한다.

코드의 오버헤드를 고려해 복사가 일어나지 않게 하기 위해서는 포인터로 구성하는게 최적의 조건이다

기본적으로 사용하는 메모리가 53인것을 고려해 내부 자료구조가 어떤형식으로 되는지 유추를 진행함
이는


typedef struct dir_property
{
	char	*name;
	char	*path; //경로에는 파일명이 속하지 않는다.
	int		size; //size가 -1일경우에는 디렉토리, 
	DWORD	date;
};


로 진행했을 때 평균 이름 크기 15글자, path 길이를 65글자로 진행했을 때 나오는 크기와 비슷함 
name포인터 8바이트, + path포인터 8바이트, size 변수 4바이트
data변수 4바이트, + name 문자열 16바이트, path 문자열 66바이트로
총 106바이트가 1개의 크기, 50만개의 자료를 담았을 때 53메가가 나오는것을 확인할 수 있었다.
하지만 String 을 활용할경우 크기를 약 70메가 바이트까지 할당하는것을 계산해볼 수 있다.
string 의 일반적인 두배씩 값을 할당
실제 동작할때 또한 70메가 바이트 이상 까지 올라가는것을 확인해보면 실 동작에서 Cstring을 활용하고
저장할 때는 char *로 활용하리라 짐작해볼 수 있다.

프로그램을 실행시켰을 때 변경 이력을 감지하는 방법으로
1. 해쉬값 활용 -> 디렉토리의 해쉬값을 사용해 어느 디렉토리가 변경되었는지 추적
2. 매번 새로 받아오기
3. NTFS USN journal 활용
세가지를 고려하였고, 2번은 너무느려서 제외하였다.
1번의 경우 혹시 윈도우에서 제공하는 해쉬값이 있다면 매우 빠른 속도로 처리가 가능할것으로 생각해
찾아보았으나, 실제로 해쉬함수를 제공할지언정 해쉬값 자체를 제공하는것이 아니였기 때문에 
해쉬값 생성의 시간이 오래걸릴것으로 예상해 NTFS USN journal 값을 활용하는것이 가장 적합하다고 판단

filesystem read 하는 과정을 지난번에 mfc라이브러리를 사용하면 안되지만 CFILe 을 사용해버렸고,
이번에 filesystem(C++라이브러리)를 고려해 시간 테스트를 해보았다.
실제로 전체 정보를 재귀적으로 받아오기만하는데(출력x) 걸리는 시간은 1분 7초 가량이 걸리는것을 확인할 수 있었다.(오차 약 1초)
현재 50만개의 데이터를 받아오는데 걸리는시간이 이정도임을 고려한다면 100만개의 데이터는 시간이 더 느릴것으로 예상된다.
이에 속도를 더 빠르게 할 수 있는 방안에 대해 고려해본 결과
filesystme(NTFS)에서 meta data를 활용해 파일 정보를 읽어오는 부분을 생각하게 되었고,
이는 metadata에 있는 정보를 가져와 바이트 단위로 끊어서 필요정보를 파싱할 수 있다면 모든 정보를 긁어오는데
1초도 걸리지 않을것으로 예상한다.
-> https://blog.naver.com/bitnang/70184788707 NTFS의 인덱스와 파일구조에 대해 설명하는 링크이다.


검색시 cpu 사용량증가를 통해 검색 알고리즘 유추
실제로 검색을 진행할 때 cpu 변동량을 확인할 수 있었는데
한글로 검색을 진행할 경우 5%대를 유지할 수 있었고
영어는 평균적으로 2%대를 유지하는것을 확인할 수 있었다.
아마 언어를 변경하는 과정에서 특정 행동이 들어가는것으로 유추해볼 수 있다.
또한 길이를 활용해서 검색을 진행하는것이 아닌것을 확인할 수 있었다.
-> 검색 속도가 비정상적으로 빠른데 이거 db를 활용해서 쿼리문을 사용하는것이 맞는가하여
실제 공식 홈페이지로 가서 확인해본 결과 맞다 db를 사용해 인덱스를 활용하는방식을 보이고 있었다.


1. 첫 실행시 파일 시스템을 통해 모든 데이터를 가져오는데 이 시간을 5초에 해결한다.
2. 두번째 실행부터는 파일 시스템호출이 아닌 한번에 모든 db 데이터를 불러오는 방식을 사용한다.
3. 검색할 때 총 길이를 기준으로 검색이전 필터링을 거친다.총 문자열 길이보다 긴 검색어면 검색을 할 필요가 없다.
4. 프로그램은 종료되지 않고, 백그라운드에서 지속적인동작을하기 때문에 강제종료로만 종료가 가능하다.
5. 정렬은 폴더, 파일 순서로 정렬하며, 모든 폴더의 정렬 후 파일의 정렬이 들어간다
뿐만아니라 휴지통에 있는 파일까지 검색해 화면에 표출하는것을 확인할 수 있고, 휴지통에 있는 파일의 경우 크기가
나오지 않아 0으로 속해 디렉토리로 표기되게 되는데 이건 가능하면 진행할 사항이다.
6. DB를 구축하고, 저장하는건 프로그램이 종료되는 시점에 진행을 하는데 이는 프로그램을 강제종료시켰을 때 db 가 저장되지 않고
실행할때마다 시간이 소요되는것으로 파악하며, 동작중에는 새로운 파일을 생성하지 않는(프로그램상에서)것을 직접 확인하였다.
%APPDATA%\Eeverything 에 저장되는것으로 유추
7. NTFS USN journal, ADS 두개를 활용해서 탐색 및 재시작의 동기화 진행


NTFS USN journal 사용법
https://stackoverflow.com/questions/46978678/walking-the-ntfs-change-journal-on-windows-10

받아오는 정보 USN_RECORD_V2 내부 항목
https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-fscc/d2a2b53e-bf78-4ef3-90c7-21b918fab304
-> 3일간의 파일변경된 모든 로그를 가지고 오는데 걸리는시간 == 0.1 초 와..


프로그램 실행 -> if 백그라운드 동작 
true -> 화면 창 실행
false -> 백그라운드, 화면창 두개 다 실행

프로그램 실행 -> db 가 있는가?
있다. -> db에서 정보 전부 가져오기
없다. -> filesystem에서 정보 전부 가져오기




테이블 구조
모두 한 테이블에 저장
이름, 경로, 크기, 수정한 날짜 보관

쓰레드로 동기화해 받아오는 정보는 실시간으로 반영되어야한다.

date 형식 timestamp 사용해 날자 표현
https://couplewith.tistory.com/181

쓰레드를 만드는 방법
https://m.blog.naver.com/mincheol9166/220718941148
단 여기서 ThreadFunction 매개변수 _mothod 는 실행시킨 장소의 객체를 의미함으로
실행시킨 곳의 자료형으로 형변환하면 그대로 사용할 수 있다.


리스트 항목을 유저가 보는곳만 업데이트해서 보여주는 방법
https://genius-duck-coding-story.tistory.com/53

MFT 크기 얻는 방법
FSCTL_GET_NTFS_VOLUME_DATA 제어 코드를 사용하여 MFT의 크기를 얻을 수도 있습니다 .
https://github.com/MicrosoftDocs/sdk-api/blob/docs/sdk-api-src/content/winioctl/ni-winioctl-fsctl_get_ntfs_volume_data.md

윈도우 공식 사이트 NTFS 설명
https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)#ntfs-physical-structure
여기서 위치로 이동시켜볼 수 있다.

MFT 구조 설명 한국 블로그
https://lemonpoo22.tistory.com/217

MFT 정보 불러오는 API (처음에는 직접 디스크를 읽다가 발견함)
https://learn.microsoft.com/ko-kr/windows/win32/api/winioctl/ns-winioctl-ntfs_volume_data_buffer?devlangs=cpp&f1url=%3FappId%3DDev14IDEF1%26l%3DKO-KR%26k%3Dk(WINIOCTL%252FNTFS_VOLUME_DATA_BUFFER)%3Bk(NTFS_VOLUME_DATA_BUFFER)%3Bk(DevLang-C%252B%252B)%3Bk(TargetOS-Windows)%26rd%3Dtrue

MFT 속성정보 (LCN  에 대해 알아볼 수 있다.)
https://blog.naver.com/bitnang/70184584840

상단의 정보 세개를 조합하면 대략적인 MFT 의 형태를 다 볼 수 있고, 시작지점을 찾아가는 방법을 파악할 수 있다.
시작지점을 찾아가는 방법은 두가지가 있는데
LONGLONG mftZoneStartOffset = volumeData.MftStartLcn.QuadPart * bytesPerCluster;
LONGLONG mftZoneStartOffset = mftZoneStart + 1 * bytesPerCluster;
논리 클러스터 지점 * bytepercluster 
mftzonestart + 1 * bytespercluster
이렇게 두가지다


 
MftZoneStart

마스터 파일 테이블 영역의 시작 논리 클러스터 번호입니다.

MftStartLcn

마스터 파일 테이블의 시작 논리 클러스터 번호입니다.

NTFS 버전 3.1 이상에 mft index 부분이 포함되기 시작했다는 문서링크(깃허브)
https://github.com/libyal/libfsntfs/blob/main/documentation/New%20Technologies%20File%20System%20(NTFS).asciidoc


FIle read & 정렬에 있어 현재 구현된 버전은 2가지의 종류로 구현함 1번은 NFTS 를 활용해 구현하였고
여기서는 vector를 활용헤 인덱스와 fileinfo 구조체를 저장하고 있다. 하지만 windows api 를 활용해 구현한 버전에 있어서는 배열로 직접 만들어 활용하고 있고, 이는 차후 방식을 그대로 당겨와 적용이 가능한 방식이다.
- 사실 집에서 winapi를 활용해 코딩을 돌리면 터져서 집 코딩이 불가능해서 NTFS 의 보완을 진행했습니다. 그래도 값을 가져오는부분을 대부분 완성하였기 때문에 괜찮은것같습니다.

프로그램 실행 속도는 디버그 모드와 릴리즈 모드의 차이가 많이 나는데 실질적으로 릴리즈 모드를 활용하면 최적화가 진행되기 때문에 속도차이가 많이 남


어려웠던점 - 버전까지 고려해서 코딩을 진행해야된다는점
바이트 단위로 처리해야되어 매우 코딩의 난이도가 높았던 점
아직 처리못한 애러(닫기 문자열 출력) 부분의 성능이슈


현재 진행되고 있는 프로젝트의 경우 단일 쓰레드로 모든 연산을 진행하는것을 확인할 수 있있는데 실질적으로 runlist(MFT 데이터 분리단위) 당으로 쓰레드를 동작시킬 경우 속도가 쓰래드 갯수만큼 향상될 수 있는것을 예상할 수 있음 이유는 프로그램 동작의 대부분의 딜레이는 연산처리에서 발생하고 disk read 의 횟수자체는 적은편으로 일반적으로 20회가 채 되지않음 

쓰레드를 활용해 모든 데이터 처리를 그러한 방식으로 진행했을 경우 runlist 1개를 읽는속도에 모든 데이터를 읽어올 수 있게 됨으로, 사실상 Database에 저장할 필요 없이 매 실행마다 diskread 즉 초기화를 진행해도 상관 없을정도의 퍼포먼스를 가질 수 있으리라 예상할 수 있음

주말간 공부해보고 자료를 찾아본 결과 
동작하지 않은 이유는 여태까지 진행했던 프로젝트는 
NTFS 3.1 버전이상의 자료를 가지고 코딩을 진행하였고, 이 자료상 MFT no 가 추가되어있었다. 하지만, 실제로 대부분의 컴퓨터에서는 NTFS 버전이 낮거나 혹은 맞더라도 MFT no 가 제대로 존재하지 않았고(메모리에 올라가 있으나 모든 데이터가 명확하게 들어있지 않음 - 최상위 디렉토리 no 가 5가 아니게 설정됨 공식에서 5라고 명시)
그로인해 path 가 깨지고 name 이 없는상태가 발생하였다.
하지만 이번에 no를 저장되어있는걸 활용하는것이 아닌 크기를 통해 직접 계산 OR 카운팅을 진행해 직접 연결을 활용해 저장하였고, 이 애러는 해결할 수 있었다.


lnk 파일의 경우 윈도우에서 lnk확장자로 나오는것이 아니라 
이미지에 바로가기 이미지로 나오고 확장자 없는 이름을 가지고 있는것을 확인할 수 있다.
이는 현재 MFT 에서 lnk 파일을 lnk파일이라고 이름을 찍기 때문에 실행이 안되는것을 확인
예외처리를 추가해야 실행시킬 수 있는데 이 예외처리는 차후에 진행하도록 한다.

파일 read MFT 를 진행하며 가장 큰 잘못은 회사에서 사용 가능한 툴만 사용해서 체킹을 진행했다는 점이였습니다. 개인적으로 집에서 공부할 때 개인용 무료 툴을 사용했더라면 MFT의 구조를 더 빠르고 명확하게 알아볼 수 있었으나, 그 방법을 미처 생각하지 못해 전체적인 시간이 오래걸렸습니다. 1주차 마지막에 보기 시작한 개인 무료 툴은 MFT viewer 를 사용하여 전체적인 구조체 및 entry attribute 의 순서, 파일 존재등을 명확하게 파악할 수 있었습니다.

현재 파일이 everything 보다 많이 나오는 이유는
접근 권한이 없는 시스템 파일에 접근해 파일의 정보를 가져오기 때문입니다.

코드를 작성하며, 가독성을위해 enum 을 사용하는것에 대해 알게되었습니다.
여태까지는 저 혼자 프로젝트를 하거나, 팀 프로젝트를 하더라도 소규모르 비 전공자들과 함께하여 enum을 사용하지 않았으나 이번에 제대로 프로젝트를 진행하며 스터디를 할 때
enum의 중요성에 대해 알게되고, enum을 활용하게 되었습니다. 첫 활용이라 
모든곳에 enum이들어있는게 아닌 부분부분 빠져있는 부분이 있으나, 이러한 점은 익숙해짐에 따라 점차 잘 활용하게 되리라 생각합니다.

현재 정렬에 있어 약 180만개의 데이터에 뻗어 버리는 현상을 발견하였습니다.
지금 사용하는 알고리즘의 고도화가 필요할 것 같은데 메모리때문에 뻗는것인지 원인을 찾는 방법에 대해 더 알아봐야될것 같습니다. 시간이 부족해 터지는 이유를 명확히 특정짓지 못하였고, 시간이 난다면 알아보고 수정할 수 있으리라 생각합니다.

현재 MFT 를 읽어왔을 때 유저계정 디렉토리맨 뒤에 골뱅이가 붙는 현상이 있습니다.
이는 다양한 기능을 위해 붙는 문자로 알고있지만, 현재 프로그램에서 사용하지 않기 때문에 제거해주는 알고리즘이 필요합니다. (현재 구현 x 하드 코딩으로 진행)

sqlite 를 1000000만개단위로 읽어올 경우 프로그램이 터지는 현상을 발견
이에 분할해서 가져오는 방법을 선택
https://www.devkuma.com/docs/sqlite/limit-offset/
이 방식을 통해 현재 데이터를 10만개단위로 잘라서 읽어오는 과정이 있음

현재 MFC 원본 데이터에 직접 접근하여 읽어오는 방식을 취하고 있지만, 읽어온 데이터를
화면에 출력하는 과정에서 인코딩 문제가 발생하여 화면에 글자가 깨지는 현상이 발생
이는 인코딩 과정을 수정하면 괜찮아질것으로 예측중

현재 문제점
data read 과정에서 MFT 를 활용해서 읽어오는데 이 과정에서 path 가 없다.
Winapi file change read 를 활용하면 path 만 가져와서 파일 변경을 알려준다.
즉 적용이 안된다.

문제가 발생한 이유 : MFT 를 활용하며 다른 api와 호환 가능성을 염두에 두고 설계를 진행해야되었으나, 설계에서 미처 생각하지 못하였다. 시간이 있었어도 아마 생각하지 못했으리라 예측한다. 경험이 부족했다.

그럼 해결책은 : api 를 사용해서 전체를 동기화하거나
path 를 구현한다.

path 구현시 전체시간의 1/3 정도를 path 구현에 잡아먹음으로 쓰레드를 통해 구현하도록 한다.
쓰레드를 통해 구현했을 때 시간은 조금 빨라지리라 예측하지만, 어떤 문제가 발생할 지 모른다.
현재 제출까지 시간이 얼마 남지 않았음으로 쓰레드를 활용하지 않고 path 를 구현하도록 한다.

sqlite 에 값을 넣을 때 처음에는 일반적인 방식으로 값을 넣었으나
sqlite3_exec활용해서 넣었음
그러나 이 방식은 문자가 깨지는경우 인서트가 제대로 되지 않는 경우가 발생한다.
그래서 sqlite3_bind_~~ 를 활용해서 값을 넣어주는 방식으로 변경

MFC 에서는 READ_USN_JOURNAL_DATA 일반 c++ 에서는 READ_USN_JOURNAL_DATA_V0
로 서로 다른 이름으로 같은 구조체를 사용하고 있어서
테스트코드에서는 동작하는데 mfc에서는 동작하지 않는 코드가 되어서 
직접 라이브러리를 확인하고, 구조가 동일한것을 파악한 후 이름 변경을 통해 사용할 수 있었음
처음에는 구조체를 동일하게 만들어서 사용하려고 정의로 들어가서 확인하고 가져오던 중 혹시하는 생각에 버전을 지워보고 동일한 구조체가 다른 이름으로 존재하는것을 확인
버전차이의 중요성에 대해 더욱 깊게 느껴볼 수 있는 시간이였음


발생한 문제점 및 해결과정
처음에 MFT 의 Record no 을 거쳐서 path 를 만드는 과정을 직접 구현하는데 있어서 최상위 폴더에 속한 디렉토리가 모두 이름이 표출되지 않는 문제점이 있었음
이에 문제점을 찾던 중 Record no 5번과 최상위 디렉토리가 일치하지 않는 문제점을 발견 해결책을 고안하던 중
Record no 가 버전에 따라 저장되기도 하고 저장되지 않기도 한다는 점을 발견
이에 Record no 를 내부 저장된 값으로 활용하는것이 아닌 직접 카운트해서 사용하기로 함
하지만, 후에 USN 저널API를 활용해 파일의 변경사항을 받아오는 코드를 작성, 이 과정에서 직접 카운트한 Record no 와 받아온 Record no 이 다르다는것을 발견
이에 근본적인 해결을 위해 직접 하드디스크를 열어 주소값에들어가 각각의 값들을 확인해봄
USN 저널 API에서 받아오는 값이 정확하다는것을 확인할 수 있었음 -> 이말은 실제로 저장되는 MFT Record no 가 정확하다는 의미였고
직접 카운트하는 방법이 잘못되었고 전에 작성한 받아오는 코드에 문제가 있음을 확인
다시 받아오는 코드 작성하던 중 UINT 값을 QWORD 로 받아와 문제가 발생한것을 확인 다시금 두 값의 크기를 확인해본 결과 다르다는것을 알게되었고, 서로 다른 크기의 자료형을
memcpy 로 읽어오는 부분에 문제가 있다는것을 발견 읽어오는부분, 받아오는 부분 두 곳 전부 UINT 로 수정함으로 써 정상동작하도록 바꿈


코딩을 진행하며 있던 모든 애로사항들을 작성하기엔 양이 너무 방대해 굵직한 케이스로 작성하기로 결정
이에 맞추어 메모를 진행하였으


오늘 목표
생성과 동시에 path 저장, db에 path 저장 -> 출력시 path 찾아오는 부분 없앨것
실시간 변경 감지 및 저장







코딩해야될 부분
쓰레드를 활용한 실시간 동기화

현재 보완점
검색 후 정렬
크기, 경로, 수정날짜 순으로 정렬
검색 쓰레드 분리
이어서 검색
전체적인 확장성의 부족 (NTFS 외의 다른 파일시스템은? api를 활용하면 해결될것으로 보임)
->api 활용했을 시 코드는? 월요일 코드에 들어있음

발표할 때 제작할 피피티 내용
1. 처음 프로젝트 받았을 때 생각, 구현 방식
2. 설계서를 작성하며 변경된 생각, 구현 방식
3. 개발을 진행하며 느낀점, 변경된 생각, 구현방식
4. 개발을 완료하고 부족했던점, 개선점, 잘한점
5. 전체적인 코드의 압축버전 대부분은 설계서를 참고하고, 피피티에는 코드가 올라가서는 안되고

오늘 내일 진행할 사항

지금 db 에 들어가야되는 사항
nowupdate 라는 값으로 현재 시간을 받아서 넣어둬야된다.
현재 시간을 기준으로 언제 업데이트된 파일인지를 확인하고, USN 저널에서 그 시간, 날짜를 기준으로 업데이트를 진행해야된다.
실시간 탐지도 마찬가지이다.

데이터가 모두 로드된 이후로 쓰레드를 동작시켰기 때문에 없는 데이터에 접근하는 일은 없다. -> 쓰레드 분기점 위치 중요



모든 동작은 쓰레드를 활용해 진행되어야하며, 메인쓰레드에서는 초기화가 끝나면 읽기만 진행하기 때문에 thread safe 한 코드가 되리라 예상됨
동기화 쓰레드는 하나만 만들어 진행하며 이 쓰레드에서 오랜기간 및 실시간 동기화를 동시에 진행한다.

무한루프로 동작함 (sleep (1000) )
동기화 함수 매개변수(저장되어있는 과거 마지막 동기화(사용)시간)
{
	USN 저널을 통해 기록 받아오기;

	while (저널 끝 읽을 때 까지)
	{
		if (저장되어있는 과거시간 이후인가)
		{
			현재 사항을 동기화시킨다.
			-> map값 변경, vector 값 변경 등등
		}
		else 
			continue;
	}

}
이 방식을 사용하면 실시간 탐지와 여태까지의 모든 기록을 확인할 수 있다.
단점 : USN 저널을 활용하면 기본 크기인 128 ~ 512 둘중 하나로 설정되어있음 용량만큼 변경 사항이 저장되며
설정에 따라 최대 4gb 까지 변경사항의 로그를 남긴다. 하지만, 이러한 기록은 변경사항이 많아져 해당 용량을 모두 사용하였을 경우
가장 오래된 데이터부터 지우며 새로 데이터를 작성해나간다. 
그러한 사용 방식에 있어 용량을 확장하는 방안을 생각해보았으나, 이 usn 저널이 통재로 메모리에 올라가는 경우가 왕왕있다는것을 알게되었고(어떤 케이스인지 확인하지 않음)
이 경우 메모리에 최대값인 4기가가 들어간다면 큰 문제가 발생할것으로 보고, 기본 설정에서 변경하지 않으며, 기존 날자가 지난경우 데이터 전체 초기화를 통해 새로 받아오는것이 가장 안전할것으로 판단됨
이러한 사유로 USN 저널 크기를 변경하지 않고, 기본 크기로 사용하되, DB에 최종 동기화 시간을 저장하여 이 시간이 USN 저널 최초 시간보다 빠를경우 전체 데이터 초기화를 진행하기로 한다.
다른 방법이 있는지는 모르겟어나, 현재로써는 가장 적합한 방식이라고 고려됨
완벽하게 분석하지 않고 api를 사용한건 문제가 발생할 여지가 큼 하지만, 현재 주어진 시간동안 모든 부분을 완벽하게 분석할 수 없어 이 부분은 동작하도록 만들고 세부 분석은 넘어가도록함



기능적 개선사항
disk를 읽어오는 방식이 현재는 NTFS 로만 구성되어있지만, NTFS 파일 시스템이 아닌경우 api를 활용하는 코드를 추가한다. -> 구현된 코드가 존재
검색된 데이터의 정렬이 안되는 상황 -> 검색된 데이터도 정렬하면 된다. 검색별 정렬 인덱스 백터를 구성한다.
내부에 사용중인 맵 -> Unordered map 으로 변경하거나(성능), map을 사용하지 말고 해시함수를 직접 사용해 메모리 효율을 높인다.
vector -> vector는 사실 사용해도 될거같다는 판단을 하였으나, vector 를 사용하는것이 아닌 배열과 배열 사이즈를 저장하는 int형 변수를 활용하도록 한다.
이는 동적 배열을 의미하며 처음 배열 크기는 60만개 그 이후로 10만개씩 증가시키는 방안을 택하도록 한다.
유사한 코드(중복 x)를 합치는 방안을 고려해 코드의 크기를 줄여준다.
USN 저널부분은 명확하게 분석하고 사용한것이 아닌, 주어지는 api를 활용하여 진행한 코드이기에 분석시 더 성능을 높일 수 있는 방법이 있을것이다. 현재 비효율적인 부분은
동기화를 진행할 때마다 모든 저널을 읽어와서 파악하는 과정이 들어가며, 최신화 과정에서의 중복되는 과정을 없앨 수 있을것이다. 현재로써는 저널 분석이 완벽하지 않아 low단에서 읽어올 수 없어서 불가능하다.


path 구성 : 현재 MFT Record no를 활용해 path 를 직접 구성하는 방법을 취하고 있기에 window api의 changeDirectory api를 활용할 수 없다.
전체 파일의 path를 구성하는것에 시간이 너무 오래걸리는것을 확인할 수 있었고, usn journal를 활용하게 되었다. 하지만, 굳이 USN 을 활용하지 않아도
path를 빠르게 구성할 수 있다면 api를 활용하는것도 나쁘지 않다고 생각이 들었고, 가능하다면 path 받아오는 부분을 직접 빠르게 만들어 활용해보는것도 나쁘지 않은 생각이다.
코드의 전반적인 CString 사용 : CString 을 사용해 일반적인 string 보다 무겁고 코드가 복잡해졌다.(C언어 사용자 기준) 이는 개선의 여지가 명확하며 코드 내부에 사용되는
대부분의 CString을 string 으로 대체하여 사용하는것이 메모리상 효과를 보이리라 생각된다. 하지만
ClistCtrl에서 CString 을 사용하고있는데 변환이 많아져 시간적인 성능의 하락을 고려해야한다. (화면에 출력되는 아이템 갯수 및 옆의 슬라이드바를 잡고 내리는 순간 출력되는 화면의 성능에 영향)
현재 디렉토리와 파일을 동일하게 취급하고 있는데 화면에 출력할 때 디렉토리별, 파일별로 분류해야된다. 이는 현재 vector 하나에 모두 저장하고 있는것을 vector 두개로 나누어 저장하고
인덱스를 각각 보유하며, CListctrl 에 저장하고있는 list count 를 두 벡터의 합으로 저장하면 된다. 그리고 index에 맞추어 출력물을 보내면 해결된다. 하지만 아직 구현하지 못하였다. (능력이 부족해 시간이 부족함)

ClistCtrl 에서 이미지를 추가할 수 있는가? -> 이미지 추가 하는 방법을 연구해야된다...

현재 쓰레드로 구현되어있는 동기화를 프로세스 단위로 분리해 프로세스 1개당 cpu 점유율을 낮춰줌으로써 유저단에서 보이는 프로그램의 부하를 낮추어준다. ->사실 별로 티도안난다지금

현재 디비가 존재하는지 유무만 채크하기 때문에 db 값에 이상이 생겼을 경우를 파악할 수 없다. 예를들어 db만 만들어지고 내부 테이블이 없다거나 이런 경우에 문제가 발생할 수 있는데
이는 내부 검증하는 기능을 구상해서 넣으면 된다. 

데모코드와 같은 다른 사람들의 코드를 보던 중 궁금증이 생겼다. i++ 과 ++i를 혼용해서 쓰지않고 대부분 ++i를 사용하는데 혹시 속도차이가 있는게 아닐까
평소 i++만 사용하던 차 궁금증 해소를 위해 알아보니 i++의 경우 복사생성자가 호출되고, 이는 구조상 ++i 보다 느려질 수 밖에 없다라는것을 보고 
대부분의 코드에서 ++i를 기본으로 사용하는 이유에 대해 알게되었다. 이를 추천하는것은 구글 c++ style guide 에서도 확인해볼 수 있다.(https://google.github.io/styleguide/cppguide.html#Preincrement_and_Predecrement)

디버깅모드와 릴리즈모드에서 메모리 증가차이가 좀 난다.
디버깅 모드에서는 메모리가 한번 정해지면 더이상 증가하지 않는데 비해
릴리즈 모드에서는 계속 증가하였고, 한참을 기다려본 결과 일정 수치까지 올라가고 더이상 증가하지 않고 멈춰있는 현상을 확인할 수 있었다. 실제로 코드를 전부 확인하여 메모리 할당이 더이상 없게 만들었을 때 초기에 증가하는 현상에대해 명확히 정리할 수 없었다.
하지만, 개인적으로 생각해본 결과, 메모리가 증가한 이유는 단편화의 가능성이 높아보인다
코드 내부에 백터를 사용하는데 지속적으로 생성, 해제를 반복하기 메모리 단편화가 발생하는것으로 유추하고있다. 하지만 명확한 답은 나오지 않았다. 약 20분간 동작시켰을 때
메모리가 더이상 증가하지 않는것을 확인할 수 있었다.

현재 이벤트 알림을 받는부분을 구현하지 못해(API를 사용하지 못하였다)
쓰레드가 지속적으로 많은 일을 담당하게 되었다.그래서 cpu 사용량이 너무 높게 잡히는 현상이 있다. 이 문제를 해결하기 위해 찾아본 결과 디렉토리 변경 알림을 받아 그것이 실행될 때 마다 실행하는 방법이 있었다. 코드도 있었으나 코드를 분석하기엔 남은 시간이 얼마 없어 넘기도록 한다.
https://stackoverflow.com/questions/38885286/usn-nfts-change-notification-event-interrupt 를 참고

이벤트를 통해 쓰레드 종료를 진행하긴 하였으나, 완벽하지 못하다.
처음 생각했던 이벤트는 쓰레드가 동작중에 이벤트가 호출되면 정지되고 멈추는것을 예상하였으나,
실제로 코드상 이벤트 감지가 계속 들어가야되는것을 확인하였고, 이는 효율적이지 못한 방법임을 알게되었다. 아직까지 더 효과적인 방법을 찾지 못하였음으로 쓰레드 작업단위를 작게 분리하여 이벤트값을 받아내는 방식을 택하였다. 
이는 위험성이 있는 코드로 세밀하게 문제가 발생했을 메인 쓰레드의 메모리를 먼저 정리하고, 쓰레드가 멈추는 경우가 발생할 수 있어 설계적 문제임을 알게되었다.

문제점
기본적으로 설계를 진행할 때 구상할 점으로 공유 데이터를 어떻게 safe 하게 관리하냐인데
종료시점에 문제가 발생하는것을 예상하지 못하였고
두번째로 공유데이터가 너무 많이 발생하였다. 공유데이터의 예시로는 
list data, list sort data(6개), nowsort 변수(현재 정렬상태) 총 8개로 
thread에서 동시에 봐야하는 데이터는 listdata, nowsort, nort index 배열 총 세개로
이들을 한번에 처리하지않으면 문제가 발생한다.

현재 쓰레드를 view 에 넣어서 관리하고있는데
쓰레드가 clistctrl에 들어가서 관리를 하게 된다면 더욱 짧고 가독성 높으며 안전한 코드가 될 수 있다. -> 소멸자 호출 시 쓰레드 종료 등


디비에 마지막 최신화 시간을 저장 (디비기능 쓰지말고 직접 저장해야됨)
언제 시간으로 저장한다? USN 저널 마지막에 있는 시간으로 저장해라

디비에서 시간을 불러와서 체크
1. 디비에 저장된 시간이 USN 저널에 있는 시간보다 빠른 시간이다.
-> 디비 날리고 새로만들어라
2. 디비에 저장된 시간이 USN 저널에 있는 시간보다 느린 시간이다.
-> 저널을 기준으로 시간체크하며 최신화 진행하면 된다.


부모 디렉토리 찾아가는거 보는법
https://hec-ker.tistory.com/329


NameSpace 종류 및 저장방식
https://blog.naver.com/bitnang/70184584840



어려웠던점: 크기 출력
어떤 자료형(비트 바이트 키로바이트)로 저장되어있는지 문서가 없다.
모든 자료형으로 해도 안맞아서
왜그런가 하고 알아보니 memcpy 로 받아 넣은게 아니라 빅엔디안으로 계산되어 값을 받아오고 있어서 제대로 되지 않았다.

시간 계산도 무엇이 기준인지 찾는게 어려웠지만 코드에 주석으로 달아두었다.

전체적으로 보았을 때 이제는 file data read에서 시간이 오래걸리는것이 아니라
내부 데이터가 몇십만개가 넘어감에 따라 가공에서 시간이 오래걸리는것을 알 수 있다.
그렇다면 각각의 데이터 가공을 필요할 때 마다 진행하는것이 가장 속도적인 이득을 볼 수 있는데
이는 time에서 시간을 많이 먹는것으로 확인하였기에 시간이 난다면 저부분을 미리 만들어서 저장하는것이 아니라.
ownerdata cListCtrl 을 활용하고 있으니 각각의 데이터를 요청하는 타이밍에 맞추어서 값을 계산해서 전달해준다면
훨씬 빠른속도의 계산이 가능하리라 생각한다. -> 단 이를 진행하기 위해서도 어차피 시간 데이터를 가지고 있기 때문에
속도적인 측면은 개선될지 모르나 공간적인면은 개선되지 않는다.
처음 : 파일을 읽어올 때 시간을 계산해 저장해놓는다 -> 14초 (path 없는상태)
개선 : 파일을 읽어온 그대로 저장하고, 화면 출력시 계산한다. -> 8초(path 없는상태)
획기적인 시간 개선이 가능하였다.
정렬시간 34초 -> 개선 구상방안 -> 병렬정합을 활용하고 있는데 첫번째 잘라낸 부분에서 두개의 쓰래드를 활용한다면?



path 를 추가했을때 딜레이 되는시간은 4초인것으로 확인
앞서 확인한 path 없는상태가 8초인것을 감안하면 성능의 50퍼센트를 잡아먹는것
이를 해결하고싶으나, 시간상의 문제로 넘어감. 해결방안 = mymalloc 을 만드는것 (malloc 할당의 느린 속도를 대채하기위해
system call 수를 줄인다 실제 55만번의 시스템콜을 10회 미만으로 줄일 수 있는 방법)
이게 진행된다면 현제 4초를 2초내로 줄일 수 있으리라 예상

path 가 없는파일들 : $로시작하며 부팅 혹은 파일시스템과 관련된 정보들 경로도 없는 그냥 파일이다.

https://blog.naver.com/jsky10503/221361470898

```
