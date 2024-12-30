
#### Unreal Engine pak 내용물 추출
   
```cmd
UnrealPak.exe {추출할 pak 파일 경로} -Extract {추출 되는 경로}
```


#### 윈도우 읽기 전용 아닌 파일 찾기

현재 디렉토리와 하위 디렉토리들에서 읽기 전용이 아닌 파일을 찾는 방법

```
Get-ChildItem -Recurse | Where-Object { !$_.PSIsContainer -and !$_.Attributes.HasFlag([System.IO.FileAttributes]::ReadOnly) } | Select-Object FullName
```

