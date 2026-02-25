# Laboratorio de Pentesting Active Directory – MARVEL-SHIELD.local

> ⚠️ **Aviso importante**
> Este laboratorio fue ejecutado en un entorno **controlado** con fines **educativos y defensivos**. No se realizaron pruebas sobre sistemas productivos.

---

##  Descripción general

Este proyecto documenta un **laboratorio de pentesting en Active Directory** diseñado para simular un **escenario realista de ataque interno** en una red corporativa.

El objetivo principal **no es únicamente la explotación**, sino el **análisis del impacto post-compromiso, el riesgo operativo y las mitigaciones defensivas**, alineadas a buenas prácticas empresariales.

**Características del entorno:**

* **Dominio:** `MARVEL-SHIELD.local`
* **Controlador de Dominio (DC):** Windows Server 2022
* **Tipo de escenario:** Pentest interno (same network segment)

---

## 🧩 Topología del laboratorio

El entorno fue desplegado deliberadamente en el **mismo segmento de red**, reflejando una condición común en muchas organizaciones donde no existe una segmentación interna adecuada.

```mermaid
flowchart LR
    subgraph Network["Red Interna (Mismo Segmento)"]
        DC["🖥️ Controlador de Dominio\nWindows Server 2022\nMARVEL-SHIELD.local"]
        SP["💻 SPIDERMAN\nEquipo unido al dominio"]
        IR["💻 IRONMAN\nEquipo unido al dominio"]
        KALI["🐉 Kali Linux\nMáquina atacante"]
    end

    DC --- SP
    DC --- IR
    SP --- DC
    IR --- DC

    KALI --- DC
    KALI --- SP
    KALI --- IR
```

---

##  Flujo del ataque

El siguiente diagrama representa la **cadena completa del ataque**, desde el acceso inicial hasta la enumeración post-compromiso del dominio.

```mermaid
sequenceDiagram
    participant SP as SPIDERMAN (Usuario de dominio)
    participant IR as IRONMAN (Usuario de dominio)
    participant DC as DC (MARVEL-SHIELD.local)
    participant KALI as Kali Linux (Atacante)

    SP->>DC: Solicitud DNS (recurso mal escrito)
    DC-->>SP: Falla en la resolución DNS

    SP->>KALI: Broadcast LLMNR / NBT-NS
    KALI-->>SP: Respuesta falsificada (Responder)

    SP->>KALI: Autenticación NTLMv2
    KALI->>KALI: Captura del hash NTLMv2

    KALI->>KALI: Crackeo del hash (Hashcat)
    KALI->>DC: Autenticación válida al dominio

    KALI->>DC: Enumeración LDAP (ldapdomaindump)
    KALI->>DC: Recolección de datos con BloodHound

    KALI->>KALI: Identificación de rutas de escalamiento de privilegios
```

---

##  Herramientas y técnicas utilizadas

### Acceso inicial

* Envenenamiento LLMNR / NBT-NS
* Responder

### Ataques a credenciales

* Captura de hashes NTLMv2
* Ataques de diccionario con Hashcat

### Movimiento lateral y enumeración

* Autenticación SMB
* Enumeración LDAP (`ldapdomaindump`)
* Análisis de rutas de ataque con BloodHound

---

##  Estrategia de mitigación (Defense-in-Depth)

El siguiente diagrama mapea **controles defensivos** directamente contra cada fase del ataque identificado en el laboratorio.

```mermaid
flowchart TB
    subgraph Phase1["🧪 Acceso inicial – Abuso de resolución de nombres"]
        LLMNR["Deshabilitar LLMNR"]
        NBTNS["Deshabilitar NBT-NS"]
        DNS["Forzar configuración correcta de DNS"]
    end

    subgraph Phase2["🔑 Protección de credenciales"]
        PASS["Políticas de contraseña fuertes (>14 caracteres)"]
        SMB_SIGN["Forzar SMB Signing"]
        KERB["Priorizar Kerberos sobre NTLM"]
    end

    subgraph Phase3["🚧 Control de movimiento lateral"]
        SEG["Segmentación de red (VLANs)"]
        ADMIN["Limitar privilegios de administrador local"]
        NTLM["Restringir autenticación NTLM"]
    end

    subgraph Phase4["🔍 Detección y monitoreo"]
        LOG["Centralización de logs"]
        AUTH["Alertas por autenticación NTLM"]
        LDAP["Monitoreo de consultas LDAP anómalas"]
    end

    subgraph Phase5["🛠️ Resiliencia post-compromiso"]
        ROTATE["Rotación de contraseña de krbtgt"]
        AUDIT["Auditorías periódicas de Active Directory"]
        HARD["Endurecimiento continuo de AD"]
    end

    Phase1 --> Phase2
    Phase2 --> Phase3
    Phase3 --> Phase4
    Phase4 --> Phase5
```

---

##  Conclusiones clave

* Los compromisos de Active Directory **raramente dependen de CVEs**, sino de **mal diseño, configuraciones inseguras y confianza excesiva**.
* Obtener privilegios elevados **no es el objetivo final**; el valor real está en comprender el **impacto operativo y de negocio**.
* La seguridad efectiva en Active Directory requiere **controles en capas**, no soluciones aisladas.

---

##  Valor del laboratorio

Este proyecto demuestra:

* Simulación de escenarios realistas de ataque interno
* Capacidad de análisis post-compromiso
* Pensamiento ofensivo y defensivo combinado
* Habilidad para traducir hallazgos técnicos en **controles de seguridad**

---

##  Notas finales

Este laboratorio está orientado a:

* Aprendizaje en ciberseguridad
* Demostración de portafolio profesional
* Discusión técnica en entrevistas

No se afectaron sistemas productivos.
