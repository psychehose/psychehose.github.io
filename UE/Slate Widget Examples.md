
## Common Slate Arguments (공통)

- IsEnabled - This will specify whether or not the widget is able to be interacted with. If it is disabled, it will be greyed out. 위젯이 상호작용할 수 있는지 여부를 지정합니다. 비활성화된 경우 회색으로 표시됩니다.

- ToolTip - This will specify what kind of custom SToolTip widget will be used for this widget's tool tip. If not specified, it will not appear. 이 위젯의 ​​도구 설명에 사용할 사용자 정의 SToolTip 위젯의 종류를 지정합니다. 지정하지 않으면 나타나지 않습니다.

- ToolTipText - This will specify what kind of text will show up as a simple tooltip for this widget's tool tip. If not specified, or if the ToolTip attribute was used, it will not appear. 이 위젯의 ​​도구 설명에 대한 간단한 도구 설명으로 표시할 텍스트 종류를 지정합니다. 지정하지 않거나 ToolTip 속성을 사용한 경우에는 표시되지 않습니다.

- Cursor - This will specify what cursor will appear while the mouse is hovering over this widget. 이는 마우스가 이 위젯 위에 있는 동안 어떤 커서가 나타날지 지정합니다.
  
- Visibility 


다음 인수는 모든 단일 위젯에 있는 것은 아니지만 대부분의 위젯에 있습니다.

- Text - This will specify the text that this widget will have, if applicable.
  
- Content - This will specify what widget should be placed in the content section of the widget, if applicable.
  
- ReadOnly - This will prevent this widget from being editable if _true_.
  
- Style - This will specify the style of images or text font used by the widget. How this is applicable varies by widget.
  
- Padding - The padding of a widget amount of spacing in slate units around the left, top, right, and bottom parts of the widget within its parent. These can be specified as a single value for all four parts, or as a horizontal and vertical value, or as four separate values.
  
- HAlign - The horizontal alignment of content within the widget.
  
- VAlign - The vertical alignment of content within the widget.
  
### Visibility

The visibility of a widget determines how the widget will appear, as well as its interactivity.
위젯의 가시성에 따라 위젯이 표시되는 방식과 상호작용성이 결정됩니다.

- Visible (Default) - The widget will appear normally. 위젯이 정상적으로 나타납니다.
  
- Collapsed - The widget will not be visible and will take up no space in the layout. It will not be interactive. 위젯은 표시되지 않으며 레이아웃에서 공간을 차지하지 않습니다. 인터렉션형이 아닙니다.

- Hidden - The widget will not be visible, but will take up space in the layout. It will not be interactive. 위젯은 표시되지 않지만 레이아웃에서 공간을 차지합니다. 인터렉션형이 아닙니다.

- HitTestInvisible - Visible to the user, but only as art. It will not be interactive. 사용자에게 표시되지만 아트로만 표시됩니다. 대화형이 아닙니다.

- SelfHitTestInvisible - Same as HitTestInvisible, but does not apply to child widgets. HitTestInvisible과 동일하지만 하위 위젯에는 적용되지 않습니다.
  
  ### Alignment
  
- HAlign_Fill/VAlign_Fill
- HAlign_Left
- VAlign_Top
- HAlign_Center/VAlign_Center
- HAlign_Right
- VAlign_Bottom
