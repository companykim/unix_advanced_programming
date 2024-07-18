# chapter 6 시스템 자료 파일과 시스템

## 6.2 패스워드 파일

UNIX 시스템들은 패스워드 파일을 /etc/passwd에 ASCII 형태로 저장해 왔다. </br>
보통의 경우 패스워드 파일에는 사용자 이름이 root인 항목이 존재한다. </br>
암호화된 패스워드 필드에 문자 하나만 들어있다. 암호화된 패스워드를 누구나 읽을 수 있으면 보안상 문제가 생길 수 있기 때문에 요즘에는 암호화된 패스워드를 다른 파일에 저장하고 패스워드 파일의 해당 부분에는 그냥 문자 하나만 기록해둔다. </br>
패스워드 파일 항목의 일부 필드들은 비어 있을 수 있다. 암호화된 패스워드가 비어 있다면, 일반적으로 그것은 해당 사용자 계정에 패스워드가 없다는 것이다. </br>
셸 필드에는 사용자의 로그인 셸로 쓰이는 실행 가능 프로그램의 이름이 담긴다. 셸 필드가 비어 있는 경우, 기본적으로 쓰이는 프로그램은 /bin/sh이다. 그런데 squid 줄의 셸 필드에는 /dev/null이 지정되어 있다. </br>
/dev/null 이외에 특정 사용자의 로그인 방지를 위해 쓰이는 기법은 로그인 셸로 /bin/false를 지정하는 방법도 있는데, 이는 성공적이지 못한 종료 상태를 반환하는 것이 유일한 용도인 프로그램이다. </br>
비슷하게 항상 성공에 해당하는 종료 상태를 돌려주는 /bin/true도 그런 목적으로 쓰인다. </br>
nobody라는 사용자 이름은 누구나 시스템에 로그인할 수 있게 하는 데 쓰인다. 단, 이 이름으로 로그인 한 사용자에게는 어떠한 특권도 없는 사용자 ID(65534)와 그룹 ID(65534)가 배정된다. 이 사용자 ID와 그룹 ID로는 기타 계정이 읽거나 쓸 수 있는 파일들만 읽거나 쓸 수 있다. </br>
finger(1) 명령을 지원하는 몇몇 시스템들은 주석 필드 안에 추가적인 정보를 설정할 수 있게 한다. 시스템이 finger 명령을 지원하지 않는다고 해도 주석 필드에 정보를 집어 넣는 일은 얼마든지 가능하다.

관리자가 패스워드 파일을 안전하게 수정할 수 있도록 vipw 명령을 제공하는 시스템들도 있다. </br>
vipw 명령은 패스워드 파일에 대한 변경들을 직렬화하며, 파일의 변경 내용이 시스템의 다른 자료 파일들에 적절히 반영되도록 만든다. 또한 비슷한 기능성을 그래픽 사용자 인터페이스를 통해서 제공하는 시스템들도 많이 있다.

POSIX.1에서 패스워드 파일의 항목들을 조회하는데 쓰이는 함수는 단 두가지이다. 이 함수들은 사용자의 로그인 이름이나 수치 사용자 ID를 돌려준다.
    #include <pwd.h>

    struct passwd *getpwuid(uid_t uid);
    struct passwd *getpwnam(const char *name);
getpwuid 함수는 예를 들어 ls 프로그램이 파일 i노드에 담긴 수치적 사용자 ID로부터 그에 해당하는 사용자 로그인 이름을 얻을 때 쓰인다. </br>
getpwnam 함수는 예를 들어 login 프로그램에서 사용자가 로그인 이름을 입력할 때 쓰인다. </br>
두 함수가 모두 정보가 기록된 passwd 구조체를 가리키는 포인터를 돌려준다. 보통의 경우 이 구조체는 함수 내부의 static 변수이므로, 함수를 다시 호출하면 그 내용이 갱신된다. </br>
이 두 POSIX.1 함수들은 특정한 로그인 이름이나 사용자 ID 중 하나를 알고 있는 상황에서 그에 해당하는 사용자 ID나 로그인 이름을 알아내고자 할 때 유용하다. </br>
그러한 사전 지식 없이 패스워드 파일에 담긴 정보들을 차례로 훑고 싶을 때는 다음 세 함수들을 사용해야 한다. 
    #include <pwd.h>

    struct passwd *getpwent(void);
    void setpwent(void);
    void endpwent(void);
getpwent 함수는 패스워드 파일의 다음 항목들을 돌려준다. 자신이 정보를 채운 구조체를 가리키는 포인터를 돌려준다. 그 구조체는 함수가 호출될 때마다 갱신된다. </br>
첫 호출시 getpwent는 자신의 작업에 필요한 파일을 연다. 그런데 이 함수가 돌려주는 항목들의 순서에 대해서는 어떠한 보장도 없다. 이는 /etc/passwd 파일의 내용을 hashing해서 사용하는 시스템들이 있기 때문이다. </br>
setpwent 함수는 해당 파일을 되감으며, endpwent 함수는 그 파일들을 닫는다. getpwent 함수를 다 사용했으면 반드시 endpwent로 파일들을 닫아야 한다. 

(ex1) 함수 시작 부분에서는 setpwent를 호출해서 파일들을 되감는다. 이는 호출자가 getpwent를 호출해서 이미 파일들을 열어둔 경우를 대비한 것이다. </br>
마찬가지로 함수 끝 부분에서는 endpwent를 호출해서 파일들을 닫는다. getpwnam이나 getpwuid 모두 파일들을 열어둔 채로 남겨두어서는 안되기 때문이다. 

## 6.3 그림자 패스워드

암호화된 패스워드는 사용자의 패스워드를 단방향 암호화 알고리즘으로 암호화한 것이다. 알고리즘이 단방향이므로, 암호화된 패스워드로부터 원래의 패스워드를 추측할 수는 없다.
Linux 2.4.22와 Solaris 9는 그림자 패스워드 파일에 접근하기 위한 개별적인 함수들을 제공한다. 이들은 패스워드 파일에 접근하는데 쓰이는 함수들과 비슷하다. 
    #include <shadow.h>

    struct spwd *getspnam(const char *name);
    struct spwd *getspent(void);
    void setspent(void);
    void endspent(void);

## 6.4 그룹 파일

gr_mem 필드는 해당 그룹에 속한 사용자 이름들을 가리키는 포인터들의 배열이다. 이 배열의 마지막 원소는 널 포인터이다. </br>
그룹 이름이나 수치 그룹 ID를 조회할 때에는 다음 두 함수를 사용한다.
    #include <grp.h>

    struct group *getgrgid(gid_t gid);
    struct group *getgrnam(const char *name);
패스워드 파일에 대한 함수들처럼, 일반적으로 이 두 함수는 하나으 static 구조체에 대한 포인터를 돌려주며, 그 구조체는 함수가 호출될 때마다 갱신된다. </br>
그룹 파일 전체를 훑을때 쓰는 함수들
        #include <grp.h>

        struct group *getgrent(void);
        void setgrent(void);
        void endgrent(void);
setgrent 함수는 그룹 파일을 열고 되감는다. getgrent 함수는 그룹 파일의 다음 항목을 읽는다. endgrent 함수는 그룹 파일을 닫는다. 

## 6.5 추가 그룹 ID

추가 그룹 ID에 의해, 사용자는 패스워드 파일 항목의 그룹 ID에 해당하는 그룹뿐만 아니라 최대 16개의 추가적인 그룹들에도 속할 수 있게 되었으며, 파일 접근 권한 점검 역시 그에 맞게 변경되었다. </br>
즉, 파일의 그룹 ID를 사용자의 유효 그룹 ID와 비교할 뿐만 아니라 사용자의 추가 그룹 ID들도 파일의 그룹 ID와 비교해서 최종적인 파일 접근 허용 여부를 결정하게 되었다.

추가 그룹 ID들을 사용할 때의 장점은 사용자가 명시적으로 그룹을 바꾸지 않아도 된다는데 있다. 한 사용자가 여러개의 그룹들에 동시에 속하는일이 드물지 않다는 점에서 이런 변화는 바람직하다.
        #include <unistd.h>

        int getgroups(int gidsetsize, gid_t grouplist[]);

        #include <grp.h>
        #include <unistd.h>

        int setgroups(int ngroups, const gid_t grouplist[]);

        #include <grp.h>
        #include <unistd.h>

        int initgroups(const char *username, gid_t basegid);
getgroups 함수는 grouplist로 주어진 배열에 추가 그룹 ID들을 채운다. 함수는 배열에 저장된 추가 그룹 ID들의 개수를 돌려준다. </br>
한가지 특별한 경우로, gidsetsize를 0으로 하면 함수는 추가 그룹 ID 개수만 돌려주고 grouplist 배열은 채우지 않는다.</br>
슈퍼사용자는 setgroups 함수를 이용해서 현재 프로세스의 추가 그룹 ID 목적을 설정할 수 있다. grouplist는 설정할 추가 그룹 ID들의 배열이고 ngroups는 배열의 원소 개수이다. ngroups의 값이 NGROUPS_MAX보다 커서는 안된다. </br>
보통의 경우 setgroups 함수는 initgroups 함수에서만 호출된다. 

UNIX 시스템들은 패스워드 파일과 그룹 파일 이외에도 수많은 자료 파일들을 일상적인 용도로 사용한다. </br>
모든 자료 파일마다 적어도 다음 세 종류의 함수들이 제공된다. </br>
1. 다음 레코드를 읽는 get 함수 - 대체로 이런 종류의 함수들은 정보를 담은 구조체를 가리키는 포인터를 돌려준다. 
2. 파일이 아직 열려 있지 않으면 먼저 파일을 열고 파일을 되감는 set 함수 - 파일의 시작에서부터 레코드들을 얻고자 할 때에는 이런 함수를 호출해야 함.
3. 자료 파일을 닫는 end 함수 - 파일을 다 사용한 후에는 반드시 이런 함수를 호출해서 파일들을 모두 닫아야 한다.

그밖에도 자료 파일이 어떤 형태이든 키 참조를 지원한다면 특정 키에 해당하는 레코드를 찾는 함수들도 제공된다.

## 6.8 로그인 계정 관리

대부분의 UNIX 시스템들은 utmp 파일에 현재 로그인해 있는 사용자를, wtmp 파일에 모든 로그인, 로그아웃을 기록했었다. </br>
요즘 UNIX 시스템 버전들도 대부분은 여전히 utmp 파일과 wtmp 파일을 지원하는데, 이 파일들에 저장되는 정보의 양이 늘어났다.

## 6.9 시스템 식별

POSIX.1은 현재 호스트 및 운영체제에 대한 정보를 돌려주는 uname 함수를 정의한다. 
        #include <sys/utsname.h>

        int uname(struct utsname *name);
name에는 utsname 구조체의 주소를 넣어야 한다. 그러면 함수가 그 구조체에 정보를 채워 넣는다. 
        struct ustname {
            char sysname[]; // 운영체제 이름
            char nodename[]; // 이 노드의 이름
            char release[]; // 운영체제의 현재 릴리즈
            char version[]; // 이 릴리즈의 현재 버전
            char machine[]; // 하드웨어 종류 이름
        };

BSD 기반 시스템들에서부터, 호스트 이름만을 돌려주는 gethostname이라는 함수가 제공되었다. </br>
보통의 경우 이 이름은 TCP/IP 네트워크 상의 호스트의 이름이다. 현재는 gethostname 함수가 POSIX.1의 일부이다. 
        #include <unistd.h>

        int gethostname(char *name, int namelen);

namelen 인수에는 name이 가리키는 버퍼의 크기를 넣어야 한다. 버퍼가 충분히 큰 경우 버퍼에는 호스트 이름에 해당하는 널 종료 문자열이 저장된다. </br>
gethostname 함수가 설정할 수 있는 호스트 이름의 최대 길이는 HOST_NAME_MAX라는 상수로 정의된다. </br>
호스트가 TCP/IP 네트워크에 연결되어 있다면, 대개의 경우 호스트 이름은 호스트의 완전히 안정된 형태의 도메인 이름이다. </br>
호스트 이름은 hostname이라는 명령으로도 조회하거나 설정할 수 있다. 보통 시스템의 호스트 이름은 시스템 부팅시 /etc/rc나 init가 실행하는 시동 파일들 중 하나가 설정한다. 

## 6.10 시간 및 날짜 함수들

UNIX 커널이 제공하는 기본적인 시간 서비스는 UNIX 기원인 1970년 1월 1일 0시 0분 0초로부터 흐른 초들의 개수를 시간 값으로 사용한다. 이를 달력 시간이라고 부른다. </br>
프로그램 안에서는 time_t라는 자료 형식으로 표현한다. 이 달력 시간은 시간과 날짜 모두를 표현한다. </br>
UNIX 시스템은 (a) 지역시간이 아니라 UTC를 기준으로 하며, (b) 일광절약시간 등의 변환을 자동으로 처리하며, (c) 시간과 날짜를 단일한 수량으로 표현한다는 점에서 다른 운영체제들과 차별화된다. </br>
time 함수는 현재 시간 및 날짜에 해당하는 시간 값을 돌려준다. 
        #include <time.h>

        time_t time(time_t *calptr);
이 함수는 현재 시간 및 날짜에 해당하는 시간 값을 돌려주며, calptr 인수가 널이 아니면 그것이 가리키는 곳에도 시간 값을 저장한다.

gettimeofday 함수는 time보다 더 높은 해상도의 시간을 제공한다. 이러한 고해상도 시간은 몇몇 응용 프로그램에서 중요하다.
        #include <sys/time.h>

        int gettimeofday(struct timeval *restrict tp, void *restrict tzp);
이 함수는 단일 UNIX 규격의 한 XSI 확장으로 정의되어 있다. tzp에는 반드시 NULL을 넣어야 한다. </br>
gettimeofday 함수는 UNIX 기원으로부터 측정된 현재 시간을 tp가 가리키는 메모리에 저장한다. tp는 timeval 구조체를 가리키는데, 거기에는 초와 마이크로초 값이 저장된다. 
        struct timeval {
            time_t tv_sec; // 초
            time_t tv_usec; // 마이크로초
        };
UNIX 기원 이후 흐른 초들의 개수에 해당하는 정수 값을 얻었다면, 다양한 시간 함수들을 이용해서 그것을 사람이 읽기 좋은 시간 및 날짜로 변환할 수 있다. 

localtime 함수와 gmtime 함수는 달력 시간을 소위 분할된 시간으로 변환한다. 분할된 시간에 해당하는 구조체는 tm이다.
        struct tm { // 분할된 시간
            int tm_sec; // 현재 분의 초: [0 - 60]
            int tm_min; // 현재 시의 분: [0 - 59]
            int tm_hour; // 자정 이후의 시: [0 - 23]
            int tm_mday; // 그 달의 날짜: [1 - 31]
            int tm_mon; // 달: [0 - 11]
            int tm_year; // 1900 이후의 연도
            int tm_wday; // 요일번호(일요일이 0): [0 - 6]
            int tm_yday; // 1월 1일 이후의 일수: [0 - 365]
            int tm_isdst; // 일광절약시간 플래그: <0, 0, >0
        };
초의 최대값이 59가 아니라 60인 것은 윤초 때문이다. 그 달의 날짜 필드를 제외한 모든 필드는 0에서 시작함을 주의할 것. </br> 
일광절약시간 플래그는 일광절약시간이 적용될 때에는 0보다 큰 값, 적용되지 않을 때에는 0, 적용 여부를 알 수 없을 때에는 0보다 작은 값이다.

        #include <time.h>

        struct tm *gmtime(const time_t *calptr);
        struct tm *localtime(const time_t *calptr);
localtime 함수와 gmtime 함수 모두 달력 시간을 분할된 시간으로 변환하지만, 전자는 달력 시간을 먼저 지역 시간대와 일광절약시간 플래그를 고려한 지역 시간 기준의 분할된 시간으로 변환하는 반면, </br>
후자는 UTC 기준의 분할된 시간으로 변환한다. </br>

mktime 함수는 지역 시간 기준의 분할된 시간을 time_t 값으로 변환한다.
        #include <time.h>

        time_t mktime(struct tm *tmptr);
asctime 함수와 ctime 함수는 주어진 시간 값을 date 명령의 기본 출력과 비슷한, 사람이 읽기 좋은 26바이트짜리 문자열로 변환해서 돌려준다. 

        #include <time.h>

        char *asctime(const struct tm *tmptr);
        char *ctime(const time_t *calptr);
asctime의 인수는 분할된 시간 값을 가리키는 포인터이고 ctime의 인수는 달력 시간 구조체를 가리키는 포인터이다.

마지막 시간 함수인 strftime는 가장 복잡한 함수이다. 이 함수는 시간 값을 printf 함수와 비슷한 방식으로 서식화한다.
        #include <time.h>

        size_t strftime(char *restrict buf, size_t maxsize, const char *restrict format, const struct tm *restrict tmptr);
이 함수의 마지막 인수는 서식화할 시간 값으로, 구체적으로 말하자면 분할된 시간 값을 가리키는 포인터이다. </br>
변환 지정자 %U, %V, %W는 셋 다 해당 연도의 주 번호로 확장되는데, %U는 첫번째 일요일을 포함되는 주를 첫번째 주로 간주하고, </br>
%W는 첫번째 월요일을 포함하는 주를 첫번째 주로 간주한다. %V는 1월1일을 포함하는 주에 올해의 날들이 넷 이상이면 그 주가 1번 주가 된다. </br>
그렇지 않으면 그 주는 작년에 속한 주로 간주되고, 그 다음 주가 1번 주가 된다. 두 경우 모두, 한 주의 첫 요일은 월요일이다. </br>
printf류 함수들처럼 strftime 함수는 일부 변환 지정자들에 대해 수정자들을 지원한다. 
