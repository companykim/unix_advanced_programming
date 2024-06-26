# chapter 4 파일과 디렉토리

## 4.2 stat, fstat, lstat 함수

    #include <sys/stat.h>

    int stat(const char *restrict pathname, struct stat *restrict buf);

    int fstat(int filedes, struct stat *buf);

    int lstat(const char *restrict pathname, struct stat *restrict buf);

stat 함수는 pathname으로 주어진 경로이름에 해당하는 파일에 대한 정보를 담은 구조체를 둘째 인수를 통해서 돌려준다. </br>
fstat 함수는 파일 서술자 filedes에 대해 이미 열려 있는 파일에 대한 정보를 조회한다. </br>
lstat 함수는 stat와 비슷하되 지정된 파일이 기호 링크(symbolic link)이면 그 링크가 가리키는 파일이 아니라 그 링크 자체에 대한 정보를 돌려준다. </br>
이 함수들의 둘째 인수는 특정한 구조체를 가리키는 포인터로 함수를 호출하기 전에 그 구조체를 미리 준비해 두어야 한다.
    struct stat {
      mode_t      st_mode;       /* 파일 종류 및 모드(접근 권한) */
      ino_t       st_ino;        /* i노드 번호(일련 번호) */
      dev_t       st_dev;        /* 장치 번호(파일 시스템) */
      dev_t       st_rdev;       /* 특수 파일들에 대한 장치 번호 */
      nlink_t     st_nlink;      /* 링크 개수 */
      uid_t       st_uid;        /* 소유자의 사용자 ID */
      gid_t       st_gid;        /* 소유자의 그룹 ID */
      off_t       st_size;       /* 바이트 단위 크기(정규 파일의 경우) */
      time_t      st_atime;      /* 마지막으로 접근된 시간 */
      time_t      st_mtime;      /* 마지막으로 수정된 시간 */
      time_t      st_ctime;      /* 파일 상태가 마지막으로 변경된 시간 */
      bliksize_t  st_blksize;    /* 최적의 I/O 블록 크기 */
      blkcnt_t    st_blocks;     /* 할당된 디스크 블록 개수 */
    };

## 4.3 파일의 종류

UNIX 시스템의 파일들은 정규 파일(regular file), 디렉토리로 나뉜다.

정규 파일: 가장 흔한 파일이며, 파일에 담기는 자료의 형태가 특별하게 정해져 있지는 않다. </br>
UNIX 커널은 그 자료가 텍스트인지 이진 형식인지를 구분하지 않는다. 단, 이진 실행 가능 파일(executable file)의 경우는 예외인데, </br>
프로그램을 실행하려면 커널은 프로그램이 담긴 실행 파일의 형식을 알아야 한다.

디렉토리 파일: 다른 파일들의 이름과 그 파일들에 대한 정보를 가리키는 포인터들을 담은 파일이다. </br>
디렉토리 파일에 대한 읽기 접근 권한을 가진 프로세스는 디렉토리의 내용을 읽을 수 있다. 그러나 디렉토리 파일을 직접 수정하는 것은 커널만이 가능하다.

블록 특수 파일: 디스크 드라이브 같은 장치에 대한 고정 크기 단위의 버퍼링 I/O 접근을 제공하는 파일이다. </br>
문자 특수 파일: 장치에 대한 가변 크기 단위의 비버퍼링 I/O 접근을 제공하는 파일이다. 시스템의 모든 장치는 블록 특수 파일 아니면 문자 특수 파일이다. </br>
FIFO: 프로세스 간 통신에 쓰이는 파일. 명명된 파이프(named pipe)라고도 불린다. </br>
소켓: 프로세스 간 네트워크 통신에 쓰이는 파일이다. 한 호스트 안의 프로세스들 사이의 비네트워크 통신에 사용할 수도 있다. </br>
기호 링크: 다른 파일을 가리키는 파일이다.

파일의 종류는 stat 구조체의 st_mode 멤버에 부호화된다.
    S_ISREG(): 정규 파일
    S_ISDIR(): 디렉토리 파일
    S_ISCHR(): 문자 특수 파일
    S_ISBLK(): 블록 특수 파일
    S_ISFIFO(): 파이프 또는 FIFO
    S_ISLNK(): 기호 링크
    S_ISSOCK(): 소켓

POSIX는 구현들이 메시지 대기열이나 semaphore 같은 프로세스 간 통신(IPC) 객체들을 파일로 나타내는 것도 허용한다. </br>
    S_TYPEISMQ(): 메시지 대기열
    S_TYPEISSEM(): 세마포어
    S_TYPEISSHM(): 공유 메모리 객체

## 4.4 SUID와 SGID

실제 사용자 ID와 실제 그룹 ID는 해당 사용자나 그룹이 실제로 누구인지를 식별한다. 이 두 필드들은 사용자가 로그인할 때 패스워드 파일의 해당 항목에서 추출된다. </br>
유효 사용자 ID와 유효 그룹 ID, 추가 그룹 ID들은 프로세스의 파일 접근 권한들을 결정한다. </br>
저장된 SUID와 저장된 SGID는 프로그램 실행 시의 유효 사용자 ID와 유효 그룹 ID의 복사본이다. </br>
보통의 경우 유효 사용자 ID는 실제 사용자 ID와 동일하고, 유효 그룹 ID는 실제 그룹 ID와 동일하다. 모든 파일에는 하나의 소유자와 그룹 소유자가 존재한다. </br>
파일의 소유자는 stat 구조체의 st_uid 멤버에 담겨 있으며 그룹 소유자는 st_gid 멤버에 해당한다.

프로그램 실행 파일을 실행할 때 프로세스의 유효 사용자 ID는 실제 사용자 ID와 같고 유효 그룹 ID는 실제 그룹 ID와 같은 것이 보통이다. </br>
그러나 파일의 모드 워드(st_mode)의 특별한 비트를 설정함으로써 "이 파일이 실행될 때, 프로세스의 유효 사용자 ID를 파일의 소유자(st_uid)로 설정하라"고 지정하는 것이 가능하다. 이것이 바로 SUID 비트다. </br>
프로세스 유효 그룹 ID를 파일의 그룹 소유자(st_gid)로 설정하라고 하는 것은 SGID 비트이다.

## 4.5 파일 접근 권한

디렉토리나 문자 특수 파일 등 모든 종류의 파일은 접근 권한들을 가진다. 각 파일마다 9개의 접근 허용 비트들이 존재한다. 이들은 크게 세 범주로 나뉜다.

| st_mode 마스크 | 의미 |
| :--: | :--: |
| S_IRUSR | 사용자 읽기(user-read) |
| S_IWUSR | 사용자 쓰기(user-write) |
| S_IXUSR | 사용자 실행(user-execute) |
| S_IRGRP | 그룹 읽기(group-read) |
| S_IWGRP | 그룹 쓰기(group-write) |
| S_IXGRP | 그룹 실행(group-execute) |
| S_IROTH | 기타 읽기(other-read) |
| S_IWOTH | 기타 쓰기(other-write) |
| S_IXOTH | 기타 실행(other-execute) |

사용자는 파일의 소유자를 의미하는데, 이 아홉 비트들을 수정할 때는 chmod 명령을 사용한다. </br>
이 명령은 인수에 포함된 u를 사용자, g를 그룹, o를 기타로 인식한다.

## 4.6 새 파일과 디렉토리의 소유권

새 디렉토리의 소유권에 대한 규칙들은 새 파일의 소유권에 대한 규칙들과 동일하다. </br>
새 파일의 사용자 ID는 프로세스의 유효 사용자 ID로 설정된다. 새 파일의 그룹 ID가 결정되는 방식은 여러가지인데 POSIX는 다음과 같은 두 가지 규칙 중 하나를 선택하도록 허용한다.

1. 프로세스의 유효 사용자 그룹 ID를 새 파일의 그룹 ID로 사용한다.
2. 새 파일이 생성된 디렉토리의 그룹 ID를 새 파일의 그룹 ID로 사용한다.

두번째 옵션(디렉토리의 그룹 ID 상속)이 쓰인다면, 한 디렉토리 안에 생성된 모든 파일과 디렉토리는 항상 그 디렉토리에 속하는 그룹 ID를 가지게 되며, 그러한 소유권은 그 디렉토리 아래의 계통 구조로도 전파된다.

## 4.7 access 함수

파일을 열 때 커널은 유효 사용자 ID와 유효 그룹 ID에 기초해서 접근 허용 여부를 판정한다. </br>
그런데 프로세스가 실제 사용자 ID와 실제 그룹 ID에 기초해서 접근 가능 여부를 판정하고 싶을 때도 있다. </br>
이는 SUID나 SGID 기능을 이용해서 프로세스를 다른 누군가의 계정으로 실행하는 경우에 유용하다. </br>
프로세스가 SUID를 통해서 루트 계정으로 실행된다고 해도, 어떤 파일에 실제 사용자가 접근할 수 있는지에 대한 판정이 필요할 수도 있는데 이 때 유용한 것이 access 함수이다.

    #include <unistd.h>

    int access(const char *pathname, int mode);

| mode | 설명 |
| :--: | :--: |
| R_OK | 읽기 권한 판정 |
| W_OK | 쓰기 권한 판정 |
| X_OK | 실행 권한 판정 |
| F_OK | 파일 존재 여부 판정 |

## 4.8 umask 함수

어떤 프로세스이든 가지는 파일 모드 생성 마스크 </br>
umask 함수는 프로세스의 모든 파일 모드 생성 마스크를 설정하고 마스크의 이전 값을 돌려준다.
    #include <sys/stat.h>

    mode_t umask(mode_t cmask);

cmask 인수에는 S_IRUSR, S_IWUSR 등 아홉 상수들 중 하나 또는 여러 개를 비트 단위 OR로 결합한 값을 지정해야 한다. </br>
파일 모드 생성 마스크는 프로세스가 새 파일이나 디렉토리를 생성할 때마다 쓰인다.

UNIX 시스템에서 보통의 사용자가 umask 값을 직접 변경하는 경우는 드물다. 보통의 경우는 로그인시 셸의 시동 스크립트에서 설정하며, 그 이후에는 변경되지 않는다. </br>
그렇지만 새 파일을 생성하는 프로그램을 작성할 때 특정한 접근 권한 비트들이 활성화됨을 보장하고 싶은 경우라면 프로세스 실행 도중 umask 값을 수정할 필요가 있다.

사용자는 umask의 값을 적절히 설정함으로써 자신이 생성하는 파일들의 기본 접근 권한들을 제어할 수 있다. 
| 마스크 비트 | 의미 |
| :--: | :--: |
| 0400 | 사용자 읽기(user-read) |
| 0200 | 사용자 쓰기(user-write) |
| 0100 | 사용자 실행(user-execute) |
| 0040 | 그룹 읽기(group-read) |
| 0020 | 그룹 쓰기(group-write) |
| 0010 | 그룹 실행(group-execute) |
| 0004 | 기타 읽기(other-read) |
| 0002 | 기타 쓰기(other-write) |
| 0001 | 기타 실행(other-execute) |

단일 UNIX 규격은 셸의 umask 명령이 기호 형태의 권한 값들도 지원해야 한다고 요구한다. </br>
어떤 권한들을 거부할 것인지를 지정하는 8진수 형태와 달리, 기호 형태는 어떤 권한들을 허용할 것인지를 지정한다.

## 4.9 chmod 함수와 fchmod 함수

이 두 함수는 기존 파일의 접근 허용 비트들을 변경하는데 쓰인다.
    #include <sys/stat.h>

    int chmod(const char *pathname, mode_t mode);

    int fchmod(int filedes, mode_t mode);

chmod 함수는 지정된 파일에 작동하는 반면, fchmod 함수는 이미 열린 파일에 작동한다. </br>
프로세스가 파일의 접근 허용 비트들을 변경할 수 있으려면 프로세스의 유효 사용자 ID가 파일의 소유자 ID와 같거나 프로세스가 슈퍼사용자 특권을 가지고 있어야 한다. </br>
mode 인수에는 아래 표의 상수들을 비트단위 OR로 결합한 값을 넣는다.
| mode | 설명 |
| :--: | :--: |
| S_ISUID | 실행에 대한 SUID |
| S_ISGID | 실행에 대한 SGID |
| S_ISVTX | 저장된 텍스트(saved-text, 끈적이 비트) |
| S_IRWXU | 사용자(소유자)의 읽기, 쓰기, 실행 |
| S_IRUSR | 사용자(소유자)의 읽기 |
| S_IWUSR | 사용자(소유자)의 쓰기 |
| S_IXUSR | 사용자(소유자)의 실행 |
| S_IRWXG | 그룹의 읽기, 쓰기, 실행 |
| S_IRGRP | 그룹의 읽기 |
| S_IWGRP | 그룹의 쓰기 |
| S_IXGRP | 그룹의 실행 |
| S_IRWXO | 기타(세계)의 읽기, 쓰기, 실행 |
| S_IROTH | 기타(세계)의 읽기 |
| S_IWOTH | 기타(세계)의 쓰기 |
| S_IXOTH | 기타(세계)의 실행 |

chmod와 fchmod 함수는 다음과 같은 조건들 하에서 접근 허용 비트 두개를 자동으로 해제한다. </br>
1. 정규 파일에 대한 끈적이 비트에 특별한 의미를 두는 시스템들에서 슈퍼 사용자 특권을 가지지 않은 프로세스가 정규 파일에 대해 끈적이 비트(S_ISVTX)를 설정하려고 하면 mode의 끈적이 비트가 자동으로 꺼진다. </br>
결과적으로 슈퍼 사용자만이 정규 파일에 대해 끈적이 비트를 설정할 수 있다. 이러한 처리는 불산한 의도를 가진 사용자가 시스템 성능에 해를 끼치려는 목적으로 끈적이 비트를 설정하려는 시도를 방지하기 위한 것이다.

2. 새로 생성된 파일의 그룹 ID가 호출한 프로세스가 속하지 않은 그룹의 ID로 설정될 수도 있다. </br>
새 파일의 그룹 ID가 프로세스의 유효 그룹 ID 또는 프로세스의 추가 그룹 ID들 중 하나와 같지 않으며 프로세스가 슈퍼 사용자 특권들을 가지지 않는 경우 SGID 비트가 자동으로 꺼진다. </br>
그래서 사용자는 자신이 속하지 않은 그룹이 소유하는 SGID 파일을 생성할 수 없게 된다.

## 4.10 끈적이 비트

S_ISVTX 비트를 끈적이 비트라고 불렀다. 실행가능한 프로그램 파일에 이 비트가 설정되어 있는 경우, 그 프로그램이 처음으로 실행되었다가 종료될 때 시스템은 프로그램 텍스트의 복사본을 교체 영역(swap area)에 저장했다. </br>
이후 프로그램이 실행되면 시스템은 디스크 파일이 아니라 교체 영역에 있는 프로그램 텍스트를 메모리에 적재한다. 보통의 UNIX 파일 시스템에서 파일의 자료 블록들은 여기저기 흩어져 있을 수 있는 반면, 교체 영역은 하나의 연속적인 파일로 처리되므로, 결과적으로 프로그램이 좀 더 빠르게 메모리에 적재된다. 

이 끈적이 비트는 텍스트 편집기라던가 c 컴파일러의 빌드 공정을 구성하는 여러 프로그램들 등 자주 쓰이는 응용프로그램들에 대해 설정되는 경우가 많았다. UNIX 시스템 이후 버전들에서는 저장된 비트라고 칭했는데, 이로부터 S_ISVTX라는 상수 이름이 비롯되었다. 더 최근의 UNIX 시스템은 대부분 가상 메모리 시스템과 좀 더 빠른 파일 시스템을 가지고 있기 때문에 이러한 끈적이 기법을 사용할 필요가 없어졌다.

현대적인 시스템에서는 끈적이 비트의 용도가 확장되었다. 단일 UNIX 규격은 디렉토리에 끈적이 비트를 설정하는 것도 허용한다. 디렉토리에 이 비트가 설정되어 있으면, 그 디렉토리의 한 파일을 삭제하거나 이름을 변경하기 위해서는 사용자가 그 디렉토리에 대해 쓰기 권한을 가지고 있어야 할 뿐만 아니라, 그 파일의 소유자이거나, 그 디렉토리의 소유자이거나, 슈퍼 사용자이어야 한다. 

끈적이 비트가 설정되는 전형적인 디렉토리는 /tmp와 /var/spool/uucppublic을 들 수 있다. 이들은 보통의 경우 어떤 사용자라도 파일을 생성할 수 있는 디렉토리들이다.

## 4.11 chown, fchown, lchown 함수

chown류 함수들은 파일의 사용자 ID와 그룹 ID를 변경하는데 쓰인다.
        int chown(const char *pathname, uid_t owner, gid_t group);
        int fchown(int filedes, uid_t owner, gid_t group);
        int lchown(const char *pathname, uid_t owner, gid_t group);

이 세 함수는 서로 비슷하게 기능하는 반면 주어진 파일이 기호 링크일 때는 차이를 보인다. 그런 경우 lchown 함수는 기호 링크 자체의 소유자를 변경한다. owner 인수나 group 인수가 -1이면 해당 ID는 변하지 않는다.

지정된 _POSIX_CHOWN_RESTRICTED가 작용한다면,

1. 슈퍼 사용자 프로세스만이 그 파일의 사용자 ID를 변경할 수 있다.
2. 슈퍼 사용자가 아닌 프로세스의 경우, 프로세스가 그 파일을 소유하며, owner 인수로 지정된 값이 -1 또는 파일의 사용자 ID이며, group 인수로 지정된 값이 프로세스의 유효 그룹 ID이거나 프로세스의 추가 그룹 ID들 중 하나이면 프로세스는 파일의 그룹 ID를 변경할 수 있다.

결과적으로, _POSIX_CHOWN_RESTRICTED가 작용하는 경우 사용자는 다른 사용자가 소유한 파일의 사용자 ID를 변경할 수 없다.

## 4.12 파일 크기

stat 구조체의 st_size 멤버는 파일의 크기를 담는다. 이 필드는 정규 파일, 디렉토리, 기호 링크에 대해서만 의미를 가진다. </br>
정규 파일의 경우 파일 크기가 0인 것도 가능하다. </br>
디렉토리의 경우 파일 크기는 어떤 수의 배수인 것이 보통이다. </br>
기호 링크의 경우 파일의 크기는 파일 이름의 바이트 수이다. </br>
요즘의 UNIX 시스템들은 대부분 st_blksize와 st_blocks라는 필드들을 제공한다. 전자는 파일에 대한 I/O에서 선호되는 블록 크기이고, 후자는 실제로 할당된 블록들의 개수이다.

현재 파일 끝을 지나친 곳으로 파일 오프셋 위치를 옮긴 후에 자료를 기록하면 구멍이 생긴다. 

## 4.13 파일 절단

파일의 끝 부분에 있는 일부 자료를 제거해서 파일을 절단하고 싶을 때가 있다. open에 O_TRUNC를 지정해서 할 수 있는 파일 비우기는 파일 절단의 한 특별한 경우에 해당한다.
        #include <unistd.h>

        int truncate(const char *pathname, off_t length);

        int ftruncate(int filedes, off_t length);

이 두 함수는 기존의 파일을 길이가 length 바이트가 되도록 절단한다. 파일의 원래 크기가 length보다 컸다면 length 이후의 자료에는 더 이상 접근할 수 없게 된다.

## 4.14 파일 시스템

## 4.15 link, unlink, remove, rename 함수

한 파일의 i노드를 여러 개의 디렉토리 항목들이 가리킬 수 있다. 기존 파일에 대한 링크를 생성할 때에는 link 함수를 사용한다.
        #include <unistd.h>

        int link(const char *existingpath, const char *newpath);
이 함수는 기존 파일 existingpath를 가리키는 새 디렉토리 항목 newpath를 생성한다.

기존 디렉토리 항목을 제거할 때는 unlink 함수를 사용한다.
        #include <unistd.h>

        int unlink(const char *pathname);
이 함수는 pathame이 가리키는 파일의 디렉토리 항목을 제거하고 해당 링크 횟수를 감소한다. </br>
파일을 언링크하기 위해서는 해당 디렉토리 항목을 담은 디렉토리에 대한 쓰기 권한과 실행 권한이 필요하다. </br>
디렉토리에 끈적이 비트가 설정되어 있다면 프로세스가 디렉토리에 대한 쓰기 권한을 가지고 있어야 할 뿐만 아니라 그 파일의 소유자이거나, 그 디렉토리의 소유자이거나, 슈퍼 사용자 특권들을 가지고 있어야 한다. </br>
파일의 내용은 링크 횟수가 0에 도달했을 때에만 삭제될 수 있다. </br>

프로그램 실행 도중 임시 파일을 생성해서 사용하는 경우가 있는데, 프로그램이 폭주하는 경우에도 임시 파일이 자동으로 삭제되게 만들고자 할 때 unlink의 이러한 특징이 유용하게 쓰인다. </br>
pathname이 기호 링크이면 unlink는 기호 링크를 제거한다. 기호 링크 이름이 주어졌을 때 기호 링크가 가리키는 파일을 제거하는 함수는 없다. </br>
슈퍼 사용자는 디렉토리 이름을 pathname에 지정해서 unlink를 호출할 수 있으나 디렉토리를 제거할 때에는 디렉토리를 언링크하는 대신 rmdir 함수를 사용해야 한다. 하지만 remove 함수로도 파일이나 디렉토리를 언링크할 수 있다. 파일의 경우 remove는 unlink와 동일하다. 디렉토리의 경우 remove는 rmdir와 동일하다. 
        #include <stdio.h>

        int remove(const char *pathname);

파일이나 디렉토리의 이름을 변경할 때에는 rename 함수를 사용한다.
        #include <stdio.h>
        int rename(const char *oldname, const char *newname);
이 함수의 작동 방식은 oldname이 파일을 지칭하는지, 디렉토리를 지칭하는지, 기호 링크를 지칭하는지에 따라 달라진다. 또한 newname이 이미 존재하는지의 여부도 함수의 작동에 영향을 미친다. </br>
newname이 이미 존재하다면 그것을 삭제하는데 필요한 권한들이 있어야 한다.

## 4.16 기호 링크

기호 링크는 파일에 대한 간접 포인터이다. 기호 링크는 하드 링크의 다음과 같은 한계들을 우회하기 위해 도입되었다. </br>
1. 보통의 경우, 하드 링크에서는 링크와 해당 파일이 같은 파일 시스템에 존재해야 한다.
2. 디렉토리에 대한 하드 링크는 슈퍼 사용자만 만들 수 있다.

기호 링크는 한 파일 또는 디렉토리 계통 구조 전체를 시스템의 다른 장소로 옮기려 할 때 주로 쓰인다. </br>
파일을 이름으로 지칭하는 함수를 사용할 때에는 그 함수가 기호 링크를 따라가는지의 여부를 확인할 필요가 있다.

기호 링크 때문에 파일 시스템에 루프가 생길 수 있다. 경로이름을 받은 함수를 대부분은 그런 일이 생겼을 때 오류를 반환하고 errno를 ELOOP로 설정한다. 
        $ mkdir foo
        $ touch foo/a
        $ ln -s ../foo foo/testdir
        $ ls -l foo
        
        total 0
        -rw-r----- l sar    0 Jan 22 00:16 a
        lrwxrwxrwx l sar    6 Jan 22 00:16 testdir -> ../foo

        foo
        foo/a
        foo/testdir
        foo/testdir/a
        foo/testdir/testdir
        foo/testdir/testdir/a
        foo/testdir/testdir/testdir
        foo/testdir/testdir/testdir/a
        ...
foo라는 디렉토리를 만들고, 그 안에 a라는 파일과 foo를 가리키는 기호 링크를 만든 것이다. ELOOP 오류가 발생할 때까지 위와 같이 계속 출력된다.

기호 링크에 해당하는 경로이름을 지정해서 open을 호출하면, open은 기호 링크를 따라간다. 그런데 기호 링크가 가리키는 파일이 존재하지 않으면 open은 파일을 열 수 없음을 뜻하는 오류를 돌려준다. 
        $ ln -s /no/such/file myfile
        $ ls myfile
        myfile
        $ cat myfile
        cat: myfile: No such file or directory
        $ ls -l myfile
        lrwxrwxrwx l sar    13 Jan 22 00:26 myfile -> /no/such/file
ls 명령은 myfile이 존재한다고 하지만, cat은 그런 파일이 없다고 한다. 이는 myfile이 하나의 기호 링크이며 그 기호 링크가 가리키는 파일이 존재하지 않기 때문이다. </br>
ls의 -l 옵션을 적용하면 두가지 힌트를 얻을 수 있다. 하나는 결과의 첫 글자가 l이라는 것인데, 이는 해당 파일이 기호 링크라는 뜻이다. 그리고 -> 또한 이것이 기호 링크임을 알려준다.

## 4.17 symlink 함수와 readlink 함수

기호 링크를 생성할 때에는 symlink 함수를 사용한다.
        #include <unistd.h>

        int symlink(const char *actualpath, const char *sympath);
이 함수의 호출이 성공하면 actualpath를 가리키는 sympath라는 새 디렉토리 항목(기호 링크)이 생성된다. 기호 링크 생성 시 actualpath가 반드시 존재해야 하는 것은 아니다. </br>
actualpath와 sympath가 반드시 같은 파일 시스템에 존재해야 할 필요도 없다.

open 함수는 기호 링크를 따라가므로, 링크 자체를 열어서 그 링크에 담긴 이름을 읽으려면 다른 수단이 필요한데 그것이 readlink 함수이다.
        #include <unistd.h>

        ssize_t readlink(const char* restrict pathname, char *restrict buf, size_t bufsize);
이 함수는 open, read, close의 기능을 조합한 것이다. 함수 호출 성공 시 함수는 buf로 읽어들인 바이트들의 개수를 돌려준다. 

## 4.18 파일 시간들

| 필드 | 설명 | 예 | ls(1) 옵션 |
| :--: | :--: | :--: | :--: |
| st_atime | 파일 자료의 최종 접근 시간 | read | -u |
| st_mtime | 파일 자료의 최종 수정 시간 | write | 기본 |
| st_ctime | i노드 상태의 최종 수정 시간 | chmod, chown | -c |
파일 수정 시간(st_mtime)과 상태 변경 시간(st_ctime)의 차이를 주의해야 한다. 전자는 파일의 내용이 마지막으로 수정된 시간이며, 후자는 파일의 i노드가 마지막으로 수정된 시간이다. </br>
i노드의 모든 정보는 파일의 실제 내용과는 개별적으로 저장되기 때문에, 파일 수정 시간뿐만 아니라 상태 변경 시간도 따로 유지할 필요가 있다. 하지만 시스템이 i노드에 대한 최종 접근 시간을 갱신하지는 않는다. </br>
접근 시간은 시스템 관리자가 일정 기간동안 접근되지 않은 파일들을 삭제하고자 할 때 유용하게 쓰인다. </br>
파일 수정 시간과 상태 변경 시간은 내용이 변경된 파일들 또는 i노드가 수정된 파일들만 따로 보관하려 할 때 유용하다. </br>
ls 명령은 세 시간 값들 중 하나만을 표시하며, 셋 중 하나만을 정렬의 기준으로 사용한다. 기본적으로는 -l이나 -t 옵션이 주어진 경우 파일의 수정 시간을 사용한다. </br> 
그리고 -u 옵션이 주어지면 접근 시간을, -c 옵션이 주어지면 상태 변경 시간을 사용한다. </br>
파일 읽기 또는 파일 쓰기는 파일과 파일의 i노드에만 영향을 줄 뿐, 그 파일이 있는 디렉토리에는 영향을 주지 않는다.

## 4.19 utime 함수