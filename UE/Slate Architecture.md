
- Pros:
    - Programmers like that UI description is "close" to the code; easy to get at the data.
    - Invalidation is usually a non-issue; just poll data directly.
    - Easy to procedurally build interfaces.
- Cons:
    
    - Adding animation and styling is harder.
    - UI description is imperative code, so no chance to make it data-driven.


- Desired Slate Characteristics:
    
    - Easy access to model's code and data 모델의 코드와 데이터에 쉽게 접근할 수 있습니다.
    - Support procedural UI generation. 절차적 UI 생성을 지원합니다.
    - UI description should be hard to screw up.
    - Must support animation and styling. 애니메이션과 스타일을 지원해야 합니다.


## Core Tenets

* 불투명한 캐시와 중복된 상태를 피하세요. 역사적으로 UI는 상태를 캐시하고 명시적인 무효화를 요구 (from preferred to least preferred)
	1. Polling
	2. Transparent caches
	3. Opaque caches with low-grain invalidation


* 노티피케이션보다 알림을 더 선호 When UI structure is changing, prefer polling to notification. (When notification is necessary, prefer low-grain notifications to fine-grain notifications.)
  
* 피드백 루프를 피하기. Ex: 모든 레이아웃은 프로그래머 설정에서 계산됩니다. 이전 레이아웃 상태에 의존하지 않기.
	- Only exceptions are when UI state becomes the model; e.g. ScrollBars visualize UI state.
	- This is done for correctness and programmer sanity rather than performance.

* 일단 난잡하게 개발 -> 후에 일반화하기.


## Polling Data Flow and Delegates

*  Slate uses delegates as a flexible conduit for widgets that need to read and write the Model's data. Slate widgets read the Model's data when they need to display it.


STextBlock 은 **Text** 라는 델리게이트를 사용.

![[SLATE_ARCITECTURE_1.png]]

이 예제에서 Framerate는 float, integer로 저장될 확률이 높습니다. Delegate를 사용하면 값을 읽을 때마다 변환을 수행할 수 있는 유연성이 제공됩니다.



![[SLATE_ARCITECTURE_2.png]]SEditableText는 입력과 출력을 모두 담당하는 Slate 위젯입니다. STextBlock과 마찬가지로 데이터 시각화를 위해 Text 대리자를 사용합니다. 사용자가 편집 가능한 텍스트 필드에 일부 텍스트를 입력하고 Enter 키를 누르면 SEditableText가 OnTextChanged 대리자를 호출합니다. 프로그래머가 입력의 유효성을 검사하고 모델의 데이터를 OnTextChanged에 변경하는 데 적합한 기능을 연결했다고 가정합니다.


다음 프레임 동안 SEditableText는 모델의 데이터에서 읽습니다. 위의 예에서 항목 이름은 OnTextChanged 대리자에 의해 변경되었으며 Text 대리자를 통해 시각화를 위해 읽혀집니다.

### Attributes and Arguments

Using a delegate is not always desirable. Depending on the use case, the arguments to Slate widgets may need to be constant values or functions. We encapsulate this notion via the `TAttribute < T >` class. An attribute can be set to a constant or to a delegate.


## Performance Considerations

After reading the [Polling Data Flow and Delegates](https://docs.unrealengine.com/5.0/en-US/understanding-the-slate-ui-architecture-in-unreal-engine#pollingdataflowanddelegates) section, one might have serious concerns about performance.

Consider the following observations:

- UI complexity is bounded by the number of live widgets.
- Scrolling content is virtualized whenever possible; this mostly avoids live widgets off-screen.
    - Large numbers of off-screen widgets can easily tank Slate performance.
- Assumption: users with big screens have beefy machines to drive those screens; they can handle a large number of widgets.


## Invalidation vs Polling

Sometimes polling is either not performant or functionally incorrect. This is often the case with non-trivial values that cannot be expressed as a combination of simpler, trivial values. We usually invalidate in scenarios where the structure of the Model changes drastically. It is then reasonable to scrap an existing UI and recreate it. However, doing so assumes state loss, so we should not do it unless necessary.

Invalidation is - as a rule - reserved for infrequent, low-granularity events.

Consider the example of the Blueprint Editor, which displays nodes on a graph. When an update is requested, all the **Graph** Panel widgets are cleared and re-created. This is preferable to fine-grain invalidation because it is simpler and more maintainable.

모델 구조가 크게 변경된다면 다 지우고 다시 그리는 게 낫다. (like 그래프)

## Child Slots

All Slate widgets store children in child slots. (As opposed to storing a plain array of child widgets.) Child slots always store a valid widget; by default they store the **SNullWidget**, which is a widget with no visualization or interaction. Each type of widget can declare its own type of child slot, which caters to its specific needs. Consider that **SVerticalSlot** arranges its children completely differently than an **SCanvas**, which is quite different from SUniformGridPanel. The Slots allow each type of panel to ask for a set of per-child settings that affect the arrangement of the children.

모든 Slate 위젯은 하위 슬롯에 하위 항목을 저장합니다. (하위 위젯의 일반 배열을 저장하는 것과 반대입니다.) 하위 슬롯은 항상 유효한 위젯을 저장합니다. 기본적으로 시각화나 상호작용이 없는 위젯인 SNullWidget을 저장합니다. 각 위젯 유형은 특정 요구 사항을 충족하는 자체 하위 슬롯 유형을 선언할 수 있습니다. SVerticalSlot은 SUniformGridPanel과 상당히 다른 SCanvas와 완전히 다르게 자식을 배열한다는 점을 고려하세요. 슬롯을 사용하면 각 유형의 패널에서 하위 배열에 영향을 미치는 하위별 설정 세트를 요청할 수 있습니다.


## Widget Roles

위젯은 세 가지 형태로 제공됩니다

- **Leaf Widgets** - widgets with no child slots. e.g. STextBlock displays a piece of text. It has native knowledge of how to draw text. 하위 슬롯이 없는 위젯. 예를 들어 STextBlock은 텍스트 조각을 표시합니다. 텍스트를 그리는 방법에 대한 기본 지식이 있습니다.
  
- **Panels** - widgets with a dynamic number of child slots. e.g. **SVerticalBox** arranges any number of children vertically given some layout rules. 동적 개수의 하위 슬롯이 있는 위젯. 예를 들어 SVerticalBox는 일부 레이아웃 규칙에 따라 여러 자식을 수직으로 정렬합니다.

- **Compound Widgets** - widgets with a fixed number of explicitly named child slots. e.g. **SButton** has one slot called Content which contains any widgets inside the button. 동적 개수의 하위 슬롯이 있는 위젯. 예를 들어 SVerticalBox는 일부 레이아웃 규칙에 따라 여러 자식을 수직으로 정렬합니다.


## Layout
Slate layout is accomplished in two passes.


1. Pass 1: **Cache Desired Size** - the relevant functions are **SWidget::CacheDesiredSize** and **SWidget::ComputeDesiredSize**
   
2. Pass 2: **ArrangeChildren** - the relevant function is **SWidget::ArrangeChildren**

#### Pass 1: Cache Desired Size

The goal of this pass is to figure out how much space each widget wants to occupy.  Widgets with no children (i.e. leaf widgets) are asked to compute and cache their desired size based on their intrinsic properties.

Leaf Widget은 자기 고유의 intrinsic 특성을 기준으로 사이즈가 계산되고 캐싱되도록 요청됨.

Widgets that combine other widgets (i.e. compound widgets and panels) use special logic to determine their desired size as a function of the size of their children.
다른 위젯과 결합하는 위젯 (패널과 Compound 위젯)은 special logic을 통해 사이즈가 계산됩니다.

Note that each type of widget is only required to implement **ComputeDesiredSize();** the caching and traversal logic are implemented by Slate.
각 위젯 유형은 ComputeDesiredSize()를 구현하는 데만 필요합니다. 캐싱 및 순회 논리는 Slate에 의해 구현됩니다.

Slate guarantees that when ComputeDesiredSize() is called on a widget, its children have already computed and cached their desired size. Thus, this is a bottom-up pass.
Slate는 위젯에서 ComputeDesiredSize()가 호출될 때 해당 하위 항목이 이미 원하는 크기를 계산하고 캐시했음을 보장합니다. 따라서 이것은 **상향식** 패스입니다.

다음의 예를 고려해보자.

![[SLATE_ARCITECTURE_3.png]]

An STextBlock widget would compute its desired size by measuring the string that it is displaying. The SImage widget would determine its size based on the image data it is showing. Assume that the text inside the textblock requires 14 slate units of space, and the image requires 8. The horizontal panel arranges widgets horizontally, and therefore requires 14 + 8 = 22 units of space.

STextBlock의 Text를 고려하면 14사이즈 되어야하고, SImage의 Image가 들어가려면 8사이즈가 필요하다. 그래서 Horizontal Box는 14+8 = 22로 결정된다.


#### Pass 2: ArrangeChildren

ArrangeChildren is a top-down pass. Slate begin at the top-level windows and asks each window to arrange its children based on the constraints provided by the programmers. When the space allotted for each child is known, Slate can recur and arrange the children's children. The recursion continues until all the children are arranged.

AlignChildren은 **하향식 패스**입니다. 슬레이트는 최상위 창에서 시작하여 프로그래머가 제공한 제약 조건에 따라 하위 창을 정렬하도록 각 창에 요청합니다. 각 자식에게 할당된 공간이 알려지면 Slate는 자식의 자식을 반복해서 배열할 수 있습니다. 모든 하위 항목이 정렬될 때까지 재귀가 계속됩니다.


![[SLATE_ARCITECTURE_4.png]]

먼저 Horizontal Box에 사이즈를 할당하고 그것에 맞춰 자식 컴포넌트의 사이즈가 결정된다.

STextBlock은 Autosize이고 내부 Text는 14 사이즈를 필요로하기 때문에 14로 결정된다 SImage의 내부 이미지는 8의 사이즈를 필요로 하지만 Fill WIdth이기 때문에 11로 결정된다.

Note that in the actual SHorizontalBox widget, the alignment of the SImage within its slot would be driven by the **HAlign** property, which can be Left, Center, Right, or Fill.

In practice, Slate never performs a full ArrangeChildren pass. Instead, this functionality is used to implement other functionality. Key examples are hit detection and painting.
실제로 Slate는 전체 AlignChildren 패스를 수행하지 않습니다. 대신 이 기능은 다른 기능을 구현하는 데 사용됩니다. 주요 예로는 히트 감지 및 페인팅이 있습니다.

## Drawing Slate : OnPaint

During the paint pass, Slate iterates over all the visible widgets and produces a list of draw elements which will be consumed by the rendering system. This list is produced anew for every frame. 페인트 패스 중에 Slate는 표시되는 모든 위젯을 반복하고 렌더링 시스템에서 사용할 그리기 요소 목록을 생성합니다. 이 목록은 매 프레임마다 새로 생성됩니다.

We begin at the top level windows and recur down the hierarchy, appending the draw elements of every widget to the draw list.  최상위 창에서 시작하여 계층 구조를 따라 반복하여 모든 위젯의 그리기 요소를 그리기 목록에 추가합니다.

Widgets tend to do two things during paint: they output actual draw elements or figure out where a child widget should exist and ask the child widget to paint itself. 위젯은 페인트하는 동안 두 가지 작업을 수행하는 경향이 있습니다. 즉, 실제 그리기 요소를 출력하거나 하위 위젯이 어디에 있어야 하는지 파악하고 하위 위젯에게 자체적으로 페인트하도록 요청합니다.

Thus, we can think of a simplified general-case **OnPaint** function as being 따라서 단순화된 일반적인 경우의 OnPaint 함수를 다음과 같이 생각할 수 있습니다.

```cpp
    // An arranged child is a widget and its allotted geometry
struct ArrangedChild
{
	Widget;
    Geometry;
};

OutputElements OnPaint( AllottedGeometry )
{
	// Arrange all the children given our allotted geometry
    Array<ArrangedChild> ArrangedChildren = ArrangeChildrenGiven( AllottedGeometry );
	// Paint the children
    for each ( Child in ArrangedChildren )
    {
        OutputElements.Append( Child.Widget.OnPaint( Child.Geometry ) );
    }

    // Paint a border
    OutputElements.Append( DrawBorder() );
}
```

## Anatomy of a SWidget

The key functions that define an SWidget's behavior in Slate are:

- ComputeDesiredSize() - responsible for desired size. 원하는 사이즈를 책임집니다.|
  
- ArrangeChildren() - responsible for arrangement of children within the parent's allotted area. 부모에게 할당된 공간 내에서 자녀를 배치할 책임이 있습니다.
  
- OnPaint() - responsible for appearance.
  
- Event handlers - these are of the form OnSomething. These are functions that may be invoked on your widget by Slate at various times. 이는 OnSomething 형식입니다. 이는 Slate가 위젯에서 다양한 시간에 호출할 수 있는 함수입니다.
  
  
  ## Composition
  
 Composition is the notion that any slot should be able to contain arbitrary widget content. This affords users of Slate a great deal of flexibility. Composition is used whenever possible in core Slate widgets. 구성은 모든 슬롯이 임의의 위젯 콘텐츠를 포함할 수 있어야 한다는 개념입니다. 이는 Slate 사용자에게 상당한 유연성을 제공합니다. 컴포지션은 핵심 슬레이트 위젯에서 가능할 때마다 사용됩니다.
