# chapter 4 파일과 디렉터리

## 4.2 stat, fstat, fstatat, lstat 함수
int fstat(int fd, struct stat *buf); </br>
int lstat(const char *restrict pathname, struct stat *restrict buf); </br>
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag); </br>

stat 함수는 pathname 인수로 지정된 파일에 대한 정보를 한 구조체에 담아 돌려준다. </br>
fstat 함수는 fd 인수로 지정된 파일 서술자에 대해 이미 열려 있는 파일에 관한 정보를 돌려준다. </br>
lstat 함수는 stat과 비슷하되 기호 링크에 대한 정보를 돌려준다. </br>
기호 링크가 가리키는 파일이 아니라 기호 링크 자체에 대한 정보임을 주의해야 한다. 

fstat 함수는 fd 인수로 지정된 파일 서술자에 해당하는 열린 디렉터리를 기준으로 한 상대 경로이름에 관한 파일 통계치를 돌려준다. flag 인수는 기호 링크를 따라갈 것인지의 여부를 결정한다. </br>
이 인수로 AT_SYMLINK_NOFOLLOW 플래그를 설정하면 fstatat은 기호 링크를 따라가지 않고 대신 링크 자체에 대한 정보를 돌려준다. </br>

struct stat { </br>
mode_t           st_mode;   /* 파일 종류 및 모드(권한) */ </br>
ino_t            st_ino;    /* i-노드 번호(일련 번호) */ </br>
dev_t            st_dev;    /* 장치 번호(파일 시스템) */ </br>
dev_t            st_rdev;   /* 특수 파일들을 위한 장치 번호 */ </br>
nlink_t          st_nlink;  /* 링크 개수 */ </br>
uid_t            st_uid;    /* 파일 소유자의 사용자 ID */ </br>
gid_t            st_gid;    /* 파일 소유자의 그룹 ID */ </br>
off_t            st_size;   /* 바이트 단위 크기(정규 파일의 경우) */ </br>
struct timespec  st_atim;  /* 마지막 접근 시간 */ </br>
struct timespec  st_mtim;  /* 마지막 수정 시간 */ </br>
struct timespec  st_ctim;  /* 마지막 파일 상태 변경 시간 */ </br>
blksize_t        st_blksize;    /* 최선의 입출력 블록 크기 */ </br>
blkcnt_t         st_blocks    /* 할당된 디스크 블록 개수 */ </br>
}; </br>

timespec 구조체 형식은 초 및 나노초 단위의 시간을 정의한다. </br>
time_t  tv_sec; </br>
long    tv_nsec; </br>

## 4.3 파일 종류
한 UNIX 시스템의 파일들은 대부분 정규 파일 아니면 디렉터리이나, 그 둘에 해당하지 않는 파일들도 있다. </br>
1. 정규 파일(regular file). 가장 흔한 파일로, 어떤 형태이든 자료를 담는다. UNIX 커널은 텍스트 자료와 이진 자료를 구분하지 않는다. 정규 파일의 내용을 어떻게 해석하는지는 그 파일을 처리하는 응용 프로그램의 몫이다. </br>
(프로그램을 실행하려면 반드시 실행 파일의 형식을 알아야 한다. 모든 이진 실행 파일은 커널이 프로그램의 텍스트와 자료를 어디에 적재해야 하는지 알 수 있게 하는 특정한 형식을 따른다. </br>
2. 디렉터리 파일(directory file). 다른 파일들의 이름과 그 파일들에 대한 정보로의 포인터들을 담은 파일이다. 디렉터리 파일에 대한 읽기 권한을 거잔 모든 프로세스는 그 디렉터리의 내용물도 읽을 수 있으나, 디렉터리 파일에 직접 쓸 수 있는 것은 커널뿐이다. </br>
프로세스가 디렉터리를 변경하기 위해서는 파일과 디렉터리 함수들을 사용해야 한다. </br>
3. 블록 특수 파일(block special file). 디스크 드라이브 같은 장치에 대해 고정 크기 단위의 버퍼링 있는 입출력 접근을 제공하는 파일이다. </br>
4. 문자 특수 파일(character special file). 장치에 대한 가변 크기 단위의 버퍼링 없는 입출력 접근을 제공하는 파일이다. 한 시스템의 모든 장치는 블록 특수 파일 아니면 문자 특수 파일이다. </br>
5. FIFO. 프로세스들 사이의 통신에 쓰이는 파일이다. 명명된 파이프라고 부르기도 한다. </br>
6. 소켓. 프로세스들 사이의 비 네트워크 통신에 쓰이는 파일이다. 하나의 호스트 안에 있는 프로세스들 사이의 비 네트워크 통신에 소켓을 사용할 수도 있다. </br>
7. 기호 링크. 다른 파일을 가리키는 파일이다. </br>
파일의 종류는 stat 구조체의 st_mode 멤버에 부호화된다.

POSIX.1은 구현이 메시지 대기열이나 세마포 같은 프로세스 간 통신(IPC) 객체들을 파일로서 나타내는 것을 허용한다. 

<sys/stat.h>의 파일 종류 식별 매크로들 </br>
S_ISREG()    정규 파일 </br>
S_ISDIR()    디렉터리 파일 </br>
S_ISCHR()    문자 특수 파일 </br>
S_ISBLK()    블록 특수 파일 </br>
S_ISFIFO()   파이프 또는 FIFO </br>
S_ISLNK()    기호 링크 </br>
S_ISSOCK()   소켓 </br>

stat 구조체를 이용해서 IPC 객체의 종류를 파악하는 데 쓰이는 매크로들 </br>
S_TYPEISMQ()  메시지 대기열
S_TYPEISSEM()  세마포
S_TYPEISSHM()  공유 메모리 객체
