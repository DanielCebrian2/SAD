
# Crear Json

```powershell

$alumnos = '{
  "alumnos": {
    "alumno": [
      {
        "id": "1",
        "Nombre":  "Juanito",
        "Subject": "Asunto del correo",
        "cuerpo":  "Hola mundo",
        "Correo":  "xxx@gmail.com"
      },
      {
        "id": "2",
        "Nombre": "Pepe",
        "Subject": "Asunto del correo",
        "cuerpo":  "Hola mundo",
        "Correo": "xxx@gmail.com"
      }
    ]
  }
}' | ConvertFrom-Json


```
------------------------------------------------------------------------------------------

# Enviar correo desde powershell 

```powershell

$alumnos.alumnos.alumno | %{
#$_.Nombre, $_.Correo

$EmailFrom = "xxx@hotmail.com"
$EmailTo = $_.Correo
$Subject = $_.Subject
 
$SMTPServer = "smtp.outlook.com"
$SMTPClient = New-Object Net.Mail.SmtpClient($SmtpServer, 587)
$SMTPClient.EnableSsl = $true
 
$message = New-Object System.Net.Mail.MailMessage $EmailFrom, $EmailTo
$message.Subject = $Subject
$message.IsBodyHTML = $true
 
$message.Body = $_.cuerpo
$pass = Read-Host "introduce contraseña" -AsSecureString
$SMTPClient.Credentials = New-Object System.Net.NetworkCredential($EmailFrom, $pass)
 
$SMTPClient.Send($message)
}
```
