@echo off
setlocal EnableDelayedExpansion
title WiFi Forensics Extractor
color 0A

:: ============================================================
::  WiFi Logs / Artifacts Extractor for Digital Forensics
::  Collects WiFi profiles, passwords (if authorized), event
::  logs, adapter info, and connection history using only
::  built-in Windows tools (netsh, wevtutil, reg, ipconfig).
::
::  Run this script AS ADMINISTRATOR for full results.
:: ============================================================

:: ---- Check for admin rights ----
net session >nul 2>&1
if %errorLevel% neq 0 (
    echo [!] This script should be run as Administrator for complete results.
    echo [!] Some data ^(like saved WiFi passwords^) will be skipped otherwise.
    echo.
    timeout /t 3 >nul
)

:: ---- Set up output folder with case timestamp ----
for /f "tokens=2 delims==" %%I in ('wmic os get localdatetime /value') do set dt=%%I
set STAMP=%dt:~0,4%-%dt:~4,2%-%dt:~6,2%_%dt:~8,2%-%dt:~10,2%-%dt:~12,2%
set CASEDIR=%~dp0WiFi_Forensics_%COMPUTERNAME%_%STAMP%
mkdir "%CASEDIR%" 2>nul

echo ============================================================
echo   WiFi Forensics Extractor
echo   Target Host : %COMPUTERNAME%
echo   User        : %USERNAME%
echo   Output Dir  : %CASEDIR%
echo ============================================================
echo.

:: ---- 1. Case/Chain-of-custody header ----
(
    echo Case Notes / Chain of Custody
    echo ------------------------------
    echo Collected by : %USERNAME%
    echo Hostname     : %COMPUTERNAME%
    echo Collection start : %DATE% %TIME%
    echo Script: WiFi_Forensics_Extractor.bat
) > "%CASEDIR%\00_case_info.txt"

:: ---- 2. List of all saved WiFi profiles ----
echo [*] Exporting list of saved WiFi profiles...
netsh wlan show profiles > "%CASEDIR%\01_wifi_profiles_list.txt"

:: ---- 3. Detailed info + keys for each profile ----
echo [*] Exporting detailed profile data (including keys, if admin)...
set "PROFILEDETAIL=%CASEDIR%\02_wifi_profiles_detailed.txt"
if exist "%PROFILEDETAIL%" del "%PROFILEDETAIL%"
for /f "tokens=* delims=" %%A in ('netsh wlan show profiles ^| findstr /R /C:"All User Profile"') do (
    set "line=%%A"
    for /f "tokens=2 delims=:" %%B in ("!line!") do (
        set "profile=%%B"
        set "profile=!profile:~1!"
        echo ==================================================== >> "%PROFILEDETAIL%"
        echo Profile: !profile! >> "%PROFILEDETAIL%"
        echo ==================================================== >> "%PROFILEDETAIL%"
        netsh wlan show profile name="!profile!" key=clear >> "%PROFILEDETAIL%"
        echo. >> "%PROFILEDETAIL%"
    )
)

:: ---- 4. Exported .xml profile files (as stored by Windows) ----
echo [*] Exporting raw XML WiFi profile files...
mkdir "%CASEDIR%\03_xml_profiles" 2>nul
netsh wlan export profile folder="%CASEDIR%\03_xml_profiles" key=clear >nul 2>&1

:: ---- 5. Current network / adapter / driver info ----
echo [*] Capturing wireless adapter and driver info...
netsh wlan show drivers > "%CASEDIR%\04_wlan_drivers.txt"
netsh wlan show interfaces > "%CASEDIR%\05_wlan_interfaces.txt"
netsh wlan show all > "%CASEDIR%\06_wlan_show_all.txt"
ipconfig /all > "%CASEDIR%\07_ipconfig_all.txt"

:: ---- 6. Nearby / historical visible networks (BSS) ----
echo [*] Capturing visible network list (current scan)...
netsh wlan show networks mode=bssid > "%CASEDIR%\08_visible_networks.txt"

:: ---- 7. Wireless-related Windows Event Logs ----
echo [*] Exporting Windows WLAN event logs (last connections, disconnects, auth)...
mkdir "%CASEDIR%\09_event_logs" 2>nul
wevtutil epl "Microsoft-Windows-WLAN-AutoConfig/Operational" "%CASEDIR%\09_event_logs\WLAN-AutoConfig.evtx" 2>nul
wevtutil epl "Microsoft-Windows-NetworkProfile/Operational" "%CASEDIR%\09_event_logs\NetworkProfile.evtx" 2>nul
wevtutil epl "Microsoft-Windows-WWAN-SVC-EVENTS/Operational" "%CASEDIR%\09_event_logs\WWAN-SVC.evtx" 2>nul

:: Also dump the same logs as readable text
wevtutil qe "Microsoft-Windows-WLAN-AutoConfig/Operational" /f:text /c:500 > "%CASEDIR%\09_event_logs\WLAN-AutoConfig_readable.txt" 2>nul
wevtutil qe "Microsoft-Windows-NetworkProfile/Operational" /f:text /c:500 > "%CASEDIR%\09_event_logs\NetworkProfile_readable.txt" 2>nul

:: ---- 8. Registry artifacts: known networks / signatures ----
echo [*] Exporting registry keys for known network profiles...
mkdir "%CASEDIR%\10_registry" 2>nul
reg export "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles" "%CASEDIR%\10_registry\NetworkList_Profiles.reg" /y >nul 2>&1
reg export "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures" "%CASEDIR%\10_registry\NetworkList_Signatures.reg" /y >nul 2>&1
reg export "HKLM\SYSTEM\CurrentControlSet\Services\WlanSvc" "%CASEDIR%\10_registry\WlanSvc_Service.reg" /y >nul 2>&1

:: ---- 9. ARP cache / active connections (context at time of capture) ----
echo [*] Capturing ARP cache and active connections...
arp -a > "%CASEDIR%\11_arp_cache.txt"
netstat -ano > "%CASEDIR%\12_netstat.txt"

:: ---- 10. Hashing everything for integrity (chain of custody) ----
echo [*] Generating SHA256 hashes of all collected files...
set "HASHFILE=%CASEDIR%\HASHES_SHA256.txt"
if exist "%HASHFILE%" del "%HASHFILE%"
for /r "%CASEDIR%" %%F in (*) do (
    if not "%%~fF"=="%HASHFILE%" (
        certutil -hashfile "%%F" SHA256 | findstr /v "hash" >> "%HASHFILE%"
        echo   ^<-- %%F >> "%HASHFILE%"
    )
)

echo Collection end : %DATE% %TIME% >> "%CASEDIR%\00_case_info.txt"

echo.
echo ============================================================
echo   DONE. All artifacts saved to:
echo   %CASEDIR%
echo ============================================================
pause
