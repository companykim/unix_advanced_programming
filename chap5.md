# chapter 5 표준 I/O 라이브러리

## 5.2 스트림과 FILE 객체

표준 I/O 라이브러리에서 파일을 생성하거나 열면 파일 스트림을 얻게 되는데, 이를 "스트림을 파일에 연결시켰다"고 말한다. </br>
표준 I/O 파일 스트림은 단일 바이트 문자 집합과 사용할 수도 있고 다중 바이트 문자 집합과 사용할 수도 있다. </br>
문자가 단일 바이트로 접근되는지 다중 바이트로 접근 되는지를 결정하는 것은 스트림의 지향이다. 한 스트림이 처음 생성되었을 때에는 스트림에 지향이 없는 상태이다. </br>
지향이 없는 스트림에 대해 다중 바이트 I/O 함수가 호출되면 스트림의 지향은 넓은 문자 지향이 된다. </br>
지향이 없는 스트림에 바이트 I/O 함수가 호출되면 스트림의 지향은 바이트 지향이 된다. </br>
fropen 함수로는 스트림의 지향을 지울 수 있고, fwide 함수로는 스트림의 지향을 설정할 수 있다.
    #include <stdio.h>
    #include <wchar.h>

    int fwide(FILE *fp, int mode);
fwide 함수의 행동은 mode 인수에 주어진 값에 따라 달라진다. </br>
1. mode가 음수이면 fwide는 지정된 스트림을 바이트 지향으로 설정하려고 한다.
2. mode가 양수이면 fwide는 지정된 스트림을 넓은 문자 지향으로 설정하려고 한다.
3. mode가 0이면 fwide는 지향을 설정하지 않고 그냥 스트림의 현재 지향을 뜻하는 값을 돌려준다. </br>
지향이 이미 설정되어 있는 스트림에 대해서는 fwide가 지향을 변경하지 않음을 주의해야 한다. 또한 이 함수가 오류코드를 반환하지 않는다는 것도 중요하다. </br>
표준 I/O 함수 fopen으로 스트림을 열면 fopen은 FILE 객체를 가리키는 포인터를 돌려준다. </br>
FILE은 표준 I/O 라이브러리가 스트림을 관리하는 데 필요한 모든 정보를 담은 하나의 구조체이다. 그러한 정보에는 실제 I/O에 쓰이는 파일 서술자, 스트림을 위한 버퍼를 가리키는 포인터, 그 버퍼의 크기, 버퍼에 현재 들어 있는 문자 개수, 오류 플래그 등이 포함된다. </br>
스트림을 사용할 때에는 그냥 해당 FILE 포인터를 각 표준 I/O 함수의 인수로 지정하면 된다.

## 5.3 표준 I/O 스트림들 - 표준 입력, 표준 출력, 표준 오류

모든 프로세스에는 세 가지의 미리 정의된 스트림들이 자동으로 제공된다. </br>
이 스트림들은 표준 파일 서술자 STDIN_FILENO, STDOUT_FILENO, STDERR_FILENO와 동일한 파일들을 가리킨다. </br>
프로그램 안에서는 이 세 표준 I/O 스트림들을 미리 정의된 파일 포인터 stdin, stdout, stderr로 참조한다. 

## 5.4 버퍼링

표준 I/O 라이브러리가 제공하는 버퍼링의 목표는 read 호출과 write 호출의 횟수를 최소화하는 것이다. </br>
표준 I/O 라이브러리는 각 I/O 스트림에 대해 버퍼링을 자동으로 적용함으로써 프로그래머가 버퍼링에 신경 쓰는 일이 없도록 한다. 

표준 I/O 라이브러리에서 제공하는 버퍼링은 다음과 같다.

1. 전체 버퍼링. 이 경우 실제 I/O는 표준 I/O 버퍼가 꽉 찼을 때에만 일어난다. 보통의 경우 표준 I/O 라이브러리는 디스크에 있는 파일들에 이 전체 버퍼링을 적용한다. </br>
전체 버퍼링에 쓰이는 버퍼는 주어진 한 스트림에 대해 I/O가 처음 수행될 때 표준 I/O 함수가 malloc을 호출해서 할당한다. </br>
표준 I/O 버퍼의 내용을 디스크에 기록하는 작업을 방출이라고 한다. 버퍼는 표준 I/O 루틴이 자동으로 방출할 수도 있고, 프로그램의 fflush 호출에 의해 명시적으로 방출될 수도 있다. </br>
UNIX 환경에서 방출은 표준 I/O 라이브러리의 맥락에서는 버퍼 내용의 기록을 의미하지만, 터미널 드라이버의 맥락에서는 버퍼에 저장되어 있는 자료의 폐기를 의미한다. 
2. 줄 단위 버퍼링. 이 경우 표준 I/O 라이브러리는 입력이나 출력에서 새줄 문자를 만났을 때 I/O를 수행한다. 이 버퍼링 하에서 텍스트를 출력할 때 실제 I/O는 하나의 줄이 완성되었을 때만 일어나므로, 텍스트를 한 번에 한 문자씩만 출력해도 성능이 크게 떨어지지 않는다. </br>
줄 단위 버퍼링은 터미널을 가리키는 스트림에 주로 쓰인다. </br>

줄 단위 버퍼링에서는 다음을 주의해야 한다. </br>
1. 표준 I/O 라이브러리가 각 줄의 버퍼링에 사용하는 버퍼의 크기는 고정되어 있으며, 새줄 문자가 나타나기 전에라도 버퍼가 꽉 차면 실제 I/O가 수행된다.
2. 표준 I/O 라이브러리를 통해서 (a) 버퍼링 되지 않는 스트림이나 (b) 줄 단위 버퍼링 스트림으로부터의 입력이 요청되면, 줄 단위 버퍼링 출력 스트림들이 모두 방출된다.
단, (b)는 자료를 커널이 요청한 경우에만 해당한다. 요청된 자료가 이미 버퍼에 들어 있어서 커널이 자료를 읽을 필요가 없는 경우에는 해당하지 않는다. (a)의 경우, 즉, 버퍼링이 없는 스트림에서는 모든 입력에 대해 커널이 자료를 읽어야 한다.
3. 버퍼링 없음. 이 경우 표준 I/O 라이브러리는 문자들을 버퍼링하지 않는다. 보통의 경우 표준 오류 스트림은 버퍼링되지 않는다. </br>
ISO C는 다음을 요구한다. (1. 표준 입력과 표준 출력은 전체 버퍼링이어야 한다. 2. 표준 오류는 결코 전체 버퍼링이 아니다.)
대부분의 구현들은 다음과 같다. (1. 표준 오류는 항상 버퍼링되지 않는다. 2. 그 외의 스트림들은 터미널 줄을 가리키는 경우에는 줄 단위 버퍼링이고 그렇지 않으면 전체 버퍼링이다.)

기본 버퍼링 방식이 적합하지 않은 스트림이 있다면 다음 두 함수를 사용해 스트림의 버퍼링 방식을 변경할 수 있다.
    #include <stdio.h>

    void setbuf(FILE *restrict fp, char *restrict buf);
    int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
이 함수들은 반드시 스트림을 연 후(두 함수 모두 첫 인수로 유효한 파일 포인터를 받으므로 당연한 일이다), 그러나 스트림에 대해 다른 연산을 수행하기 전에 호출해야 한다. </br>
setbuf로는 버퍼링을 켜거나 끌 수 있다. 버퍼링을 켜려면 길이가 BUFSIZ인 버퍼를 가리키는 포인터를 buf 인수에 지정해야 한다. 그러면 fp가 가리키는 스트림에 대해 전체 버퍼링이 설정되는 것이 보통이나, 스트림이 터미널 장치에 연관되어 있다면 줄 단위 버퍼링을 설정하는 구현도 있다. </br>
버퍼링을 끄려면 buf에 NULL을 넣으면 된다. 

버퍼는 언제라도 방출 가능하다. 
    #include <stdio.h>

    int fflush(FILE *fp);
이 함수는 스트림에 대해 아직 기록되지 않은 자료를 커널에 넘겨준다. 

## 5.5 스트림 열기

        #include <stdio.h>

        FILE *fopen(const char *restrict pathname, const char *restrict type);
        FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
        FILE *fdopen(int filedes, const char *type);
1. fopen 함수는 지정된 파일을 연다. </br>
2. freopen 함수는 지정된 파일을 지정된 스트림으로 연다. 스트림이 이미 열려 있으면 닫은 후 다시 연다. </br>
3. fdopen 함수는 기존 파일 서술자를 받아서 그 서술자에 표준 I/O 스트림을 연관시킨다. </br>

type의 값들에서 문자 b는 표준 I/O 시스템이 파일의 내용을 텍스트가 아니라 이진 값으로 다루도록 지정하는 효과를 낸다. </br>
fdopen의 type 인수는 의미가 좀 다른데, 이 함수는 이미 열려 있는 파일에 대한 서술자를 받으므로 스트림을 쓰기용으로 연다고 해도 파일이 절단되지는 않는다. 또한, 표준 I/O 추가 모드로 파일을 생성하지 못한다. 

읽기 및 쓰기를 위해 파일을 여는 경우, 다음과 같은 제약이 적용된다.
1. 출력 이후 즉시 입력을 수행할 수 없는데, fflush나 fseek, fsetpos, rewind를 거쳐야 한다.
2. 입력 이후 즉시 출력을 수행할 수 없는데, fseek, fsetpos, rewind를 거치거나 입력 연산이 파일의 끝을 만나야 한다.

열린 스트림을 닫을 때에는 fclose 함수를 사용한다. 
        #include <stdio.h>

        int fclose(FILE *fp);
파일이 닫히기 전에 버퍼에 있는 출력 자료가 방출된다. 

## 5.6 스트림 읽고 쓰기

스트림을 연 후에는 스트림에 대해 세 종류의 서식 없는 I/O 연산들을 수행할 수 있다.
1. 문자 단위 I/O. 한번에 한 문자씩 읽거나 쓸 수 있다.
2. 줄 단위 I/O. 한번에 한 줄씩 읽거나 쓰고 싶은 경우에는 fgets 함수나 fputs 함수를 사용한다. 각 줄은 세줄 문자로 끝난다.
3. 직접 I/O. 이런 종류의 연산은 fread 함수와 fwrite 함수가 지원한다. 각 I/O 연산마다 지정된 크기의 여러 객체들을 읽거나 쓴다.

### 입력 함수들

다음 세 함수들은 한번에 하나의 문자를 읽어들인다. 
        #include <stdio.h>

        int getc(FILE *fp);
        int fgetc(FILE *fp);
        int getchar(void);
getchar 함수는 getc(stdin)과 동등하게 정의된다. getc와 fgetc의 차이는 getc가 매크로로 구현될 수 있는 반면, fgetc는 그럴 수 없다.
1. getc의 인수에 side effect를 가지는 표현식을 지정하지는 말아야 한다.
2. fgetc는 반드시 실제의 함수이므로 그 주소를 취할 수 있다. 예를 들어 fgetc의 주소를 다른 함수의 인수로 지정할 수 있다.
3. fgetc 호출은 getc보다 더 많은 시간을 소비할 수 있다.

이 세 함수들은 스트림의 다음 문자를 돌려주는데, 문자 자체는 unsigned char 형식으로 취급되지만, 반환값은 그것을 int로 변환한 결과이다. </br>
문자가 unsigned char 형식인 이유는 최상위 비트가 설정되어 있다고 해도 음수가 되지 않기 때문이다. </br>
이 세 함수들이 오류가 발생했을 때와 파일 끝에 도달했을 때 모두 동일한 값을 돌려줌을 주의할 것. 두 상황을 구분하려면 ferror 함수나 feof 함수를 호출해야 한다.

        #include <stdio.h>

        int ferror(FILE *fp);
        int feof(FILE *fp);
        void clearerr(FILE *fp);
대부분의 구현은 각 스트림의 FILE 객체에 오류 플래그, 파일 끝 플래그를 둔다. </br>
clearerr를 호출하면 두 플래그 모두 해제된다. </br>
스트림에서 문자를 읽은 후 그 문자를 다시 스트림에 되돌려 놓는 것도 가능하다. ungetc 함수가 그런 일을 한다.
        #include <stdio.h>

        int ungetc(int c, FILE *fp);

이 함수로 스트림에 되돌려 넣은 문자는 다음 번 문자 읽기에서 반환된다. 문자들을 여러 번 읽고 여러 번 되돌려 넣은 경우, 문자들은 되돌려 넣은 순서의 역순으로 다시 읽힌다. </br>

### 출력 함수
        #include <stdio.h>

        int putc(int c, FILE *fp);
        int fputc(int c, FILE *fp);
        int putchar(int c);
입력 함수들처럼, putchar는 putc와 동등하며, putc는 매크로로 구현될 수 있는 반면 fputc는 그럴 수 없다.

## 5.7 줄 단위 I/O

줄 단위 입력
        #include <stdio.h>

        char *fgets(char *restrict buf, int n, FILE *restrict fp);
        char *gets(char *buf);
두 함수 모두 한줄을 읽어 들일 버퍼의 주소를 받는다. gets 함수는 표준 입력을 읽는 반면 fgets는 지정된 스트림을 읽는다. </br>
fgets는 n 인수에 버퍼의 크기를 넣어야 한다. 

fputs와 puts 함수는 줄 단위 출력을 수행한다.
        #include <stdio.h>

        int fputs(const char *restrict str, FILE *restrict fp);
        int puts(const char *str);
fputs 함수는 주어진 널 종료 문자열을 주어진 스트림에 기록한다. puts 함수는 주어진 널 종료 문자열을 표준 출력에 기록한다.

## 5.8 표준 I/O의 효율성

반드시 함수여야 하는 fgetc와 fputc를 이용해서 같은 일을 수행하는 프로그램을 만드는 것도 물론 가능하다. </br>\
ex1, ex2에서 표준 I/O 스트림들을 명시적으로 닫지는 않음을 주목해야 한다. 어차피 exit 함수에 의해 버퍼의 자료가 방출되고 열린 스트림들이 모두 닫힐 것이기 때문이다.

## 5.9 이진 I/O 

이진 자료에 대해 I/O를 수행할 때에는 한번에 하나의 구조체 전체를 처리하게 되는 것이 일반적이다. </br>
getc나 putc로 이러한 작업을 수행한다면 구조체 전체를 한번에 한바이트씩 읽거나 쓰는 루프를 돌려야 할 것이므로 비효율적이다. 이러한 작업에서는 줄 단위 함수들도 쓸모가 없다. </br>
왜냐하면 구조체 안에는 널 바이트들이 존재할 수 있는데 fputs나 fgets는 널 바이트를 만나면 읽기나 기록을 멈추기 때문이다. 이러한 이유로, 이진 I/O를 위한 두 개의 개별적인 함수가 제공된다. 
        #include <stdio.h>

        size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
        size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
이 함수들의 주된 용도는 </br>
1. 이진 배열을 읽거나 쓴다.
2. 하나의 구조체를 읽거나 쓴다.

이진 I/O의 한 가지 근본적인 문제는 한 시스템에서 이진 I/O로 기록된 자료를 그와 동일한 시스템에서만 읽을 수 있다는 것이다.

## 5.10 스트림 위치 조회 및 설정

표준 I/O 스트림의 읽기, 쓰기 위치를 조회하거나 설정하는 방법 </br>
1. ftell 함수와 fseek 함수를 이용한다. 파일의 위치를 하나의 긴 정수(long)에 저장할 수 있다고 가정한다는 문제가 있다.
2. ftello 함수와 fseeko 함수를 이용한다. 이들은 긴 정수로는 적합하지 않은 파일 오프셋들을 허용하기 위해 단일 UNIX 규격이 제공하는 것으로 , 긴 정수 대신 off_t 형식을 사용함.
3. fgetpos 함수와 fsetpos 함수를 이용한다. 이들은 ISO C의 일부이며, 파일의 위치를 fpos_t라는 추상 자료 형식에 담는다. 구현들은 이 형식을 파일의 위치를 담는데 충분한 크기의 구체적인 자료 형식으로 정의할 수 있다.

비 UNIX 시스템으로의 이식성까지 갖추어야 하는 응용프로그램이라면 반드시 fgetpos와 fsetpos를 사용해야 한다.
        #include <stdio.h>

        long ftell(FILE *fp);
        int fseek(FILE *fp, long offset, int whence);
        void rewind(FILE *fp);
이진 파일의 경우 파일 위치 지시자는 파일 시작을 기준으로 측정한 현재 읽기/쓰기 위치이다. 이진 파일에 대해 ftell이 돌려주는 값이 이 바이트 위치이다. </br>
fseek로 이진 파일의 위치를 변경할 때에는 offset 인수에 이동량을, whence 인수에 이동의 기준을 지정해야 한다. whence의 값은 lseek 함수의 해당 인수의 값과 동일하다. </br>
즉, SEEK_SET은 파일의 시작, SEEK_CUR는 현재 파일 위치, SEEK_END는 파일의 끝을 의미한다. </br>
ftello 함수와 fseeko 함수는 ftell, fseek 함수와 비슷하되, 반환값의 형식이 long이 아니라 off_t라는 점에서 차이를 보인다. 
        #include <stdio.h>

        off_t ftello(FILE *fp);
        int fseeko(FILE *fp, off_t offset, int whence);

        #include <stdio.h>

        int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
        int fsetpos(FILE *fp, const fpos_t *pos);

fgetpos 함수는 파일 위치 지시자의 현재 값을 pos가 가리키는 객체에 저장한다. 이후 이 값을 fsetpos 호출에서 사용함으로써 스트림을 지금의 위치로 되돌릴 수 있다.

## 5.11 서식화된 I/O

### 서식화된 출력
        #include <stdio.h>

        int printf(const char *restrict format, ...);
        int fprintf(FILE *restrict fp, const char *restrict format, ...);
        int sprintf(char *restrict buf, const char *restrict format, ...);
        int snprintf(char *restrict buf, size_t n, const char *restrict format, ...);
printf 함수는 표준 출력에, fprintf 함수는 지정된 스트림에, sprintf 함수는 buf 배열에 서식화된 문자들을 기록한다. </br>
sprintf 함수는 배열의 끝에 자동으로 널 바이트 하나를 추가하나, 이 널 바이트가 반환값(문자 개수)에 포함되지는 않는다. </br>
sprintf 호출 시 buf가 가리키는 버퍼가 넘칠 수 있음을 주의해야 함. 잠재적인 버퍼 넘침 문제 때문에 도입된 것이 snprintf라는 함수다. 이 함수를 호출할 때에는 인수로 버퍼의 크기를 명시적으로 지정한다. 기록할 문자의 개수가 버퍼의 크기를 넘으면 여분의 문자들은 그냥 폐기된다. snprintf 함수는 버퍼가 충분히 컸다면 기록되었을 문자들의 개수를 돌려준다. </br>

format 인수는 서식 문자열 또는 서식 명세이다. 서식 문자열은 퍼센트 기호(%)로 시작하는 변환 명세들과 그 외의 문자들로 구성된다. </br>
변환 명세들은 format 인수 다음의 인수들이 부호화, 표시되는 방식을 결정하는데, 변환 명세들 이외의 문자들은 수정없이 그대로 출력된다. </br>
하나의 변환 명세의 구성은
        % [플래그] [필드너비] [정밀도] [길이 수정자] 변환형식
필드너비 성분은 변환의 최소 필드 너비를 지정. 정밀도 성분은 변환 결과의 정밀도를 지정하는 것. 필드 너비나 정밀도에 특정 수치 대신 별표(*)를 지정하는 것도 가능함. </br>
길이 수정자 성분은 인수의 크기를 지정함. 변환 형식 성분은 반드시 지정해야 한다. 이것은 인수의 해석 방식을 결정한다. 
        #include <stdarg.h>
        #include <stdio.h>

        int vprintf(const char *restrict format, va_list arg);
        int vfprintf(FILE *restrict fp, const char *restrict format, va_list arg);
        int vsprintf(char *restrict buf, const char *restrict format, va_list arg);
        int vsnprintf(char *restrict buf, size_t n, const char *restrict format, va_list arg);

### 서식화된 입력

        #include <stdio.h>

        int scanf(const char *restrict format, ...);
        int fscanf(FILE *restrict fp, const char *restrict format, ...);
        int sscanf(const char *restrict buf, const char *restrict format, ...);
scanf류 함수들은 하나의 입력 스트림을 분석해서 문자열 조각들을 지정된 형식의 변수들에 배정하는데 쓰인다. format 인수 이후의 인수들은 변환 결과가 기록될 변수들의 주소들이다. </br>
인수들이 어떻게 변환, 배정되는지는 format으로 지정된 서식 문자열이 결정한다. 서식 문자열의 한 변환 명세는 % 기호로 시작한다. </br>
변환 명세의 구성
        % [*] [필드너비] [길이 수정자] 변환형식
생략 가능한 선행 발표는 배정이 일어나지 않게 만든다. </br>
필드너비 성분은 문자 단위의 최대 필드 너비를 지정한다. 길이 수정자 성분은 변환 결과가 배정될 인수의 크기를 지정한다. </br>
변환 형식 성분은 printf류 함수들에 쓰이는 값과 비슷하되 몇가지 차이가 존재한다. 하나는 입력의 부호 있는 정수가 부호 없는 정수 형식의 인수에 저장될 수 있다는 것이다. </br>
scanf 함수들에도 <stdarg.h>에 지정된 가변 인수 목록들을 사용하는 버전들이 존재한다. </br>
        #include <stdarg.h>
        #include <stdio.h>

        int vscanf(const char *restrict format, va_list arg);
        int vfscanf(FILE *restrict fp, const char *restrict format, va_list arg);
        int vsscanf(const char *restrict buf, const char *restrict format, va_list arg);

## 5.12 구현 세부사항

UNIX 시스템에서 표준 I/O 라이브러리 루틴들은 내부적으로 I/O 루틴들을 호출해서 작업을 수행한다. </br>
표준 I/O 스트림마다 그에 연관된 파일 서술자가 존재하는데, 그 파일 서술자는 fileno 함수로 얻을 수 있다.
        #include <stdio.h>

        int fileno(FILE *fp);
이 함수는 이를테면 dup나 fcntl 함수를 호출하고자 할 때 필요하다.

(ex3) 이 프로그램은 각 스트림에 대해 버퍼링 정보를 출력하기 전에 I/O 연산을 출력해야 한다. 이는 보통의 경우 스트림에 대한 버퍼가 최초의 I/O 연산에서 비로소 할당되기 때문이다. </br>
구조체 멤버 _IO_file_flags_, _IO_buf_base, _IO_buf_end와 상수 _IO_UNBUFFERED, _IO_LINE_BUFFERED는 linux 가 사용하는 GNU 표준 I/O 라이브러리가 정의한 것이다. </br>
이 시스템의 경우 터미널에 연결된 표준 입력과 표준 출력에 대해서는 줄 단위 버퍼링이 쓰임을 알 수 있다. 

## 5.13 임시 파일

ISO C 표준은 임시 파일 생성을 돕는 두가지 표준 I/O 라이브러리 함수를 정의한다.
        #include <stdio.h>

        char *tmpnam(char *ptr);
        FILE *tmpfile(void);
tmpnam 함수는 기존의 어떤 파일과도 이름이 같지 않은 유효한 경로이름을 담은 문자열을 생성한다. 이 함수는 최대 TMP_MAX번까지 호출될 때마다 다른 이름을 생성한다. </br>
ptr이 NULL이면 생성된 경로이름은 정적 영역에 저장되며, 함수는 그 영역을 가리키는 포인터를 돌려준다. 이후, tmpnam을 다시 호출하면 그 정적 영역에 새 이름이 덮어 쓰인다. </br>
ptr이 NULL이 아니면 함수는 ptr이 적어도 L_tmpnam개의 문자들을 담을 수 있는 배열을 가리킨다고 가정한다. 이 경우 생성된 경로이름은 그 배열에 저장되며 함수는 ptr을 돌려준다.

(ex4) tmpfile의 전형적인 구현 방식은 tmpnam으로 고유한 경로이름을 얻고, 그것으로 파일을 생성하고, 그에 대해 즉시 unlink를 호출하는 것이다. </br>
파일을 언링크해도 그 즉시 내용이 삭제되지는 않는다. 파일이 닫혀야 실제로 삭제된다. 이러한 특성 덕분에, tmpfile를 사용하는 프로그램이 임시 파일을 명시적으로 삭제해 주지 않아도 된다. </br>

단일 UNIX 규격의 한 XSI 확장에는 임시 파일에 대한 두가지의 또 다른 함수가 정의되어 있다.
        #include <stdio.h>

        char *tempnam(const char *directory, const char *prefix);
tempnam 함수는 tmpnam과 비슷하되, 생성된 경로이름의 디렉토리와 접두어를 지정할 수 있다는 점에서 차이를 보인다. </br>
디렉토리의 경우, 함수는 다음 네가지 방식을 차례로 시도해서 가장 먼저 성공한 방식의 디렉토리 이름을 사용한다.
1. 환경 변수 TMPDIR가 정의되어 있으면 그 변수의 값이 디렉토리로 쓰인다.
2. directory가 NULL이 아니면 그것이 디렉토리로 쓰인다.
3. <stdio.h>의 p_tmpdir 문자열이 정의되어 있으면 그것이 디렉토리로 쓰인다.
4. 지역 디렉토리가 디렉토리로 쓰인다.

prefix 인수가 NULL이 아니면 그것이 가리키는 문자열의 파일이름의 접두어로 쓰인다. 그 문자열은 반드시 5자 이하이어야 한다. </br>
이 함수는 malloc 함수를 호출해서 할당한 메모리에 경로이름을 저장한다. 

(ex5) 주어진 명령줄 인수가 빈칸으로 시작하면 이 프로그램은 널 포인터를 함수에 넘겨준다. 

XSI가 정의하는 또다른 임시 파일 관련 함수는 mkstemp이다. 이것은 tmpfile과 비슷하되 파일 포인터가 아니라 파일 서술자를 돌려준다는 점에서 차이가 난다. 
        #include <stdlib.h>

        int mkstemp(char *template);
반환된 파일 서술자는 읽기 및 쓰기용으로 열린 것이다. template 인수는 임시 파일의 이름의 틀로 쓰인다. 이 인수에는 xxxxxx로 끝나는 경로이름 문자열을 지정해야 한다. </br>
호출 성공시 함수는 template의 xxxxxx 부분이 적절히 치환된 고유한 경로이름을 만들어서 임시 파일을 생성한다. </br> 
tmpfile과 달리, mkstemp로 생성된 임시 파일은 자동으로 제거되지 않는다. 그 임시 파일을 파일 시스템 이름 공간에서 제거하려면 명시적으로 파일을 언링크해야 한다. </br>
tmpnam이나 tempnam에는 한가지 단점이 있다. 이 함수들을 사용할 때의 문제는 응용 프로그램이 고유한 경로이름을 얻은 시점과 그것으로 파일을 생성한 시점 사이에 시간 여유가 존재한다는 것이다. </br>
그 시간 여유 도중에 다른 프로세스가 동일한 이름으로 파일을 생성할 수도 있다. 따라서 그런 문제가 없는 tmpfile 함수나 mkstemp 함수를 사용하는 것이 바람직하다.
