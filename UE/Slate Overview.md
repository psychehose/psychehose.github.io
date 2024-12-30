**Slate** is a completely custom and platform agnostic user interface framework that is designed to make building the user interfaces for tools and applications such as Unreal Editor, or in-game user interfaces, fun and efficient. It combines a declarative syntax with the ability to easily design, lay out, and style components that allows for easily creating and iterating on UIs.

The Slate UI solution makes it extremely easy to put together graphical user interfaces for tools and applications and iterate on them quickly.


## Declarative Syntax

Slate의 선언적 구문을 사용하면 프로그래머가 간접 계층을 추가하지 않고도 UI 구축에 액세스할 수 있습니다. 새 위젯을 선언하고 생성하는 프로세스를 단순화하기 위해 완전한 매크로 세트가 제공됩니다.

```cpp
SLATE_BEGIN_ARGS( SSubMenuButton )
    : _ShouldAppearHovered( false )
    {}
    /** The label to display on the button */
    SLATE_ATTRIBUTE( FString, Label )
    /** Called when the button is clicked */
    SLATE_EVENT( FOnClicked, OnClicked )
    /** Content to put in the button */
    SLATE_NAMED_SLOT( FArguments, FSimpleSlot, Content )
    /** Whether or not the button should appear in the hovered state */
    SLATE_ATTRIBUTE( bool, ShouldAppearHovered )
SLATE_END_ARGS()
```

## Composition


```cpp
// Add a new section for static meshes
ContextualEditingWidget->AddSlot()
.Padding( 2.0f )
[
    SNew( SDetailSection )
    .SectionName("StaticMeshSection")
    .SectionTitle( LOCTEXT("StaticMeshSection", "Static Mesh").ToString() )
    .Content()
    [
        SNew( SVerticalBox )
        + SVerticalBox::Slot()
        .Padding( 3.0f, 1.0f )
        [
            SNew( SHorizontalBox )
            + SHorizontalBox::Slot()
            .Padding( 2.0f )
            [
                SNew( SComboButton )
                .ButtonContent()
                [
                    SNew( STextBlock )
                    .Text( LOCTEXT("BlockingVolumeMenu", "Create Blocking Volume") ) 
                    .Font( FontInfo )
                ]
                .MenuContent()
                [
                    BlockingVolumeBuilder.MakeWidget()
                ]
            ]
        ]

    ]
];
```

위의 코드는 아래 UI를 만든다.

![[SLATE_OVERVIEW_1.png]]


## Styles

Styles can be created and applied to the various parts of a widget. This makes it easy to iterate on the look of the components in the UI, as well as share and reuse styles. 스타일을 생성하여 위젯의 다양한 부분에 적용할 수 있습니다. 이를 통해 UI의 구성 요소 모양을 쉽게 반복할 수 있을 뿐만 아니라 스타일을 공유하고 재사용할 수도 있습니다.

```cpp
// Tool bar
{
    Set( "ToolBar.Background", FSlateBoxBrush( TEXT("Common/GroupBorder"), FMargin(4.0f/16.0f) ) );

    Set( "ToolBarButton.Normal", FSlateNoResource() );      // Note: Intentionally transparent background
    Set( "ToolBarButton.Pressed", FSlateBoxBrush( TEXT("Old/MenuItemButton_Pressed"), 4.0f/32.0f ) );
    Set( "ToolBarButton.Hovered", FSlateBoxBrush( TEXT("Old/MenuItemButton_Hovered"), 4.0f/32.0f ) );

    // Tool bar buttons are sometimes toggle buttons, so they need styles for "checked" state
    Set( "ToolBarButton.Checked", FSlateBoxBrush( TEXT("Old/MenuItemButton_Pressed"),  4.0f/32.0f, FLinearColor( 0.3f, 0.3f, 0.3f ) ) );
    Set( "ToolBarButton.Checked_Hovered", FSlateBoxBrush( TEXT("Old/MenuItemButton_Hovered"),  4.0f/32.0f ) );
    Set( "ToolBarButton.Checked_Pressed", FSlateBoxBrush( TEXT("Old/MenuItemButton_Pressed"),  4.0f/32.0f, FLinearColor( 0.5f, 0.5f, 0.5f ) ) );

    // Tool bar button label font
    Set( "ToolBarButton.LabelFont", FSlateFontInfo( TEXT("Roboto-Regular"), 8 ) );
}
```



```cpp
SNew( SBorder )
.BorderImage( FEditorStyle::GetBrush( "ToolBar.Background" ) )
.Content()
[
    SNew(SHorizontalBox)

    // Compile button (faked to look like a multibox button)
    +SHorizontalBox::Slot()
    [
        SNew(SButton)
        .Style(TEXT("ToolBarButton"))
        .OnClicked( InKismet2.ToSharedRef(), &FKismet::Compile_OnClicked )
        .IsEnabled( InKismet2.ToSharedRef(), &FKismet::InEditingMode )
        .Content()
        [
            SNew(SVerticalBox)
            +SVerticalBox::Slot()
            .Padding( 1.0f )
            .HAlign(HAlign_Center)
            [
                SNew(SImage)
                .Image(this, &SBlueprintEditorToolbar::GetStatusImage)
                .ToolTipText(this, &SBlueprintEditorToolbar::GetStatusTooltip)
            ]
            +SVerticalBox::Slot()
            .Padding( 1.0f )
            .HAlign(HAlign_Center)
            [
                SNew(STextBlock)
                .Text(LOCTEXT("CompileButton", "Compile"))
                .Font( FEditorStyle::GetFontStyle( FName( "ToolBarButton.LabelFont" ) ) )
                .ToolTipText(LOCTEXT("CompileButton_Tooltip", "Recompile the blueprint"))
            ]
        ]
    ]
]
```


## Developer tools

The Slate Widget Reflector provides a means of debugging and analyzing the UI and associated code. This helps track down bugs and undesirable behavior as well as profile and optimize your user interface. 슬레이트 위젯 리플렉터는 UI 및 관련 코드를 디버깅하고 분석하는 수단을 제공합니다. 이는 버그와 바람직하지 않은 동작을 추적하고 사용자 인터페이스를 프로파일링하고 최적화하는 데 도움이 됩니다.

  
## Using Slate in a Project

슬레이트를 사용하기 위해서 아래 모듈을 설정해줘야 한다.

![[SLATE_OVERVIEW_2.png]]


