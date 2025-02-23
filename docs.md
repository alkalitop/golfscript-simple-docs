# GolfScript

# 1. 자료형

GolfScript에는 4가지 자료형이 있다: integer(정수), array(배열), string(문자열), block(블록)

## 1-1. integer

```ruby
7
```

그냥 정수 그 자체.

## 1-2. array

```ruby
[1 2 3]
```

GolfScript의 array는 deque(doubly ended queue)와 유사하다. `[`와 `]`를 이용하여 표기하고, 각 원소는 공백으로 구분한다. 음수 인덱스로 뒤에서부터 접근하는 것이 가능하다.

## 1-3. string

```ruby
'2357'
"2357"
```

**string은 integer(또는 그 값에 해당하는 ASCII 문자)로 이루어진 array**인데, 일반적인 array와는 출력 방식이 다르다. `'`또는 `"`를 이용하여 표기한다.

### 1-3-1. raw string

`'`를 이용하여 표기한 string을 raw string이라고 한다. (아마도) 컴파일러에 의해 escaped string으로 자동변환된다.

```ruby
'\n'
>> "\\n"

' \' '
>> " ' "

```

raw string에서는 오직 역슬래시(\)와 작은 따옴표(‘)만 escape된다. 즉 개행문자(\n)를 raw string에 넣게 되면 \만 escape되기 때문에 escaped string으로 변환될 때 개행문자로 인식되지 않는 문제가 발생할 수 있으니 주의.

### 1-3-2. escaped string

```ruby
"\n"
>> "\n"

"\144"
>> "d"
```

`"`를 이용하여 표기한 string을 escaped string이라고 한다.

## 1-4. block

```ruby
{-1* -}
```

block은 `{`와 `}`를 이용하여 표기한다. 해당 block이 실행을 위한 코드를 나타내는 경우 이외에는 문자열로 취급된다.

# 2. 동작 원리

GolfScript 프로그램은 여러 요소를 나열한 것을 후위표기식(postfix notation) 기반으로 실행한다.

## 2-1. 중위표기식 & 후위표기식

### 중위표기식(Infix Notation)

수학에서 일반적으로 사용되며, 연산자를 피연산자 사이에 배치하는 표기식이다. 우리가 그동안 봐왔던 그거 맞다.

### 후위표기식(Postfix Notation)

연산자를 피연산자 뒤에 쓰는 표기식이다. 수식을 앞에서부터 읽어 나가면서 스택을 이용하여 계산한다.

### 중위표기식 → 후위표기식 변환

왼쪽에서 오른쪽으로 토큰을 하나씩 조회한다.

1. 피연산자 토큰은 스택에 넣지 않고 그대로 출력
2. 연산자 토큰은 다음과 같이 적용한다: 스택의 연산자와 현재 연산자의 우선순위를 비교하여 스택의 top 연산자보다 현재 연산자의 우선순위가 높을 때까지 스택 요소를 pop 하고 출력한 후 현재 연산자를 스택에 push한다. (스택이 비어있으면 바로 push하면 된다.)
3. 여는 소괄호를 만나면 스택에 push한다.
4. 닫는 소괄호를 만나면 여는 소괄호가 나올때까지 스택 요소를 pop 하고 출력한다. 그 이후 현재 닫는 소괄호와 스택 top의 여는 소괄호를 삭제한다.
5. 표기식에 문자가 남지 않았다면 stack의 모든 원소를 pop 후 출력한다.
6. 출력된 결과물이 후위표기식이다.

### 후위표기식 계산

왼쪽에서 오른쪽으로 토큰을 하나씩 조회한다.

1. 피연산자 토큰은 스택에 push한다.
2. 연산자 토큰은, 해당 연산자가 필요로 하는 개수만큼의 피연산자를 스택에서 pop하고, 나중에 나온 피연산자부터 연산을 실행한 후 결괏값을 다시 스택에 push한다.

### 예제

```ruby
1 1+ 
output >> 2

8 2/3- 3 2*+
output >> 7
```

# 3. 입출력

## 3-1. 입력

GolfScript에는 입력 연산자나 함수가 따로 존재하지 않는다. 일반적으로 stdin의 입력이 한꺼번에 문자열로 변환되어 스택에 push된 후에 코드가 실행된다.

### ~ (dump operator) 이용

dump operator를 이용하여, 다른 프로그래밍 언어처럼 공백이나 개행문자로 구분된 정수들을 입력받을 수 있다. 단일 정수도 입력받을 수 있다.

```ruby
input << 1 2
~-
output >> -1
```

| expr | stack |
| --- | --- |
| - | 1 2 |
|  | -1 |

## 3-2. 출력

GolfScript에는 출력 연산자나 함수가 따로 존재하지 않는다. 일반적으로 코드 실행이 끝나면, 스택에 남아있는 요소들이 **bottom부터 차례대로** 구분자 없이 출력된다. 

```ruby
input << 1 2
~4-
>> 1 -2
output >> 1-2
```

| expr | stack |
| --- | --- |
| 4- | 1 2 |
| - | 1 2 4 |
|  | 1 -2 |

# 4. 연산자

## 4-1. `!`

### 4-1-1. logical not

args: 1

피연산자가 0 [] “” {} 이면 1을 반환, 그렇지 않다면 0을 반환한다.

```ruby
1!
>> 0

{asdf}!
>> 0

""!
>> 1
```

## 4-2. `;`

### 4-2-1. pop

args: 1

스택 pop 연산을 실행한다.

```ruby
1 2 3;
>> 1 2
```

## 4-3. `\`

### 4-3-1. swap 2

args: 2

스택의 상위 2개 원소의 자리를 서로 바꾼다.

```ruby
1 2 3 \
>> 1 3 2
```

## 4-4. `@`

### 4-4-1. rotate

args: 3

스택의 상위 3개 원소를 회전하여, 3번째 원소가 top에 위치하도록 한다.

```ruby
1 2 3 4 @
>> 1 3 4 2
```

## 4-5. `.`

### 4-5-1. dup

args: 1

스택 top 원소를 복사한 후 스택에 push한다.

```ruby
1 2 3.
>> 1 2 3 3
```

## 4-6. `$`

### 4-6-1. stack ith copy

args: 1(integer)

주어진 정수를 i라 했을 때, 스택의 i번째 원소를 복사한 후 스택에 push한다.

```ruby
1 2 3 4 5  1$
>> 1 2 3 4 5 4
```

### 4-6-2. sort

args: 1(array/string)

주어진 array나 string을 오름차순(ascending order)으로 정렬한다.

```ruby
'asdf'$
>> "adfs"
```

### 4-6-3. sort by

args: 2(array/string, block)

주어진 array나 string의 각각의 원소는 그 원소에 block을 실행한 결괏값을 우선순위로 갖는다. 해당 우선순위에 따라 정렬한다.

```ruby
 [5 4 3 1 2]{-1*}$
 >> [5 4 3 2 1]
```

## 4-7. `+`

### 4-7-1. add

args: 2(integer, integer)

두 정수를 더한다. 

```ruby
5 7+
output >> 12
```

### 4-7-2. concat

args: 2(coerce)

두 요소를 합친다.

```ruby
'asdf'{1234}+
>> {asdf 1234}

[1 2 3][4 5]+
>> [1 2 3 4 5]
```

## 4-8. `-`

### 4-8-1. neg sign

args: 1(integer)

양의 정수 왼쪽에 붙여써서 음수를 나타낸다. 띄어쓰면 subtract로 인식되니 주의.

```ruby
-3
```

### 4-8-2. subtract

args: 2(integer, integer)

두 정수에 대해 뺄셈을 실행한다. 오른쪽 피연산자에 붙여쓰면 neg sign으로 인식되니 주의.

```ruby
5 8-
>> -3

1 2-3+
>> 1 -1

1 2 -3+
>> 1 -1

1 2- 3+
>> 2
```

### 4-8-3. set diff

args: 2(array, array)

두 array의 차집합을 구한다.

```ruby
[5 2 5 4 1 1][1 2]-
>> [5 5 4]
```

## 4-9. `*`

### 4-9-1. mul

args: 2(integer, integer)

두 정수를 곱한다.

```ruby
2 4*
>> 8
```

### 4-9-2. execute times

args: 2(block, integer)

주어진 block을 특정 횟수만큼 반복하여 실행한다. 다른 프로그래밍 언어의 **for 처럼 사용**할 수 있다.

```ruby
2 {2*} 5*
>> 64
 
0 1{.@+}10*
>> 55 89
```

### 4-9-3. repeat

args: 2(array/string, integer)

주어진 array나 string을 특정 횟수만큼 반복한다.

```ruby
[1 2 3]2*
>> [1 2 3 1 2 3]

3'asdf'*
>> "asdfasdfasdf"
```

### 4-9-4. join

args: 2(???)

```ruby
[1 2 3]','*
>> "1,2,3"

[1 2 3][4]*
>> [1 4 2 4 3]

'asdf'' '*
>> "a s d f"

[1 [2] [3 [4 [5]]]]'-'*
>> "1-\002-\003\004\005"

[1 [2] [3 [4 [5]]]][6 7]*
>> [1 6 7 2 6 7 3 [4 [5]]]
```

### 4-9-5. fold

args: 2(array/string, block)

주어진 array나 string의 각 원소에 순차적으로 block 내의 연산을 누적 적용하여 반환한다. 다른 언어의 reduce와 유사하다.

```ruby
[1 2 3 4]{+}*
>> 10

'asdf'{+}*
>> 414
```

## 4-10. `/`

### 4-10-1. div

args: 2(integer, integer)

두 정수에 대해 나눗셈을 실행하여 몫을 반환한다.

```ruby
7 3/
>> 2
```

### 4-10-2. split

args: 2(array, array | string, string)

우리가 아는 그 split 맞다.

```ruby
[1 2 3 4 2 3 5][2 3]/
>> [[1] [4] [5]]

'a s d f'' '/
>> ["a" "s" "d" "f"]
```

### 4-10-3. split in groups of size

args: 2(array, integer)

특정한 크기의 array으로 분할한다.

```ruby
[1 2 3 4 5] 2/
>> [[1 2] [3 4] [5]]
```

### 4-10-4. unfold

args: 2(block, block)

whiler과 유사하게 동작한다. 첫번째 block은 condition block이고, 두번째 block은 execution block이다. condition block을 실행하기 전에 스택의 top 요소가 복제된다. condition block을 실행한 결괏값(해당 값은 스택에 다시 push되지 않음)이 참이면 그 때의 스택 top 값이 array에 append되고 execution block이 실행된다. 거짓이면 정지한다.

```ruby
0 1 {100<} { .@+ } /
>> 89 [1 1 2 3 5 8 13 21 34 55 89]
```

### 4-10-5. each

args: 2(array, block)

주어진 array 각각의 원소에 block을 실행한 결괏값을 차례대로 스택에 저장한다.

```ruby
[1 2 3]{1+}/
>> 2 3 4
```

## 4-11. `%`

### 4-11-1. mod

args: 2(integer, integer)

나머지 연산을 실행한다.

```ruby
7 3 %
>> 1
```

### 4-11-2. clean split

args: 2(string, integer)

`/`의 split과 똑같이 연산 후, 빈 문자열들을 제거하고 반환한다.

```ruby
'assdfs' 's'% 
>> ["a" "df"]

'assdfs' 's'/
>> ["a" "" "df" ""]
```

### 4-11-3. every ith element

args: 2(array, integer)

Python의 [::n] indexing과 완전히 똑같이 작동한다.

```ruby
[1 2 3 4 5] 2%
>> [1 3 5]

[1 2 3 4 5] -1%
>> [5 4 3 2 1]
```

### 4-11-4. map

args: 2(array, block)

주어진 array 각각의 원소를 그 원소에 block을 실행한 결괏값으로 덮어쓴다.

```ruby
[1 2 3]{.}%
>> [1 1 2 2 3 3]
```

## 4-12. `?`

### 4-12-1. power

args: 2(integer, integer)

주어진 두 정수에 대해 거듭제곱 연산을 수행한다.

```ruby
2 8?
>> 256
```

### 4-12-2. find with equality

args: 2(any, array/string)

두번째 argument로 주어진 array나 string에서 첫번째 argument로 주어진 값이 있는 인덱스를 반환한다. 존재하지 않으면 -1 반환.

```ruby
5 [4 3 5 1] ?
>> 2
```

### 4-12-3. find with condition

args: 2(array/string, block)

주어진 array나 string에서 block의 식을 처음으로 만족하는 원소의 인덱스를 반환한다. 만족하는 원소가 존재하지 않으면 -1 반환.

```ruby
[1 2 3 4 5 6] {.* 20>} ?
>> 5
```

## 4-13. `~`

### 4-13-1. bitwise not

args: 1(integer)

```ruby
5~
>> -6
```

### 4-13-2. eval

args: 1(string/block)

주어진 string이나 block을 실행한다.

```ruby
"1 2+"~
>> 3

{1 2+}~
>> 3
```

### 4-13-3. dump

args: 1(array/string)

주어진 array나 string의 원소를 차례대로 스택에 push한다.

```ruby
[1 2 3]~
>> 1 2 3

"1 2 3"~
>> 1 2 3
```

## 4-14. backtick(`)

### 4-14-1. inspect(uneval)

args: 1

피연산자의 소스코드를 나타내는 string을 반환한다.

```ruby
1`
>> "1"

[1 [2] 'asdf']`
>> "[1 [2] \"asdf\"]"

"1"`
>> "\"1\""

{1}`
>> "{1}"

```

## 4-15. `,`

### 4-15-1.

# 5. 주석

주석은 `#`를 이용하여 작성한다. `#`뒤부터 개행 전까지 작성되어있는 모든 것은 컴파일러가 무시한다.
