#unreal #error 

          Crashed: com.apple.main-thread
0  libobjc.A.dylib                0x3020 objc_msgSend + 32
1  U2Client                       0x246e35c -[FIOSView insertText:] + 636 (IOSView.cpp:636)
2  U2Client                       0x246eaac -[FIOSView unmarkText] + 811 (IOSView.cpp:811)
3  UIKitCore                      0xdc95d0 -[UIResponder(UITextInput_Internal) _unmarkText] + 68
4  UIKitCore                      0xa7d590 -[UIKBInputDelegateManager unmarkText] + 48
5  UIKitCore                      0x4d3e18 -[UIKeyboardImpl _teardownExistingDelegate:forSetDelegate:force:delayEndInputSession:] + 2180
6  UIKitCore                      0x4d2808 -[UIKeyboardImpl setDelegate:force:delayEndInputSession:] + 624
7  UIKitCore                      0x2c4b68 -[UIKeyboardSceneDelegate _reloadInputViewsForKeyWindowSceneResponder:force:fromBecomeFirstResponder:] + 2584
8  UIKitCore                      0x325d3c -[UIKeyboardSceneDelegate _reloadInputViewsForResponder:force:fromBecomeFirstResponder:] + 88
9  UIKitCore                      0x364d8c -[UIResponder _finishResignFirstResponderFromBecomeFirstResponder:] + 328
10 UIKitCore                      0x3643d0 -[UIResponder resignFirstResponder] + 352
11 U2Client                       0x246e8fc -[FIOSView resignFirstResponder] + 740 (IOSView.cpp:740)
12 UIKitCore                      0x52cfa8 -[UIView(UITextField) endEditing:] + 136
13 U2Client                       0x246e7bc __30-[FIOSView DeactivateKeyboard]_block_invoke + 102 (IOSInputInterface.h:102)
14 libdispatch.dylib              0x213c _dispatch_call_block_and_release + 32
15 libdispatch.dylib              0x3dd4 _dispatch_client_callout + 20
16 libdispatch.dylib              0x125a4 _dispatch_main_queue_drain + 988
17 libdispatch.dylib              0x121b8 _dispatch_main_queue_callback_4CF + 44
18 CoreFoundation                 0x56710 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ + 16
19 CoreFoundation                 0x53914 __CFRunLoopRun + 1996
20 CoreFoundation                 0x52cd8 CFRunLoopRunSpecific + 608
21 GraphicsServices               0x11a8 GSEventRunModal + 164
22 UIKitCore                      0x40a90c -[UIApplication _run] + 888
23 UIKitCore                      0x4be9d0 UIApplicationMain + 340
24 U2Client                       0xdb1650 main + 568 (LaunchIOS.cpp:568)
25 ???                            0x1ac721e4c (누락)





```objc
- (void)insertText:(NSString*)theText
{
	if (nil != CachedMarkedText) {
		[CachedMarkedText release] ;
		CachedMarkedText = nil;
	}

	// insert text one key at a time, as chars, not keydowns
	for (int32 CharIndex = 0; CharIndex < [theText length]; CharIndex++)
	{
		int32 Char = [theText characterAtIndex : CharIndex];

		// FPlatformMisc::LowLevelOutputDebugStringf(TEXT("sending key '%c' to game\n"), Char);

		if (Char == '\n')
		{
			// send the enter keypress
			FIOSInputInterface::QueueKeyInput(KEYCODE_ENTER, Char);

			// hide the keyboard
			[self resignFirstResponder] ;
		}
		else
		{
			FIOSInputInterface::QueueKeyInput(Char, Char);
		}
	}
}



```

원인 예상
1. 이미 해제한 CachedMarkedText를 접근하려고 했기 때문에
2. theText에 nil 값이 들어오기 때문에

해결
1. iOS - UI 관련된 작업은 main - thread에서 실행