# idle-shutdown
Create Idle Shutdown from Windows Task Scheduler

Windows OS 작업 스케줄러에 `유휴종료` 작업 등록

1. 유휴상태가 1시간 이상 지속되면
2. 대기시간 없이 즉시
3. 시스템종료 명령 실행 - 5분 후 시스템 종료 메시지 출력
   - shutdown /s /t 300 /d p:0:0 /c "컴퓨터를 사용하지 않아 5분 후 종료합니다. 취소하려면 shutdown /a 를 실행하세요."
4. 5분 간격으로 작업 무한 반복

> Filename: Register_idleShutdown_Task.ps1

```powershell
# XML 템플릿 작성
$taskXml = @"
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.4" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <Triggers>
    <IdleTrigger>
      <Repetition>
        <Interval>PT5M</Interval>  <!-- 작업 반복 간격 (5분) -->
      </Repetition>
    </IdleTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>SYSTEM</UserId>
      <LogonType>InteractiveToken</LogonType>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>  <!-- 배터리 상태에서도 실행 -->
    <StopIfGoingOnBatteries>false</StopIfGoingOnBatteries>          <!-- 배터리 상태로 전환돼도 중단 안 함 -->
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>true</StartWhenAvailable>
    <IdleSettings>
      <Duration>PT1H</Duration>    <!-- 유휴 상태가 1시간 이상 지속되면 실행 -->
      <WaitTimeout>PT0M</WaitTimeout>  <!-- 다음 시간 동안 유휴 상태 대기: 대기 안 함 -->
      <StopOnIdleEnd>true</StopOnIdleEnd> <!-- 컴퓨터의 유휴 상태가 끝나면 중지 -->
      <RestartOnIdle>true</RestartOnIdle>  <!-- 유휴 상태가 재개되면 다시 시작 -->
    </IdleSettings>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>shutdown.exe</Command>
      <Arguments>/s /t 300 /d p:0:0 /c "컴퓨터를 사용하지 않아 5분 후 종료합니다. 취소하려면 shutdown /a 를 실행하세요."</Arguments>
    </Exec>
  </Actions>
</Task>
"@

# XML을 작업 스케줄러에 등록
$taskName = "유휴종료"

# XML 파일 생성
$taskFile = "$env:temp\$taskName.xml"
$taskXml | Out-File -FilePath $taskFile -Encoding Unicode

# 작업 등록
schtasks /create /tn $taskName /xml $taskFile /f

# 임시 파일 삭제
Remove-Item -Path $taskFile

Write-Host "작업 스케줄러에 '유휴종료' 작업이 정상적으로 등록되었습니다."

```

> PowerShell (Admin Mode)

```powershell
powershell -ExecutionPolicy Bypass -File .\Register_IdleShutdown_Task.ps1
```
