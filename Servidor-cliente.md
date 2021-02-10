# Servidor cliente forwarding

## Ejecutar en este orden
- CLIENTE 2 
- SERVIDOR
- CLIENTE 1

## Cliente 1
```powershell

$port=2021
$endpoint = new-object System.Net.IPEndPoint ([IPAddress]::Loopback,$port)
$udpclient=new-Object System.Net.Sockets.UdpClient

#Crear un certificado para firmar el mensaje
#https://www.jesusninoc.com/11/18/exports-a-certificate-to-a-personal-information-exchange-pfx-file/
$cert = New-SelfSignedCertificate -DnsName daniexamen4 -CertStoreLocation "Cert:\CurrentUser\My" -KeyUsage KeyEncipherment,DataEncipherment, KeyAgreement -Type DocumentEncryptionCert  -KeyExportPolicy ExportableEncrypted

#Encrypts content by using the Cryptographic Message Syntax format
"Hello" | Protect-CmsMessage -To cn=daniexamen4 -OutFile secret.txt

$val=[string](Get-Content secret.txt)

$b=[Text.Encoding]::ASCII.GetBytes($val.ToString())
$bytesSent=$udpclient.Send($b,$b.length,$endpoint)
$udpclient.Close()
```
----------------------------------------

## Servidor
```powershell

$port=2021
$endpoint = new-object System.Net.IPEndPoint ([IPAddress]::Any,$port)
$udpclient=new-Object System.Net.Sockets.UdpClient $port
#Comprobar que el hash recibido y el hash del fichero recbido es correcto
$content=$udpclient.Receive([ref]$endpoint)
$convert=[Text.Encoding]::ASCII.GetString($content)
$convert | Out-File ficherorecibido.txt
$mensaje=Unprotect-CmsMessage -Path ficherorecibido.txt
if ($mensaje -eq "hello"){  
    $port=2022
        $endpoint = new-object System.Net.IPEndPoint ([IPAddress]"127.0.0.1",$port)
        $udpclient=new-Object System.Net.Sockets.UdpClient
        $b=[Text.Encoding]::ASCII.GetBytes($mensaje)
        $bytesSent=$udpclient.Send($b,$b.length,$endpoint)
}
$udpclient.Close()
```
--------------------------------

## Cliente 2
```powershell
$port=2022
$endpoint = new-object System.Net.IPEndPoint ([IPAddress]::Any,$port)
$udpclient=new-Object System.Net.Sockets.UdpClient $port
 
#Comprobar que el hash recibido y el hash del fichero recbido es correcto
$content=$udpclient.Receive([ref]$endpoint)
$convert=[Text.Encoding]::ASCII.GetString($content)
$convert=[Text.Encoding]::ASCII.GetString($content)

if($convert -eq "hello")
{
    "Aprobado"
} 
$udpclient.Close()
```
