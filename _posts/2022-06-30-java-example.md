---
title:  "[java] 유용한 샘플 코드 정리"
excerpt: "Java 유용한 샘플 코드 정리"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2022-06-30
last_modified_at: 2022-06-30
---

### time 관련

```java
//07:05:45PM -> 19:05:45
String s = "07:05:45PM";
SimpleDateFormat formatter = new SimpleDateFormat("hh:mm:ssa");
Date date =(Date)formatter.parse(s);
//
SimpleDateFormat formatterOutput = new SimpleDateFormat("HH:mm:ss");
String output = formatterOutput.format(date);
```

### Java Varargs 

* 가변인자 관련 
```java
//case1
class Add{
    public void add(Integer ...integers){
        Integer[] objects = integers;
        for(int i=0; i<objects.length; i++){
            objects[i];
        }
    }
    
    public static void display(String ...strs) {        
        for(String s: strs) {
            System.out.println(s);        
        }
    }
}
//case2

```


### sort 관련

* 배열(Array) 정렬 하기 ( 오름차순,내림차순 등 )

```java
class CustomComparator implements Comparator<String> {
    @Override
    public int compare(String o1, String o2) {
       return o2.compareTo(o1); //내림차순
    }
}

String[] stringArr = new String[] {"A","C","B","E","D"};   
Arrays.sort(stringArr,new CustomComparator());   
//
List<Student> studentList = new ArrayList<Student>();
Collections.sort(studentList,new CustomComparator() );

Collections.sort(list, Collections.reverseOrder());
```

* 샘플

```java
class Student implements Comparable<Student>
{
    int ID = 0;
    String Name = "";
    double CGPA = 0;

    public Student(int id, String name, double cgpa)
    {
        this.ID = id;
        this.Name = name;
        this.CGPA = cgpa;
    }

    public int getID() { return ID; }
    public String getName() { return Name; }
    public double getCGPA() { return CGPA; }

    @Override
    public int compareTo(Student o) {

        // System.out.println("compare: " + this.Name + " " + o.Name);

        if (this.CGPA != o.CGPA) return this.CGPA - o.CGPA > 0 ? -1 : 1;
        else if(this.Name.charAt(0) != o.Name.charAt(0)) return  (this.Name.charAt(0) - o.Name.charAt(0));
        else return (this.ID - o.ID);
    }
}
```

### Generics 관련

* Generics 

```java
public <T> List<T> fromArrayToList(T[] a) {   
    return Arrays.stream(a).collect(Collectors.toList());
}
//The <T> in the method signature implies that the method will be dealing with generic type T. This is needed even if the method is returning void.

//As mentioned, the method can deal with more than one generic type. Where this is the case, we must add all generic types to the method signature.



//Here is how we would modify the above method to deal with type T and type G:
public static <T, G> List<G> fromArrayToList(T[] a, Function<T, G> mapperFunction) {
    return Arrays.stream(a)
      .map(mapperFunction)
      .collect(Collectors.toList());
}
```

### 패턴 관련

* 한글, 영어, 숫자 및 특수문자 몇개(._:-) 로만 이루어졌는지 체크해보자

```java
    //한글, 영어, 숫자 및 특수문자 몇개(._:-) 로만 이루어졌는지 체크해보자
    
    private final static Pattern NAME_PATTERN = Pattern.compile("^[ㄱ-ㅎㅏ-ㅣ가-힣a-z0-9A-Z._:-]*$");
    //
    Matcher matcher = NAME_PATTERN.matcher(inputString);
    if(!matcher.find()){
        //이상한 문자가 있음
    }
```

* 입력 문자열 구조 

```java

//    // 문자열중에 "/v + 숫자" 로 시작하는 부분을 찾기

    private static Pattern URL_VERSION_PATTERN = Pattern.compile("^*[/v|/V][0-9]{1}");
    
    
    /**
     * 호출한 url정보중 version 정보를 추출하여 숫자 형태로 변환후 리턴한다.
     *  @param url http://aaa/bbb/ccc/v1.3 or http://aaa/bbb/ccc/v1.3.1/...
     *  @return 1_003_000 / 1_003_001
     */
    public static long extractVersionNumber(String url){

        Matcher matcher = URL_VERSION_PATTERN.matcher(url);

        if(matcher.find()){
            int start = matcher.start();
            int end = url.indexOf("/", matcher.start());
            //ex) url이 http://xxx/v.1.3 처럼 버전정보뒤에 /이 없는 경우
            if(end<=start){
                end = url.length();
            }
            String version = url.substring(matcher.start(), end);
            return Utils.convertVersionToNumberCode(version);
        }else{
            return -1;
        }
    }
```

 * IP 패턴

```java
  // 25[0-5]        = 250-255
  // (2[0-4])[0-9]  = 200-249
  // (1[0-9])[0-9]  = 100-199
  // ([1-9])[0-9]   = 10-99
  // [0-9]          = 0-9
  // (\.(?!$))      = can't end with a dot
  private static final String IPV4_PATTERN_SHORTEST =
          "^((25[0-5]|(2[0-4]|1[0-9]|[1-9]|)[0-9])(\\.(?!$)|$)){4}$";
```

* 기타

```java
 자주 사용하는 정규 표현식 
정규 표현식	설명
^[0-9]*$	숫자
^[a-zA-Z]*$	영문자
^[가-힣]*$	한글
\\w+@\\w+\\.\\w+(\\.\\w+)?	E-Mail
^\d{2,3}-\d{3,4}-\d{4}$	전화번호
^01(?:0|1|[6-9])-(?:\d{3}|\d{4})-\d{4}$	휴대전화번호
\d{6} \- [1-4]\d{6}	주민등록번호
^\d{3}-\d{2}$	우편번호



1. 매칭될 문자를 지정하거나, 제외하는 방법입니다.
[abc]           a, b, c중 하나이면 일치 합니다.
[^abc]          a, b, c를 제외한 다른 글자 이면 일치합니다.
[a-zA-Z]        a 부터 z까지의 소문자 알파벳 이거나 A 부터 Z까지의 대문자 알파벳 중의 하나라면 일치합니다.(범위)
[a-d[m-p]]      a 부터 d까지, 또는 m 부터 p까지 중에 하나와 일치합니다: [a-dm-p] (합집합)
[a-z&&[def]]    d, e, f 중의 하나와 일치합니다. (교집합)
[a-z&&[^bc]]    b와 c를 제외한 a 부터 z까지 중의 하나와 일치합니다: [ad-z] (차집합)
[a-z&&[^m-p]]   m부터 p 까지를 제외한, a 부터 z까지 중의 하나와 일치합니다: [a-lq-z] (차집합)

2. 미리 정의된 문자를 지정하는 방법입니다.
.       임의의 문자 (라인 종결자와 일치할 수도 하지 않을 수도 있음)
\d      숫자 문자: [0-9]
\D      숫자 문자가 아닌것: [^0-9]
\s      화이트 스페이스 문자: [ \t\n\x0B\f\r]
\S      화이트 스페이스 문자가 아닌것: [^\s]
\w      알파벳 단어 문자(word 문자): [a-zA-Z_0-9]
\W      알파벳 단어 문자가 아닌것: [^\w]

6. 경계조건 판별 조건행의 시작 끝, 단어 경계 등을 판별합니다.
^       행의 시작
$       행의 끝
\b      단어 경계
\B      단어가 아닌것의 경계
\A      입력의 시작 부분
\G      이전 매치의 끝
\Z      입력의 끝이지만 종결자가 있는 경우
\z      입력의 끝

*       앞 문자가 없을 수도 무한정 많을 수도 있음
+       앞 문자가 하나 이상
?       앞 문자가 없을 수도 있고 하나 있음
\       역슬래시 다음에 문자가 오면 특수문자 취급
[ ]     문자의 집합이나 범위 [] 안에서 ^기호로 시작되면 not 을 나타냄 ! 기호라고 생각하자!
{ }     횟수나 범위
( )     그룹지정
|       or조건


//  다음 \b은 단어 경계이며 \1첫 번째 그룹의 캡처 된 일치를 참조합니다.
// 대소문자 무시하는 옵션.

//\ b : 단어 경계의 시작
//\ w + : 모든 단어 문자
//(\ s + \ 1 \ b) * : 이전 단어와 일치하고 단어 경계로 끝나는 단어가 뒤에 오는 임의의 수의 공백. *로 묶인 모든 것은 하나 이상의 반복을 찾는 데 도움이됩니다.

//그룹화 :
//m.group (0) : 위의 경우 일치하는 그룹을 포함합니다 Goodbye goodbye GooDbYe
//m.group (1) : 위의 경우 일치하는 패턴의 첫 단어를 포함합니다 Goodbye

String regex = "\\b(\\w+)(?:\\W+\\1\\b)+";
//String regex = "\\b(\\w+)(\\s+\\1\\b)*";
Pattern p = Pattern.compile(regex, Pattern.CASE_INSENSITIVE);

Matcher m = p.matcher(input);

// Check for subsequences of input that match the compiled pattern
while (m.find()) {
     input = input.replaceAll(m.group(0), m.group(1));
}

https://wormwlrm.github.io/2020/07/19/Regular-Expressions-Tutorial.html
?:      그룹화가 필요하지만 캡처화는 필요하지 않는 경우.괄호안에 ?: 로 시작. 


* 이것 설명. 
Pattern r = Pattern.compile("<(.+)>([^<]+)</\\1>");

<(.+)>
matches HTML start tags. The parentheses save the contents inside the brackets into Group

#1
([^<]+)
matches all the text in between the HTML start and end tags. We place a special restriction on the text in that it can't have the "<" symbol. The characters inside the parenthesis are saved into Group #2.

</\\1>
is to match the HTML end brace that corresponds to our previous start brace. The \1 is here to match all text from Group #1.
```

### thread pool 관련.

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
//...
for(){
    executor.execute(() -> {
        //do someting
    });
}

try {
    //Executor shutdown and awaitTermination!
    executor.shutdown();
    boolean terminated = executor.awaitTermination(30, TimeUnit.DAYS);
    //log.info("terminated: {}", terminated);
} catch (InterruptedException e) {
    log.error("", e);
}
```

### stream 관련

* 특정 컬럼만 쉽게 추출하기.

```java
@Data
public class Entity {
    protected String id;
    protected String appKey;
    protected String channelType;
}
//..
List<Entity> entityList = something;
List<String> ids = entityList.stream().map(Entity::getId).collect(Collectors.toList());
```

* 중복 제거하기

```java
@Data
public class RequestCreateChannel {
    protected String channelName;
    protected List<Long> channelTagIds;
}
//..
RequestCreateChannel channel = someting;
List<Long> tagIds = new ArrayList<>(channel.getChannelTagIds()).stream().distinct().collect(Collectors.toList());
```

* 데이터 형식 바꾸기

```java
@Data
public class SimpleMember {
    protected String appId;
    protected String userId;
    protected ValidCode valid;
}

@NoArgsConstructor
public class SimpleMemberEntity extends SimpleMember {
    public SimpleMemberEntity(String appId) {
    //...
    }
    //...
}


List<SimpleMemberEntity> memberList = something;

// 전환하여 맞는 객체로 새롭게 생성하여 전달
List<SimpleMember> newMemberList = memberList.stream().map(SimpleMember::new).collect(Collectors.toList());
```

### 변환

```java
//Strinig to int
String from = "123";
int to = Integer.parseInt(from);

//int to String
int from = 123;
String to = Integer.toString(from);

// double 형 숫자를 String으로 변환시키기.
String tmp = Double.toString(1.24657);

 

// String형 숫자를 double형으로 변환시키기.
double tmp = Double.parseDouble("1.24657");

```


### 디자인 패턴
* factory pattern

```java
interface Food {
	public String getType();
}

class Pizza implements Food {
	public String getType() {
		return "Someone ordered a Fast Food!";
	}
}

class Cake implements Food {
	public String getType() {
		return "Someone ordered a Dessert!";
	}
}

class FoodFactory {
	public Food getFood(String order) {
		if(order.compareTo("cake")==0){
			return new Cake();
		}else if(order.compareTo("pizza")==0){
			return new Pizza();
		}
		return null;
	}//End of getFood method
}//End of factory class

public class Solution {
	public static void main(String args[]){

		//creating the factory
		FoodFactory foodFactory = new FoodFactory();

		//factory instantiates an object
		Food food = foodFactory.getFood(sc.nextLine());

		System.out.println("The factory returned "+food.getClass());
		System.out.println(food.getType());
	}

}

```


### 기타

```java
List<SomeEntity> result = new ArrayList<>();
result.forEach(some -> {});

Map<ChannelId, NettyChannelEntry> channelEntryMap = new ConcurrentHashMap<>();
Scanner in = new Scanner(System.in);

int[] num = new int[3]


String str = "abcde";

// reverse
StringBuffer sb = new StringBuffer(str);
String reversedStr = sb.reverse().toString();

//reverse
char[] arr = str.toCharArray(); // String -> char[]
char[] reversedArr = new char[arr.length];
for(int i=0; i<arr.length; i++){
    reversedArr[arr.length-1-i] = arr[i];
}
String reversedStr = new String(reversedArr);

//reverse
char[] arr = str.toCharArray(); // String -> char[]
List<Character> list = new ArrayList<>();
for(char each : arr){ // char[] -> List
list.add(each);
}

// reverse
Collections.reverse(list);

ListIterator li = list.listIterator();
while(li.hasNext()){
System.out.print(li.next()); // edcba
}



//문자열 분리
System.out.println("정규식으로 1칸이상의 공백 자르기");
String[] arStrRegexMultiSpace = str.split("\\s+");


//여러 구분자를 사용하려면 구분자들 사이에 |(파이프라인)을 넣어주시면 됩니다.
String []tokens=str.split(" |,");
String []tokens = s.split(" |!|,|'|\\?|@|`|\\.|, ");
//English alphabetic letters, blank spaces, exclamation points (!), commas (,), question marks (?), periods (.), underscores (_), apostrophes ('), and at symbols (@).
String[] strings = s.split("['!?,._@ ]+"); //이렇게 쓰자. 

for(int i=0;i<tokens.length;i++){
    System.out.println(tokens[i]);
}

//1. 띄어쓰기 기준으로 문자열을 분리
StringTokenizer st = new StringTokenizer(문자열); 
//2. 구분자를 기준으로 문자열을 분리
StringTokenizer st = new StringTokenizer(문자열, 구분자); 
/* 3. 구분자를 기준으로 문자열을 분리할 때 구분자도 토큰으로 넣는다. (true) * 구분자를 분리된 문자열 토큰에 포함 시키지 않는다. (false) * (디폴트 : false) */ 
StringTokenizer st = new StringTokenizer(문자열 , 구분자 , true/false);

String str = "안녕하세요 슬기로운개발생활 tistory 입니다.";        
StringTokenizer st = new StringTokenizer(str);                
    while (st.hasMoreTokens()) {            
        System.out.println(st.nextToken());        
    }
//


str.replaceAll("\\s+", " ");
정규식 "\s"는 다음과 같은 종류의 공백(white space)을 나타냅니다.
(\t, \n, \x0B, \f, \r)
그리고 "+"는 1번이상을 의미합니다.
즉, "\s+"는 1번 이상의 공백을 의미합니다.



HashMap<String, String> map = new HashMap<String, String>();
- LinkedHashMap은 입력된 순서대로 데이터가 출력되는 특징을 가지고 있다.
= TreeMap은 입력된 key의 소트순으로 데이터가 출력되는 특징을 가지고 있다.


Stack<Integer> stack = new Stack<>();
HashSet<Integer> set = new HashSet<Integer>();

List<List<Integer>> arr = new ArrayList<>();
arr.get(a).get(b)


class Animal{}
class Bird extends Animal{
}

//
interface AdvancedArithmetic{
  int divisor_sum(int n);
}
class MyCalculator implements AdvancedArithmetic{
    
}



//sample
public class Solution {

	public static void main(String[] args) throws Exception {

			int num = 8
			Object o;
            
            Inner inner = new Inner();
            o = inner.new Private();
            Inner.Private innerPrivate = (Inner.Private) o;
            String response = innerPrivate.powerof2(num);

	}//end of main
	static class Inner{
		private class Private{
			private String powerof2(int num){
				return ((num&num-1)==0)?"power of 2":"not a power of 2";
			}
		}
	}//end of Inner
	
}//end of Solution
```


### java reflection

```java
try {
    Class student = Class.forName("Student");

    Method[] methods = student.getDeclaredMethods();
    ArrayList<String> methodList = new ArrayList<>();
    for (int i = 0; i < methods.length; i++) {
        Method m = methods[i];
        methodList.add(m.getName());
    }
    Collections.sort(methodList);
    for(String name: methodList){
        System.out.println(name);
    }
}
catch(Exception e){

}
```


### java annotations


```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface FamilyBudget {
	String userRole() default "GUEST";
    int roleLimit() default 200;
}

class FamilyMember {
	@FamilyBudget(userRole = "SENIOR", roleLimit=100)
	public void seniorMember(int budget, int moneySpend) {
		System.out.println("Senior Member");
		System.out.println("Spend: " + moneySpend);
		System.out.println("Budget Left: " + (budget - moneySpend));
	}

	@FamilyBudget(userRole = "JUNIOR", roleLimit=50)
	public void juniorUser(int budget, int moneySpend) {
		System.out.println("Junior Member");
		System.out.println("Spend: " + moneySpend);
		System.out.println("Budget Left: " + (budget - moneySpend));
	}
}
```

### java lamda

```java
import java.io.*;
import java.util.*;
interface PerformOperation {
 boolean check(int a);
}
class MyMath {
 public static boolean checker(PerformOperation p, int num) {
  return p.check(num);
 }

   // Write your code here
   PerformOperation isOdd()
   {
       PerformOperation po = (int a)-> a%2 == 0 ? false : true;
       return po;
   }
   PerformOperation isPrime()
   {
       PerformOperation po = (int a)->  
       {
           if(a == 1) return true;
           else
           {
               for (int i =  2; i < Math.sqrt(a); i++)
                    if(a%i == 0) return false;
                return true;
           }
       };
       return po;
   }
   PerformOperation isPalindrome()
   {
       ArrayList<Integer> aa = new ArrayList<>();
       PerformOperation po = (int a)->
       {
            String str = Integer.toString(a);
           String reverse = "";
           for(int i = str.length()-1; i >= 0; i--)
           {
               reverse = reverse + str.charAt(i);
           }
           if(reverse.equals(str)) return true;
           return false;
       };
       return po;
   }
   
}

public class Solution {

 public static void main(String[] args) throws IOException {
  MyMath ob = new MyMath();
  BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
  int T = Integer.parseInt(br.readLine());
  PerformOperation op;
  boolean ret = false;
  String ans = null;
  while (T--> 0) {
   String s = br.readLine().trim();
   StringTokenizer st = new StringTokenizer(s);
   int ch = Integer.parseInt(st.nextToken());
   int num = Integer.parseInt(st.nextToken());
   if (ch == 1) {
    op = ob.isOdd();
    ret = ob.checker(op, num);
    ans = (ret) ? "ODD" : "EVEN";
   } else if (ch == 2) {
    op = ob.isPrime();
    ret = ob.checker(op, num);
    ans = (ret) ? "PRIME" : "COMPOSITE";
   } else if (ch == 3) {
    op = ob.isPalindrome();
    ret = ob.checker(op, num);
    ans = (ret) ? "PALINDROME" : "NOT PALINDROME";

   }
   System.out.println(ans);
  }
 }
}
```

* 정리. 
```java
    public static void main(String[] args) throws IOException {
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(System.getenv("OUTPUT_PATH")));

        String[] firstMultipleInput = bufferedReader.readLine().replaceAll("\\s+$", "").split(" ");

        int month = Integer.parseInt(firstMultipleInput[0]);

        int day = Integer.parseInt(firstMultipleInput[1]);

        int year = Integer.parseInt(firstMultipleInput[2]);

        String res = Result.findDay(month, day, year);

        bufferedWriter.write(res);
        bufferedWriter.newLine();

        bufferedReader.close();
        bufferedWriter.close();
    }
```


* 배열 초기화
```java
int [][][] magicSqare = {
	{
    	{1,2,3,},
    	{1,2,3,},
    	{1,2,3,}
    },
	{
    	{1,2,3,},
    	{1,2,3,},
    	{1,2,3,}
    }
};
```


* n개중 r개 뽑기

```java
public class Combination {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3}; //조합을 만들 배열 
        boolean[] visited = new boolean[arr.length];
 
        //1. 백트래킹을 이용해 구현
        System.out.println("-------- 1. 백트래킹 ---------");
    
        for(int r = 1; r <= arr.length; r++) {
        	System.out.println("\n" + arr.length + "개 중에 " + r  + "개 뽑음");
            comb1(arr, visited, 0, r);
        }
        
        //2. 재귀를 이용해 구현
        System.out.println("\n---------- 2. 재귀 ----------");
        
        for(int r = 1; r <= arr.length ; r++) {
        	System.out.println("\n" + arr.length + "개 중에 " + r  + "개 뽑음");
            comb2(arr, visited, 0, r);
        }
    }
 
    //1. 백트래킹을 이용해 구현
    static void comb1(int[] arr, boolean[] visited, int start, int r) {
        if(r == 0) {
            print(arr, visited);
            return;
        } else {
            for(int i = start; i < arr.length; i++) {
                visited[i] = true;
                comb1(arr, visited, i + 1, r - 1);
                visited[i] = false;
            }
        }
    }
 
    //2. 재귀를 이용해 구현
    static void comb2(int[] arr, boolean[] visited, int depth, int r) {
        if(r == 0) {
            print(arr, visited);
            return;
        }
        if(depth == arr.length) {
            return;
        } else {
            visited[depth] = true;
            comb2(arr, visited, depth + 1, r - 1);
 
            visited[depth] = false;
            comb2(arr, visited, depth + 1, r);
        }
    }
 
    // 배열 출력
    static void print(int[] arr, boolean[] visited) {
        for(int i = 0; i < arr.length; i++) {
            if(visited[i] == true)
                System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

}
```

* n개중 r개 뽑기
* 아래가 더 좋나?
```java

public class Main {
	public static void main(String[] args) throws Exception {
		
		int[] list = {0,1,2};
		int n = list.length;
		int r = 2;
		HashSet<Integer> set = new HashSet<>();
		combination(list, n, r, 0, set);
	}

	private static void combination(int[] list, int n, int r, int index, HashSet<Integer> set) {
		if(r==0) {
			for(int i : set) {
				System.out.print(i+" ");
			}System.out.println();
			return;
		}
		
		for(int i=index; i<n; i++) {
			set.add(i);
			combination(list, n, r-1, i+1, set);
			set.remove(i);
		}
		
	}
}
```
