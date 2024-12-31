Swift에서  map(), filter() 같은 고차함수를 많이 사용했었어요. C#으로 처음 PS를 풀 때 map을 사용하고 싶어서 c# map function으로 구글링 했을 때  LINQ의 Select() 메서드를 사용하라고 하더군요. 'Select이 뭐지?' 했는데 데이터베이스 시간에 SQL로 열심히 실습할 때 나오는 그 select가 동일합니다. 그럼 LINQ가 뭘까요?

#### LINQ?

LINQ는 Language-Integrated Query의 준말로 C#에서 직접 쿼리 기능을 통합하는 방식을 기반으로 하는 기술 이름이에요. 일반적으로 쿼리는 특수화된 쿼리 언어로 표현되어요. 예를 들면 관계형 데이터베이스에서는 SQL, XML에서는 XQuery가 사용됩니다. 그래서 하나의 형식을 사용할 때마다 하나의 쿼리 언어를 학습할 필요가 생기는 거죠. 그래서 LINQ가 등장하게 되었어요. LINQ는 다양한 데이터 소스 및 다양한 형식에 사용할 수 있는 일관된 모델을 제공합니다. 모든 형식에서 같은 방식으로 데이터를 쿼리하고 변환할 수 있게 됩니다. 

LINQ는 쿼리군요.  그럼 쿼리는 무엇일까요? 

> 쿼리는 데이터 소스에서 검색할 데이터 및 반환된 데이터에 필요한 모양과 구성을 설명하는 지침 집합이다.

쉽게 풀면 쿼리는 데이터 소스에서 데이터를 검색(추출)하는 식입니다. (쿼리를 통해 검색된 결과가 아님을 주의)

C#에서 쿼리(LINQ)를 어떻게 이용할 수 있을까요?  LINQ는 세가지 작업으로 구성됩니다!

1.  데이터 소스 가져오기

2. 쿼리 만들기

3. 쿼리 실행

### 데이터 소스 가져오기

데이터 소스는 말 그대로 우리가 쿼리하고 싶은 데이터 모음입니다. 단 쿼리 변수의 타입이 IEumerable 또는 IQueryable이므로 이것을 두 타입 중 하나를 컨펌해야만 합니다. List와 Array와 같은 컬렉션 타입은 IEnumerable을 컨펌하고 있기 때문에 데이터 소스로 사용할 수 있어요. 또 관계형 데이터 베이스나, XML과 같은 것들을 IEumerable 또는 IQueryable로 로드를 하거나 맵핑을 해서 데이터 소스로 사용할 수 있습니다.

```
// Array

int[] numbers = new int[7] { 0,1,2,3,4,5,6 };

IEumerable<int> numQuery = 
	from number in numbers 
	where (number % 2) == 0 
	select number;
```

```
// LINQ to XML
// Create a data source from an XML document.
// using System.Xml.Linq;
XElement contacts = XElement.Load(@"c:\myContactList.xml");

IQueryable<Contract> contractQuery =
	from contract in contracts
    	where ...
    	select ...
```

```
// LINQ to SQL Mapping

Northwnd db = new Northwnd(@"c:\northwnd.mdf");

// Query for customers in London.
IQueryable<Customer> custQuery =
    from cust in db.Customers
    where cust.City == "London"
    select cust;
```

### 쿼리 만들기

이제 쿼리 변수를 만들겠습니다.

쿼리 변수를 만드는 방법은 두가지가 있습니다.

1. 쿼리 구문을 이용해서!

2. 메서드 구문을 이용해서!

그럼 1번 2번을 두개 다 연습을 해볼게요. 연습을 하기 전에 데이터 소스를 먼저 만들어줍시다.

```
// Teacher.cs

public class Teacher
{
    public string Name { get; set; }
    public int ID { get; set; }
    public decimal Salary { get; set; }

    public Teacher(string name, int id, decimal salary)
    {
        Name = name;
        ID = id;
        Salary = salary;
    }
}

// Student.cs


public class Student
{
    public string Name { get; set; }
    public int ID { get; set; }
    public List<int> scroes { get; set; }

    public Student(string name, int id, List<int> scroes)
    {
        Name = name;
        ID = id;
        this.scroes = scroes;
    }
}


// Program.cs

namespace TestLINQ
{
    class Program
    {
        static void Main(string[] args)
        {
            List<Student> students = new List<Student>();
            List<Teacher> teachers = new List<Teacher>();
            
            SeedData(students, teachers);





        }

        public static void SeedData(List<Student> students, List<Teacher> teachers)
        {
            List<Student> _students = new List<Student>()
            {
                new Student("김수로", 0, new() { 90, 95,70,50, 87}),
                new Student("박혁거세", 1, new() { 80, 45,95,80, 75}),
                new Student("이은혜", 2, new() { 83, 63,89,93, 63}),
                new Student("이근왕", 3, new() { 55, 77,77,31, 90}),
                new Student("선우현", 4, new() { 100, 15,25,36, 57}),
            };

            foreach (var s in _students)
            {
                students.Add(s);
            }
            
            List<Teacher> _teachers = new List<Teacher>()
            {
                new Teacher("김철수", 5, 10000),
                new Teacher("이진혜", 6, 20000),
                new Teacher("김왕심", 7, 30000),
                new Teacher("박수빈", 8, 40000),
                new Teacher("손을왕", 9, 50000),
            };
            
            foreach (var t in _teachers)
            {
                teachers.Add(t);
            }
        }
    }
}
```

#### 1. 쿼리 구문을 이용

혹시 SQL을 사용한 경험이 있으신가요?

SQL처럼 select where from을 이용해서 쿼리 하는 방식이 쿼리 구문을 이용하는 방법입니다. SQL과 차이점이라면 순서가 반대라는 것!

쿼리 구문은 아래와 같은 형식을 가집니다.

- 쿼리 구문은 반드시 from절로 시작해야 함
- 쿼리 구문은 반드시 select절 또는 group절로 끝나야 함.
- 첫 번째 from절과 마지막 select절 또는 group절 사이에 where, orderby, join, let절과 추가 from절들이 하나 이상 들어갈 수 있음.

첫 번째 예시는 첫번째 시험에서 90점 이상을 맞은 학생들을 검색하는 쿼리 변수를 만들게요.

```
       static void Main(string[] args)
        {
            List<Student> students = new List<Student>();
            List<Teacher> teachers = new List<Teacher>();
            
            SeedData(students, teachers);
            

            IEnumerable<Student> notLessThan90 =
                from student in students
                where (student.scroes[0] >= 90)
                select student;

            
            Console.WriteLine("첫번째 시험에서 90점 이상 맞은 학생들");
            foreach (Student student in notLessThan90)
            {
                Console.WriteLine("{0}, {1}", student.Name, student.scroes[0]);
            }
```

쿼리 구문을 이용하면 상당히 직관적으로 데이터를 추출할 수 있네요. 기본적인 구조는 이해하는데 문제가 없을 것 같아요.

추가로 정렬을 의미하는 orderby도 한번 사용해 볼게요. Student에서 성이 이씨인 사람들을 뽑을건데 ID를 내림차순으로 정렬해보도록 하겠습니다.

```
            IEnumerable<Student> descendingByID = from student in students
                where (student.Name[0].ToString() == "이")
                orderby student.ID descending
                select student;

            Console.WriteLine("학생들중 이씨인 사람들 뽑아 ID를 내림차순으로 정렬");
            foreach (Student student in descendingByID)
            {
                Console.WriteLine("{0}, {1}", student.Name, student.ID);
            }
```

되게 간단하네요.  마지막으로 select 대신에 group을 한번 사용해볼게요.

Student 리스트에서 성으로 그룹핑을 하겠습니다.

```
            Console.WriteLine("학생들의 성으로 그룹화");

            IEnumerable<IGrouping<string, Student>> groupByFirstName=
                from student in students group student by student.Name[0].ToString();

            foreach (IGrouping<string, Student> group in groupByFirstName)
            {
                Console.WriteLine(group.Key);
                foreach (var student in group)
                {
                    Console.WriteLine("{0}, {1}", student.Name, student.ID);
                }
            }
            Console.WriteLine("학생들의 성으로 내림차순으로 정렬 및 그룹화");

            IEnumerable<IGrouping<string, Student>> groupByFirstNameDescending=
                from student in students 
                orderby student.Name[0].ToString() descending
                group student by student.Name[0].ToString();

            foreach (IGrouping<string, Student> group in groupByFirstNameDescending)
            {
                Console.WriteLine(group.Key);
                foreach (var student in group)
                {
                    Console.WriteLine("{0}, {1}", student.Name, student.ID);
                }
            }
```

보시다시피 group을 사용하면 IGrouping을 반환해서 네스트 타입이 되네요. 일일이 타입을 적어주는 게 불편한 경우도 있죠. 그런 경우에는 암묵적 형식인 var 키워드를 사용해도 됩니다.

```
            var groupByFirstNameDescending2=
                from student in students 
                orderby student.Name[0].ToString() descending
                group student by student.Name[0].ToString();

            foreach (var group in groupByFirstNameDescending2)
            {
                Console.WriteLine(group.Key);
                foreach (var student in group)
                {
                    Console.WriteLine("{0}, {1}", student.Name, student.ID);
                }
            }

        }
```

#### 2. 메서드 구문을 이용

위에서 쿼리를 쿼리 구문을 이용해서 작성해 봤어요. 

쿼리 구문은 코드를 컴파일할 때 .NET CLR(공용 언어 런타임)에 대한 메서드 호출로 변환해야 합니다. 이러한 메서드 호출은 Where, Select, GroupBy, Join, Max, Average 등과 같은 표준 쿼리 연산자를 호출한다고 하네요. 따라서 우리는 쿼리 구문 대신 메서드 구문을 사용하여 연산자를 직접 호출할 수도 있어요.

쿼리 구문과 메서드 구문은 의미상 동일하지만, 쿼리 구문이 더 간단하고 읽기 쉽다고 생각하는 사람이 많다고 합니다. 저도 쿼리 구문이 더 읽기가 쉬웠어요. 그러나 일부 쿼리는 메서드 호출로 표현해야 합니다. 예를 들어 필터를 하는 쿼리에서 요소 개수 또는 최댓값등을 얻으려면 메서드 호출을 사용해야 해요. 따라서 LINQ 쿼리를 작성하기 시작한 경우에도 쿼리 및 쿼리 식 자체에서 메서드 구문을 사용하는 방법을 잘 알고 있으면 유용합니다.

위에서 쿼리 구문으로 작성한 쿼리를 메서드 문법으로 작성해 볼게요.

```
            // 메서드 구문 이용
            
            Console.WriteLine("메서드 구문 이용해서");
            IEnumerable<Student> notLessThan90Method = students.Where(student => student.scroes[0] >= 90);
            
            Console.WriteLine("첫번째 시험에서 90점 이상 맞은 학생들");
            foreach (Student student in notLessThan90Method)
            {
                Console.WriteLine("{0}, {1}", student.Name, student.scroes[0]);
            }


            IEnumerable<Student> descendingByIDMethod =

                students.Where(student => student.Name[0].ToString() == "이").OrderByDescending(student => student.ID);
                
                // from student in students
                // where (student.Name[0].ToString() == "이")
                // orderby student.ID descending
                // select student;
            
            Console.WriteLine("학생들중 이씨인 사람들 뽑아 ID를 내림차순으로 정렬");
            foreach (Student student in descendingByIDMethod)
            {
                Console.WriteLine("{0}, {1}", student.Name, student.ID);
            }
            
            
            
            Console.WriteLine("학생들의 성으로 그룹화");

            IEnumerable<IGrouping<string, Student>> groupByFirstNameMethod =
                students.GroupBy(student => student.Name[0].ToString());
            
            
            foreach (IGrouping<string, Student> group in groupByFirstNameMethod)
            {
                Console.WriteLine(group.Key);
                foreach (var student in group)
                {
                    Console.WriteLine("{0}, {1}", student.Name, student.ID);
                }
            }
            
            Console.WriteLine("학생들의 성으로 내림차순으로 정렬 및 그룹화");

            IEnumerable<IGrouping<string, Student>> groupByFirstNameDescendingMethod =

                students.OrderByDescending(student => student.Name[0].ToString())
                    .GroupBy(student => student.Name[0].ToString());

            foreach (IGrouping<string, Student> group in groupByFirstNameDescendingMethod)
            {
                Console.WriteLine(group.Key);
                foreach (var student in group)
                {
                    Console.WriteLine("{0}, {1}", student.Name, student.ID);
                }
            }
```

마지막으로 Concat() 메서드를 이용해서 학생 리스트와 선생님 리스트를 합쳐서 홀수인 ID를 추출하겠습니다.

```
            Console.WriteLine("학생 리스트와 선생님 리스트를 합쳐서 홀수인 ID");

            var oddIDInTeacherAndStudent = students.Where(student => student.ID % 2 == 0).Select(x => x.ID)
                .Concat(teachers.Where(teacher => teacher.ID % 2 == 0).Select(x => x.ID));
            
            foreach (var id in oddIDInTeacherAndStudent)
            {
                Console.WriteLine(id);
            }
```

### 쿼리 실행

위에서 쿼리 변수를 열심히 만들어서 출력해 봤는데요. 항상 맨 밑에 foreach가 존재합니다. 쿼리 변수는 사실 결과 데이터를 가지고 있지 않아요. 그저 명령만을 가지고 있습니다.  실제로 데이터를 얻기 위해서는 쿼리 실행을 해줘야 합니다. 쿼리 실행 하는 법은 foreach를 사용해서 루프를 돌게 하면 되겠습니다.

즉시 실행을 하기 위해서는 Count(), Average(), Max() 등 결과를 얻기 위해 foreach를 암묵적으로 사용하는 메서드 문법을 사용하거나, ToList, ToArray 같은 것을 사용해서 결과를 바로 캐시 하는 방법을 사용하면 됩니다.

```
            var oddIDCount = students.Where(student => student.ID % 2 == 0).Select(x => x.ID)
                .Concat(teachers.Where(teacher => teacher.ID % 2 == 0).Select(x => x.ID)).Count();
            
            Console.WriteLine(oddIDCount);

            var oddIDList = students.Where(student => student.ID % 2 == 0).Select(x => x.ID)
                .Concat(teachers.Where(teacher => teacher.ID % 2 == 0).Select(x => x.ID)).ToList();
            
            Console.WriteLine(String.Join(" ", oddIDList));
```

#### GitHub
https://github.com/psychehose/LINQ_Practice

#### Ref
https://learn.microsoft.com/ko-kr/dotnet/csharp/linq/