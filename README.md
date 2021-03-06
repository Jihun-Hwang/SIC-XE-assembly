## 개요
해당 프로젝트는 [백준 1992번 '쿼드트리'문제](https://www.acmicpc.net/problem/1992)를 SIC/XE 기반의 어셈블리 언어로 해결합니다.

## 주요 알고리즘 개요

입력의 크기 N으로부터 2차원 배열의 크기가 N x N임을 알 수 있다. N x N이라는 해결해야 할 문
제의 크기를 4분의 1크기의 4개 조각으로 쪼개어 작은 문제로 만들고 재귀적으로 해결한다. 어셈
블리 코드에서 SOLVE라는 recursive subroutine은 다음과 같이 3개의 argument를 받는다.
- 쪼개어진 문제의 크기 N
- 쪼개어진 문제(2차원 배열)의 첫번째 Y index (tmpY)
- 쪼개어진 문제(2차원 배열)의 첫번째 X index (tmpX)
4분의 1크기로 줄어든 구역의 모든 원소 값들이 같은 값을 가져야 압축할 수 있다. 그렇지 않다
면 압축할 수 있을 때까지(N==1) 문제를 다시 재귀적으로 4등분해 가면서 해결한다. 

## 주요 알고리즘 시나리오

* 2차원 배열에 입력이 들어와 있다고 가정
1. 인자 N이 1인지 검사한다. 1이라면 2차원 배열의 해당 요소를 출력하고 subroutine을 빠져나온
다. (재귀 탈출 조건 검사)

2. 인자 tmpY, tmpX를 통해 2차원 배열에서의 현재 구역의 첫번째 요소 값을 계산하고 기준 값
(pivot)으로 설정한다.

3. 현재 구역의 모든 요소를 순차적으로 확인하면서 pivot과 같은 값을 갖는지 검사하고 구역의
모든 값이 pivot과 같은 값을 갖는다면 pivot을 출력하고 subroutine을 빠져나온다.

4. 구역의 값 중 pivot과 다른 값이 있다면 현재 구역을 왼쪽 위, 오른쪽 위, 왼쪽 아래, 오른쪽
아래 네 구역으로 동일하게 나누고, 순서대로 네 번의 recursive subroutine을 호출한다. 현재 구역
이 쪼개져서 압축되었음을 나타내기 위해 네 번의 recursive subroutine을 호출하기 전과 후에 ‘(’
와 ‘)’를 각각 출력한다.

## 고찰

1. Call Stack이 어떻게 사용되는지 이해했다. Call Stack은 현재 실행 중인 subroutine이 다른 subroutine을 호출하기 전에 백업해 두어야 할 자신의 정보를 Stack에 저장하기 위해 사용된다. 이 중에서, 복귀 주소를 백업해 두는 것이 매우 중요한데, L레지스터 값이 갱신되는 JSUB명령어를 실행시키기 이전에, 반드시 stack에 L레지스터의 값을 push해 두어야 한다. JSUB을 통해 subroutine을 호출하고, 해당 subroutine이 복귀하면 다시 stack으로부터 pop해서 저장해 두었던 정보들을 복구시킨다. 특히, RSUB명령어는 L레지스터에 쓰인 값을 PC로 옮기기 때문에, RSUB전에는 반드시 L레지스터에 복귀 주소가 있어야 한다. 

2. high level 언어로 선언된 2차원 배열이 메모리에 저장되는 구조와 접근할 주소 계산 방식에 대해 이해했다. 다차원 배열이라 할 지라도 메모리 구조는 바이트주소 단위로 연속적으로 존재하기 때문에 표현할 행과 열을 index를 적절히 조합하여 2차원 배열에서의 읽을 주소 값을 계산할 수 있었다.

3. 메모리 공간을 많이 사용하는 stack과 2차원 배열 선언은 어셈블리 코드의 가장 마지막에 위치시키는 것이 좋다는 것을 알았다. Stack과 2차원배열과 같이 메모리를 많이 차지하는 것들이 어셈블리코드의 중간지점에 위치한다면 format 4를 사용해야 된다는 낭비가 발생할 수 있다. Format 4를 최대한 적게 사용한다면 프로그램의 길이가 짧아져 메모리 자원을 효율적으로 사용할 수 있다. 따라서 stack과 2차원 배열처럼 메모리를 많이 잡는 것들은 어셈블리 코드의 가장 아래에 위치시킨다면 해당 symbol에 접근하는 disp를 최대한으로 활용할 수 있다. 하지만 이번과제에서는 stack과 2차원 배열 모두 그리 많은 메모리를 필요로 하지 않기 때문에, format 4를 사용할 이유도 없었으며, stack과 2차원 배열을 프로그램의 마지막에 위치시킬 이유도 없었다.

4. 메모리를 최소한으로 사용하기 위한 방법을 3가지 생각해 봤다. 
첫 번째로, 입력 크기를 제외하면 ‘0’과 ‘1’에 해당하는 아스키코드 값 0x30, 0x31만이 입력된다. 따라서 두 값은 모두 1byte로 표현 가능하므로 2차원 배열의 한 요소 값을 byte 단위로 할당할 수 있다. 하지만 byte로 저장하므로 2차원 배열로부터 값을 읽고 레지스터에 로드하는 과정에 추가적으로 instruction이 필요하게 되어 메모리를 절약하는데 크게 도움이 되지 못한다.
두 번째로 recursive subroutine에 필요한 call stack의 크기를 최소화하는 것이다. 
최대 입력 크기 N이 8이므로 최대 동시에 push되는 stack frame이 3개이다. 이 값은 바꿀 수 없기 때문에 stack frame의 크기를 줄이면 메모리 사용을 줄일 수 있다. 현재 코드에서는 stack frame에 tmpY, tmpX, tmpN, tmpL, half 5개의 word가 포함된다. 이 중에서 tmpX, tmpY는 다음 subroutine 호출 시 필요하고 tmpL은 복귀 주소를 위해 필요하다. 하지만 half의 경우 tmpN을 push하기 때문에 굳이 push할 필요가 없어진다. 하지만 half 값(tmpN / 2)을 push하지 않는다면 stack으로부터 tmpN을 pop한 뒤 다시 DIV #2 명령어가 추가적으로 필요로 되기 때문에 메모리 절약에 크게 도움이 되지 않는다.
마지막으로 Immediate addressing 최대화이다.
Immediate addressing을 최대화하면 메모리까지 접근하지 않아도 된다는 점에서 성능적인 효과가 있고, 그만큼 메모리에 할당하는 변수의 개수가 적어진다는 점에서 메모리 사용을 줄일 수 있다. 따라서 I/O device number, 각종 산술 연산에 필요한 숫자들은 immediate addressing을 최대한으로 사용했다. 하지만 ‘(‘, ‘)’ 값이나 개행 값 등 아스키코드 값들은 # 키워드를 붙인 숫자 값으로 표현했을 때 한눈에 이해하기가 어려워서 immediate addressing을 사용하지 않았다.

## Reference

- [문제출처](https://www.acmicpc.net/problem/1992)
- [C++코드](https://github.com/Jihun-Hwang/Algorithm_Study/blob/main/DivideAndConquer/1992_%EC%BF%BC%EB%93%9C%ED%8A%B8%EB%A6%AC.cpp)
