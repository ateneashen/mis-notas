# LAN-Share Setup - SMB Nativo Mejorado

Paquete estandarizado para compartir archivos entre PCs de la misma red LAN usando SMB3 con cifrado.

## Requisitos

- Windows 10/11 (Pro o superior recomendado)
- Red privada (domestica/trabajo) configurada como "Perfil Privado" en el Firewall
- Cuenta con privilegios de Administrador para ejecutar los scripts de setup

## Estructura del paquete

```
LAN-Setup/
├── config.json              <-- Editar antes de ejecutar nada
├── setup-smb-server.ps1     <-- Ejecutar como Admin (una vez por PC)
├── connect-lan-shares.ps1   <-- Conecta/reconecta unidades de otros PCs
├── install-startup-task.ps1 <-- Ejecutar como Admin (registra auto-inicio)
├── verify-lan-config.ps1    <-- Comprueba que nombres e IPs son correctos
└── README.md                <-- Este archivo
```

## Instrucciones por PC

### Paso 1: Editar config.json

Abre `config.json` y adapta los datos de **este PC**:

```json
"thisPC": {
    "name": "NOMBRE-DE-TU-PC",
    "ip": "192.168.1.XXX",
    "sharePath": "C:\\LAN-Share",
    "shareName": "LAN-Share"
}
```

En `remotePCs`, lista los demas equipos de la red:

```json
"remotePCs": [
    {
        "name": "NOMBRE-OTRO-PC",
        "ip": "192.168.1.YYY",
        "shareName": "LAN-Share",
        "driveLetter": "Z"
    }
]
```

> **Nota sobre driveLetter:** Cada PC remoto debe tener una letra diferente (Z, Y, X, W...).

### Paso 2: Ejecutar setup-smb-server.ps1 (como Administrador)

1. Click derecho en `setup-smb-server.ps1` -> "Ejecutar con PowerShell"
2. El script te pedira:
   - Contrasena para crear el usuario `LANShareUser` (si no existe)
   - Credenciales de los PCs remotos (opcional, puedes dejar en blanco y configurar luego)

**Que hace este script:**
- Deshabilita SMB1 (si aun estuviera activo)
- Habilita cifrado SMB3
- Crea la carpeta `C:\LAN-Share`
- Crea el usuario local `LANShareUser`
- Aplica permisos NTFS
- Comparte la carpeta via SMB
- Abre el firewall solo para redes privadas

### Paso 3: Ejecutar install-startup-task.ps1 (como Administrador)

1. Click derecho -> "Ejecutar con PowerShell"
2. Esto registra una tarea que ejecuta `connect-lan-shares.ps1` cada vez que inicias sesion.

### Paso 4: Probar la conexion manualmente

Abre PowerShell y ejecuta:

```powershell
.\connect-lan-shares.ps1
```

Si los otros PCs estan encendidos, deberias ver unidades de red nuevas (Z:, Y:, etc.) en el Explorador de Windows.

## Como usar despues

- **Copiar archivos:** Abre el Explorador -> "Este equipo" -> Unidades Z:, Y:, etc.
- **Ver log:** Abre `%TEMP%\LAN-Connect.log` para ver que paso al inicio.
- **Desconectar una unidad:** Click derecho en la unidad -> "Desconectar"
- **Reconectar:** Ejecuta `connect-lan-shares.ps1` o reinicia sesion. Con `/persistent:yes`, Windows intenta reconectar automaticamente tras reinicios.
- **Verificar configuracion:** Ejecuta `verify-lan-config.ps1` para comprobar que nombres e IPs cuadran.

## Seguridad incluida

- SMB1 permanentemente deshabilitado
- SMB3 con cifrado punto-a-punto activado
- Acceso sin cifrado rechazado
- Firewall: solo redes privadas
- Credenciales almacenadas en Windows Credential Manager (no en archivos de texto)
- Permisos NTFS restringidos a usuarios autenticados

## Solucion de problemas

### "Acceso denegado" al conectar a otro PC
Asegurate de que el usuario `LANShareUser` exista en AMBOS PCs con la **misma contrasena**. O bien, guarda las credenciales correctas con `cmdkey`:

```cmd
cmdkey /add:NOMBRE-PC /user:NOMBRE-PC\LANShareUser /pass:TuContrasena
```

### El script de inicio no mapea unidades
- Revisa el log en `%TEMP%\LAN-Connect.log`
- Verifica que el PC remoto responda a ping
- Asegurate de que la red este configurada como "Privada" (no Publica)

### El firewall bloquea SMB
El script `setup-smb-server.ps1` configura el firewall automaticamente, pero si usas un firewall de terceros, debes permitir:
- Puerto TCP 445 (SMB)
- Perfil: solo redes privadas

## Opciones de conexion remota (futuro)

Si en el futuro necesitas controlar otros PCs graficamente:

| Opcion | Como habilitar | Ideal para |
|---|---|---|
| **RDP** | Configuracion -> Sistema -> Escritorio remoto -> Habilitar | Control total del escritorio, nativo en Windows Pro |
| **OpenSSH** | `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0` | Terminal remoto, transferencias SCP/SFTP, muy seguro |
| **RustDesk** | Descargar desde rustdesk.com, servidor auto-hospedado en LAN | Soporte remoto visual moderno y open source |

---

**Recomendacion:** Manten este paquete (`LAN-Setup/`) en una USB o carpeta compartida. Para agregar un nuevo PC a la red, solo copias la carpeta, editas `config.json`, y ejecutas los dos scripts.
