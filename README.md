# 🔐 Lab 08 — DMVPN Fase 3 + IKEv2 + EIGRP

**Estudiante:** Enmanuel Feliz Soto | **Matrícula:** 2025-1402  
**Institución:** Instituto Tecnológico de Las Américas (ITLA)  
**Curso:** Seguridad en Redes | **Sección:** 2-1C  
**Docente:** Jonathan Esteban Rondón Corniel

---

## 📋 Descripción

DMVPN Fase 3 mejora Fase 2 con ip nhrp redirect en el Hub e ip nhrp shortcut en los Spokes, optimizando el routing para tráfico Spoke-to-Spoke.

| Campo | Valor |
|-------|-------|
| **Tipo de VPN** | DMVPN Hub-and-Spoke Optimizado |
| **Protocolo** | IKEv2 + IPSec ESP-AES256-SHA256 (mode transport) + mGRE |
| **Mecanismo** | NHRP redirect (HUB) + NHRP shortcut (Spokes) + IKEv2 |
| **Routing** | EIGRP AS 1403 sobre Tunnel0 mGRE |
| **Pre-shared Key** | `Cisco2025-1402-DMVPN3!` |

---

## 🗺️ Topología

> 📸 **[INSERTAR CAPTURA DE TOPOLOGÍA AQUÍ]**

<!-- Coloca aquí el screenshot de PNetLab con la topología del Lab 08 -->

**Entorno:** PNetLab — Cisco IOL  
**Peers:** HUB R1-S1 (20.25.8.2) | SPOKE1 R4-S2 (20.25.8.6) | SPOKE2 R5 (20.25.8.10)

### Tabla de Direccionamiento

| Rol | Router | IP WAN | IP Tunnel | LAN |
|-----|--------|--------|-----------|-----|
| HUB | R1-S1 | 20.25.8.2/30 | 14.0.3.1/24/24 | 10.14.17.0/24 (HUB) |
| SPOKE1 | R4-S2 | 20.25.8.6/30 | 14.0.3.2/24 | 10.14.27.0/24 (SPOKE1) |
| SPOKE2 | R5 | 20.25.8.10/30 | 14.0.3.3/24 | 10.14.37.0/25 VLAN10 + 10.14.37.128/25 VLAN20 (SPOKE2) |

### ISP

| Interfaz ISP | IP | Descripción |
|-------------|-----|-------------|
| Ethernet0/0 | 20.25.8.1/30 | Link to R1-S1 HUB |
| Ethernet0/1 | 20.25.8.5/30 | Link to R4-S2 SPOKE1 |
| Ethernet0/2 | 20.25.8.9/30 | Link to R5 SPOKE2 |

### Dirección Túnel
| Endpoint | IP Tunnel |
|----------|-----------|



---

## ⚙️ Configuración

El script completo de configuración se encuentra en:  
📄 [`Lab08_DMVPN_Fase3_IKEv2_EIGRP.txt`](./Lab08_DMVPN_Fase3_IKEv2_EIGRP.txt)

### Parámetros IKE/IPSec

| Parámetro | Valor |
|-----------|-------|
| Encryption | AES-256 |
| Hash/Integrity | SHA-256 |
| DH Group | 14 (2048-bit) |
| SA Lifetime (IKE) | 86400 s (24h) |
| SA Lifetime (IPSec) | 3600 s (1h) |
| PFS | Group 14 |
| Auth Method | Pre-Shared Key |

---

## ▶️ Procedimiento de Ejecución

### 1. Cargar configuración en PNetLab
```
# Aplicar configuración en cada dispositivo en el orden:
# 1. ISP → 2. R1-S1 → 3. R4-S2 → 4. R5
```

### 2. Verificar la VPN

```
show dmvpn
```
```
show ip nhrp
```
```
show ip eigrp neighbors
```
```
show crypto ikev2 sa
```

### 3. Prueba de conectividad
```
ping 10.14.37.1 source 10.14.27.1
```

---

## 📸 Capturas de Verificación

> 📸 **[INSERTAR CAPTURA: show crypto ikev2 sa]**

<!-- Captura mostrando el estado QM_IDLE / ESTABLISHED -->

> 📸 **[INSERTAR CAPTURA: show crypto ipsec sa]**

<!-- Captura mostrando pkts encaps/decaps incrementando -->

> 📸 **[INSERTAR CAPTURA: ping exitoso]**

<!-- Captura del ping source 10.14.17.0/24 (HUB) -->

---

## 🔍 Análisis y Comparativa

### Ventajas de este tipo de VPN
- Ver documentación técnica en el informe PDF

### Diferencias con otros labs
- Ver tabla comparativa en el README principal

---

## 📎 Recursos

| Recurso | Enlace |
|---------|--------|
| Repositorio Principal | [Enmafs/NetSec](https://github.com/Enmafs/NetSec) |
| Script de configuración | [`Lab08_DMVPN_Fase3_IKEv2_EIGRP.txt`](./Lab08_DMVPN_Fase3_IKEv2_EIGRP.txt) |
| Video demostración | 🎬 **[PENDIENTE — agregar link de YouTube]** |

---

> ⚠️ *Laboratorio realizado en entorno controlado (PNetLab). Fines exclusivamente académicos.*
