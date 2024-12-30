
## 기본

```objc
#import <Foundation/Foundation.h>

@interface Vehicle : NSObject {

  // Declare Member variable
  int wheels;
  int seats;
}

  
- (int) wheels;
- (int) seats;
- (void) setWheels:(int) w;

- (void) setSeats:(int) s;

- (void) print;


@end

@implementation Vehicle


- (int)wheels {
	return wheels
}

- (int)seats {
	return seats
}
  
- (void)setWheels:(int)w {
  wheels = w;
}

- (void)setSeats:(int)s {
  seats = s;
}


- (void)print {
  NSLog(@"Vehicle have %i wheels and %i seats.", wheels, seats);
}

@end
  

// 대괄호는 메세지를 구분해주는 기호
// [Receiver message] 형식

  

int main(int argc, const char * argv[]) {

  @autoreleasepool {
    // vehicle을 alloc 하고 init 하라
    Vehicle* vehicle = [[Vehicle alloc] init];
    [vehicle setWheels: 4];
    [vehicle setSeats: 2];

    [vehicle print];
    NSLog(@"wheels: %i, seats: %i", [vehicle wheels], [vehicle seats]);
  

  }
  return 0;

}
```


Objective-C 는 선언과 구현이 분리되어 있음

@interface ~ @end는 선언부임
@implementation ~@end는 구현부임

[ ] 대괄호는 message를 구분하는 기호로 주로 사용됨. [Receiver Message] 형식임. 그리고 멤버 메서드는 message의 일종임. getter setter를 이용할 때도 초기 버전에는 리시버 메세지 형식을 그대로 따라야 했으나, 지금은 Dot 접근도 가능함 


getter 구현시에 관행적으로 get을 붙이지 않는다. getWheels(X) -> wheels(O)
getter, setter를 매번 구현하는 것은 귀찮음. getter setter를 지우고 선언부 member method 영역에 @property로 선언하는 것으로 대체할 수 있음. 그리고 @property로 사용하면 멤버 변수 선언을 생략해도 됨

만약, @property 선언만 하고  멤버변수를 선언하지 않았을 때 객체 내부에서 해당 변수를 사용하기 위해서는 _ 을 붙여줘야함. @property를 선언한 변수는 내부적으로  \_\name 으로 선언되기 때문

내부에서도 같은 이름을 사용하고 싶으면 @synthesize를 사용해야함.



```objc

@interface Vehicle : NSObject {  

  // 생략 가능!
  
  // int wheels;
  // int seats;

}

@property int wheels;
@property int seats;

@end

@implementation Vehicle
@synthesize seats;

- (void)print {

  NSLog(@"Vehicle have %i wheels and %i seats.", _wheels, seats);
}

@end


int main(int argc, const char * argv[]) {

  @autoreleasepool {
    // vehicle을 alloc 하고 init 하라
    Vehicle* vehicle = [[Vehicle alloc] init];
    [vehicle setWheels: 4];
    [vehicle setSeats: 2];
    
    vehicle.wheels = 4;
    vehicle.seats = 2;

   [vehicle print];
    NSLog(@"wheels: %i, seats: %i", [vehicle wheels], [vehicle seats]);
    NSLog(@"wheels: %i, seats: %i", vehicle.wheels, vehicle.seats);

  }
  return 0;

}


```

##  함수 Arguement 쓰는 법

다른 언어와 다르게 함수를 선언할 때 Arguement를 나누는 기준이 콜론(:)이다.

```objc
// 2개의 인자를 가진 함수
- (void) setWheels:(int)w Seats:(int)s;

// 호출할 때

[vehicle setWheels:4 Seats:2];


```

## 조건문, 반복문

if, else if, else, switch문 대부분의 언어와 똑같음.
for, while문도 마찬가지


##  스트링

NSString과 NSMutableString이 있음. 둘 다 객체 타입

NSString과 NSMutableString의 차이점은 자기 자신을 바꿀 수 있냐임.
NSString은 append, insert 같은 함수가 없음. 자신은 변화 불가능이니까

String 초기화시 alloc과 init을 이용하고 @""를 이용해 값을 넣을 수 있음
보통 바로 NSString* varname = @""; 로 바로 초기화함.



```objc

int main(int argc, const char * argv[]) {

  @autoreleasepool {


    NSString* str = [[NSString alloc]init];
    str = @"This is NSString";
    
    NSString* str2 = [[NSString alloc]initWithString:@"This is NSString"];
    
    NSLog(@"str: %@", str);
    NSLog(@"str: %@", str2);

    // immutable class - 자기 자신은 변화를 못하기 때문에 새로 할당해야함.

    NSString* result;

    // substringFromIndex
    result = [str substringFromIndex:6];
    NSLog(@"result: %@", result);


    // substringToIndex
    result = [str substringToIndex:6];
    NSLog(@"result: %@", result);


   // method chaining
    result = [[str substringToIndex:11 ] substringFromIndex:8];
    NSLog(@"result: %@", result);


    //substringWithRange
    result = [[str substringWithRange:NSMakeRange(8, 3)] lowercaseString];
    NSLog(@"result: %@", result);

    result = [[str substringWithRange:NSMakeRange(8, 3)] uppercaseString];
    NSLog(@"result: %@", result);


    // NSMutableString - 자기 자신 수정 가능, 할당해서 사용해야함 string처럼 =@""로 불가

    NSMutableString* mstr = [NSMutableString stringWithString:str];
    NSLog(@"mstr: %@", mstr);

    [mstr appendString:@ " and NSMutableString"];
    NSLog(@"mstr: %@", mstr);

    [mstr insertString:@"Mutable" atIndex:8];
    NSLog(@"mstr: %@", mstr);
  

  }

  return 0;

}
```


NSMutableString은 무조건 할당해서 사용해야한다. NSString처럼 =@""로 사용하면 아래처럼 경고를 주고 NSMutableString 메서드를 사용하게 되면 런타임에서 에러가 발생한다.
![[immutable_mutable.png]]

