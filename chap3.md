# Chapter 3 파일 I/O

## 3.2 파일 서술자

커널은 열린 파일들을 파일 서술자라는 값을 통해서 참조한다. </br>
프로세스가 기존 파일의 파일을 열거나 새 파일을 생성하면 커널은 그에 해당하는 파일 서술자를 프로세스에게 돌려준다. </br>
특정한 파일을 읽거나 쓸 때에는 그 파일에 대해 open이나 creat가 돌려준 파일 서술자를 read나 write의 인수로 제공함으로써 해당 파일을 지정한다. </br>
POSIX를 준수하는 응용 프로그램에서는 0, 1, 2라는 마법의 수들을 각각 STDIN_FILENO, STDOUT_FILENO, STDERR_FILENO라는 기호 상수들로 대체해야 한다.

## 3.3 open 함수

파일을 열거나 생성할 때 쓰임.

    #include <fcntl.h>

    int open(const char *pathname, int oflag, ... /* mode_t mode */ );

셋째 인수는 ...으로 표기되어 있는데, 이것은 ISO C에서 나머지 인수들의 개수와 형식들이 가변적임을 나타내는 방법이다. </br>
pathname 인수는 열거나 생성할 파일의 이름이다. oflag는 함수의 작동 방식을 결정하는 다양한 옵션들이다.

    필수적
    O_RDONLY: 읽기 전용으로 연다.
    O_WRONLY: 쓰기 전용으로 연다.
    O_RDWR: 읽기 및 쓰기용으로 연다.

    선택적
    O_APPEND: 파일 기록 시 내용을 파일의 끝에 추가한다.
    O_CREAT: 파일이 존재하지 않으면 새로 생성한다.
    O_EXCL: O_CREAT와 이 상수를 함께 지정하면, 파일이 이미 존재하는 경우에 오류가 발생한다.
    O_TRUNC: 이미 존재하는 파일을 읽기 또는 쓰기 모드로 열었을 때 파일의 크기가 0으로 줄어들게 한다.
    O_NOCTTY: pathname에 터미널 장치를 가리키는 경우, 해당 장치를 이 프로세스의 제어 터미널로 배정하지 않도록 한다.
    O_NONBLOCK: pathname이 FIFO나 블록 특수 파일, 문자 특수 파일을 지칭하는 경우, 이 옵션은 그 파일의 열기 및 이후의 I/O 작업들에 대해 비차단 모드를 설정한다.
    O_DSYNC: 각 write 연산이 물리적 I/O 연산의 완료를 기다리게 하되, 방금 기록한 자료를 읽을 수 있는 능력에 미치지 않는 파일 특성 갱신은 기다리지 않도록 한다.
    O_RSYNC: 파일 서술자에 대한 각 read 연산이 파일의 동일한 부분에 대해 유보된 기록의 완료를 기다리게 한다.
    O_SYNC: 각 write 연산이 물리적 I/O의 완료를 기다리게 한다.

### 파일이름과 경로이름의 절단

POSIX.1에는 함수들이 긴 파일이름과 긴 경로이름을 소리 없이 잘라낼 것인지 아니면 오류를 돌려줄 것인지를 결정하는 _POSIX_NO_TRUNC라는 상수가 있다. </br>
_POSIX_NO_TRUNC가 설정된 상황에서 경로이름의 길이가 PATH_MAX보다 크거나 경로이름의 파일이름 성분이 하나라도 NAME_MAX보다 길면 errno가 ENAMETOOLONG으로 설정되며, 함수는 오류 코드를 반환한다.

## 3.4 creat 함수

새 파일은 creat 함수로도 생성 가능하다.

    #include <fcntl.h>

    int creat(const char *pathname, mode_t mode);

open(pathname, O_WRONLY | O_CREAT | O_TRUNC, mode); -> creat 함수는 다음 호출과 동일하다.

## 3.5 close 함수

열린 파일을 닫을 때는 close 함수를 호출한다.

    #include <unistd.h>

    int close(int filedes);

파일을 닫으면 프로세스가 그 파일에 대해 잠근 레코드 자물쇠들도 모두 해제된다. </br>
프로세스가 종료되면 커널이 프로세스의 열린 파일들을 모두 자동으로 닫는다.

## 3.6 lseek 함수

모든 열린 파일에는 "현재 파일 오프셋"이 존재한다. 파일 오프셋은 파일의 시작에서부터 센 바이트 수 에 해당하는 음이 아닌 정수이다. </br>
보통의 경우 읽기, 쓰기 연산들은 현재 파일 오프셋에서 시작하며, 연산이 끝나면 읽거나 쓴 바이트 수만큼 오프셋이 증가한다. O_APPEND를 지정하지 않은 이상, 파일을 열면 오프셋이 0으로 초기화된다. </br>
열린 파일의 오프셋을 명시적으로 변경할 때는 lseek 함수를 사용한다.

    #include <unistd.h>

    off_t lseek(int filedes, off_t offset, int whence);

offset에 주어진 값의 적용 방식은 whence 인수에 주어진 값에 따라 다르다. </br>
1. whence가 SEEK_SET이면 파일의 오프셋은 offset으로 설정된다. </br>
2. whence가 SEEK_CUR이면 파일의 오프셋은 현재 값에 offset을 더한 값으로 설정된다. </br>
3. whence가 SEEK_END이면 파일의 오프셋은 파일의 크기 더하기 offset으로 설정된다.

호출 성공 시 lseek 함수는 새 파일 오프셋을 돌려주므로 현재 위치로부터 0 떨어진 곳을 찾음으로써 현재 오프셋의 값을 알아낼 수 있다.

