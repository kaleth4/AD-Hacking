
# 💀 Active Directory Post-Exploitation & Audit Framework

> **"The quietest click is often the loudest in the logs."**

---

## 🎯 Descripción General

Este repositorio es un **compendio técnico de élite** dedicado a vectores de ataque, desconfiguraciones críticas (misconfigurations) y vulnerabilidades legacy/modernas para la auditoría ofensiva de entornos Active Directory. Una guía sin filtros para profesionales de seguridad que juegan en serio.

**⚠️ Disclaimer:** Este contenido es exclusivamente para auditorías autorizadas y entornos de laboratorio. El acceso no autorizado es ilegal.

---

## 🚩 Initial Access & Recon (Enumeración de Superficie)

Antes de disparar, mapeamos el terreno. El objetivo es identificar vectores de entrada sin activar alarmas.

### Técnicas de Reconocimiento

*   **AS-REP Roasting:** Extracción de hashes de usuarios que no requieren pre-autenticación Kerberos (`DONT_REQ_PREAUTH`). Sin interacción = sin logs de intento fallido.
*   **Kerberoasting (TGS-REQ):** Solicitud de tickets de servicio para crackear hashes de cuentas de servicio (SPN). Foco en cuentas con privilegios elevados. Silencioso y efectivo.
*   **LLMNR/NBT-NS Poisoning:** Intercepción de tráfico de red para captura de hashes NTLMv2 (Responder/Inveigh). El clásico que nunca falla.

---

## ⚡ Escalada de Privilegios y Lateral Movement

Una vez dentro, buscamos el **"Camino más corto al Domain Admin"**. Aquí es donde el juego se pone interesante.

### 🆕 Hallazgos Recientes (Modern Era)

*   **ADCS (Active Directory Certificate Services):** Abuso de plantillas de certificados vulnerables (ESC1, ESC2, ESC3, ESC8/Relay). Es el nuevo estándar de oro para persistencia y escalada. **El futuro es ahora.**
*   **KDC Proxy & PetitPotam:** Técnicas de coerción de autenticación para forzar a controladores de dominio a autenticarse contra un atacante. Fuerza bruta de identidad.
*   **SamAccountName / NoPac (CVE-2021-42278):** Suplantación de identidad mediante la manipulación de atributos de cuenta de computadora para obtener un ticket de administrador. Elegancia pura.

### 📜 Hallazgos Legacy (Old but Gold)

*   **GPP (Group Policy Preferences):** Búsqueda de `Groups.xml` con contraseñas cifradas en AES (cuya clave es pública por Microsoft). Un regalo envenenado de Microsoft.
*   **Unconstrained Delegation:** Si una computadora tiene delegación sin restricciones, cualquier usuario que se conecte deja su TGT en memoria. Recolección automática de credenciales.
*   **Constrained Delegation (S4U2Self/S4U2Proxy):** Abuso de extensiones de Kerberos para suplantar usuarios en servicios específicos. Control quirúrgico.

---

## 🏗️ Compromiso del Bosque (Domain Dominance)

El fin del juego. **Control total de la base de datos `ntds.dit`.**

*   **DCSync Attack:** Simulación de un proceso de replicación para extraer todos los hashes de contraseñas del dominio (requiere privilegios de `Replicating Directory Changes`). Extracción en masa.
*   **Golden Ticket:** Forjado de un Ticket Granting Ticket (TGT) usando el hash de la cuenta `krbtgt`. **Persistencia total.** Acceso indefinido sin ser detectado.
*   **Silver Ticket:** Forjado de un Ticket de Servicio (TGS) para un servicio específico (ej. CIFS, LDAP) sin pasar por el KDC. Acceso quirúrgico a servicios específicos.

---

## 🛡️ Mitigación y Hardening (Defensive POV)

Para cada ataque, existe una contramedida. Un **elite hacker** debe conocerlas para evadirlas... o reportarlas.

### Contramedidas Críticas

1.  **Tiered Administration Model:** Aislar las credenciales de administrador de dominio de las estaciones de trabajo. Segregación de privilegios.
2.  **LAPS (Local Administrator Password Solution):** Rotación automática de contraseñas de admin local. Credenciales efímeras.
3.  **Protected Users Group:** Restringe protocolos débiles (NTLM, DES) y limita la delegación para cuentas sensibles. Blindaje de cuentas críticas.
4.  **PAC Validation:** Asegurar que los tickets de Kerberos sean validados rigurosamente por los servicios. Verificación de integridad.

---

## 🔬 Análisis Técnico Profundo

### 1. Kerberoasting (Abuso de Kerberos)

**Concepto:** Un usuario autenticado solicita un ticket de servicio (TGS) para cualquier servicio que tenga un Service Principal Name (SPN). El ticket está cifrado con el hash de la contraseña de la cuenta de servicio.

**Riesgo:** El atacante extrae el ticket y realiza un ataque de fuerza bruta offline para obtener la contraseña en texto plano.

**Defensa:** 
- Utilizar contraseñas largas (+25 caracteres) para cuentas de servicio
- Emplear **Group Managed Service Accounts (gMSA)**

---

### 2. Relay de NTLM

**Concepto:** Si un atacante intercepta una solicitud de autenticación NTLMv2 (mediante técnicas como LLMNR/NBNS spoofing), puede "retransmitir" ese hash a otro servidor que no tenga activada la firma SMB (SMB Signing).

**Defensa:** 
- Habilitar **SMB Signing** (obligatorio)
- Desactivar protocolos heredados como LLMNR y NetBIOS

---

### 3. Abuso de Delegación (Unconstrained/Constrained Delegation)

**Concepto:** Si una computadora o cuenta de servicio tiene habilitada la "delegación sin restricciones", cualquier usuario que se autentique contra ella deja una copia de su TGT (Ticket Granting Ticket) en memoria, permitiendo al servidor suplantar al usuario ante cualquier otro servicio del dominio.

**Defensa:** 
- Configurar "Constrained Delegation" (Delegación restringida)
- Marcar cuentas sensibles (como Domain Admins) como "Account is sensitive and cannot be delegated"

---

## 🛠️ Herramientas y Metodologías de Auditoría

Si estás aprendiendo sobre seguridad en AD, estas herramientas son **imprescindibles** en entornos controlados:

| Herramienta | Propósito |
|---|---|
| **BloodHound** | Visualizar rutas de ataque y relaciones de confianza complejas |
| **Mimikatz** | Entender cómo se gestionan los secretos en la memoria de Windows (LSASS) |
| **Impacket** | Colección de clases Python para trabajar con protocolos de red de Windows |
| **Responder** | Poisoning de LLMNR/NBNS para captura de hashes |
| **Certipy** | Enumeración y explotación de ADCS |

---

## 🎓 Plataformas de Práctica Legal

Para entrenar de forma segura y legal:

*   **Hack The Box (Pro Labs: Dante/Zephyr)** - Laboratorios realistas de AD
*   **TryHackMe (Holocron/Wreath)** - Caminos guiados para aprender AD
*   **Detection Lab (Splunk/ELK)** - Configura tu propio lab para ver qué trazas dejan estos ataques

---

## ⚖️ Recomendación Ética

> **Nunca realices pruebas en redes de las que no tengas una autorización escrita (RoE - Rules of Engagement). El acceso no autorizado es ilegal.**

Este repositorio es una herramienta educativa. Su uso debe ser siempre dentro del marco legal y ético.

---

## 📚 Estructura del Repositorio

```
📦 AD-PostExploitation-Framework
├── 📁 recon/
│   ├── as-rep-roasting.md
│   ├── kerberoasting.md
│   └── llmnr-poisoning.md
├── 📁 escalation/
│   ├── adcs-esc1-esc8.md
│   ├── petitpotam.md
│   ├── nopac.md
│   ├── gpp-abuse.md
│   └── delegation-abuse.md
├── 📁 dominance/
│   ├── dcsync.md
│   ├── golden-ticket.md
│   └── silver-ticket.md
├── 📁 defense/
│   ├── hardening-guide.md
│   ├── detection-rules.md
│   └── incident-response.md
└── 📁 tools/
    ├── scripts/
    └── payloads/
```


## 👤 Autor
KALETH CORCHO

**Elite Security Researcher** | Active Directory Specialist

---

> **"En la seguridad, el conocimiento es poder. Pero la responsabilidad es la verdadera medida de un profesional."**

```
Active Directory
Envenenador de tráfico DNS, SQL, SMB
SMB Relay
Responder.py -I eth0 -rdw
Cracking de Hashes
Guardar el hash en un archivo (ej. hash.txt).
Comando para John the Ripper:
john --wordlist=/usr/share/wordlists/rockyou.txt hash
Enumeración de Red
Enumerar máquinas con SMB en el segmento:
crackmapexec smb 192.168.80.0/24
Ataque Relay
ntlmrelayx.py -tf target.txt -smb2support
Objetivo: Envenenar SMB con un equipo de bajos privilegios.
