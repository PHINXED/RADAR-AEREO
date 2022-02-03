```powershell
Install-Module -Name Posh-SSH -RequiredVersion 2.0.2
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned

Set-Location C:\Users\PHINX\Desktop\dump1090-win.1.10.3010.14

Start-Process -File .\dump1090.bat -RedirectStandardOutput resultado.txt
Start-Sleep -Seconds 30
Stop-Process -Name cmd
Stop-Process -Name dump1090
Get-Content resultado.txt | Out-File procesado.txt -Append
rm resultado.txt
 
Start-Sleep -Seconds 5
(Get-Content .\procesado.txt) -split "`n" | %{
$_ | Select-String " S " | Out-File fin.txt -Append
}

$fly=Import-Csv -Encoding Unicode .\fin.txt
rm .\soluc.txt
"Hex     ,Mode  ,Sqwk  ,Flight   ,Alt    ,Spd  ,Hdg    ,Lat      ,Long   ,Sig  ,Msgs   ,Ti" | out-file soluc.txt
$fly.H1 -split "`n" | %{($_).Substring(0,8)+","+($_).Substring(8,6)+","+($_).Substring(14,6)+","+($_).Substring(20,9)+","+($_).Substring(29,7)+","+($_).Substring(36,5)+","+($_).Substring(41,6)+","+($_).Substring(47,9)+","+($_).Substring(56,9)+","+($_).Substring(65,6)+","+($_).Substring(71,6)+","+($_).Substring(77,1) | Out-File soluc.txt -append}
 
$flyfinal=Import-Csv -Encoding Unicode .\soluc.txt
$flyfinal.'Flight ' | Group-Object
$flyfinal.'Flight ' | Group-Object | Select name | Select-String "BAW2799"

foreach($resultado in ($flyfinal.'Flight ' | Group-Object | Select name))
{
    If($resultado -match "BAW2799")
    {
        New-SSHSession -ComputerName 192.168.43.135 -Credential (Get-Credential) -Force
        Invoke-SSHCommand -Index 0 'python -c "import RPi.GPIO as GPIO;GPIO.setmode(GPIO.BOARD);GPIO.setup(11, GPIO.OUT);GPIO.output(11, True);"'
        Start-Sleep -Seconds 10
        Invoke-SSHCommand -Index 0 'python -c "import RPi.GPIO as GPIO;GPIO.setmode(GPIO.BOARD);GPIO.setup(11, GPIO.OUT);GPIO.output(11, False);GPIO.cleanup();"'
    
    }
}
```
- - - - - - - 
### ESQUEMA CONEXION

- - - - - - - 
![1](https://user-images.githubusercontent.com/82601259/152407832-1c7d2795-2244-4e6e-bc92-3350b9f6c5b3.png)
