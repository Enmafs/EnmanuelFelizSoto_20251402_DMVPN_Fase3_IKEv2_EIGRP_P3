# 🔐 Lab 08 — DMVPN Fase 3 + IKEv2 + EIGRP
**Estudiante:** Enmanuel Feliz Soto | **Matrícula:** 2025-1402  
**Institución:** Instituto Tecnológico de Las Américas (ITLA)  
**Curso:** Seguridad en Redes | **Sección:** 5  
**Docente:** Jonathan Esteban Rondón Corniel

---

## 📋 Descripción

DMVPN Fase 3 mejora Fase 2 con `ip nhrp redirect` en el Hub e `ip nhrp shortcut` en los Spokes. El Hub solo participa en la señalización NHRP; el tráfico Spoke-to-Spoke va directo via CEF shortcut sin pasar por el Hub. IKEv2 reemplaza a IKEv1 con negociación más eficiente y menor número de mensajes.

| Campo | Valor |
|-------|-------|
| **Tipo de VPN** | DMVPN Hub-and-Spoke Optimizado |
| **Protocolo** | IKEv2 + IPSec ESP-AES256-SHA256 (mode transport) + mGRE |
| **Mecanismo** | NHRP redirect (Hub) + NHRP shortcut (Spokes) + IKEv2 |
| **Routing** | EIGRP AS 100 sobre Tunnel0 mGRE |
| **Pre-shared Key** | `Cisco123` |

---

## 🗺️ Topología

> 📸 <img width="1472" height="805" alt="Captura de pantalla 2026-06-21 154657" src="https://github.com/user-attachments/assets/be058c52-a43b-4421-b27e-db18979ca4c7" />

<!-- Coloca aquí el screenshot de PNetLab con la topología del Lab 08 -->

**Entorno:** PNetLab — Cisco IOL  
**Peers:** HUB ISP (14.2.10.1) | SPOKE1 R1-S1 (14.2.10.2) | SPOKE2 R3 (14.2.10.3)

### Tabla de Direccionamiento

| Rol | Router | Interfaz WAN | IP WAN | IP Tunnel | LAN |
|-----|--------|-------------|--------|-----------|-----|
| HUB | ISP | e0/0 | 20.25.1.1/30 | 14.2.10.1/24 | — |
| SPOKE1 | R1-S1 | e0/0 | 20.25.1.2/30 | 14.2.10.2/24 | — |
| SPOKE2 | R3 | e0/0 | 20.25.2.6/30 | 14.2.10.3/24 | 30.30.30.0/24 |

### ISP (Hub)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.1.1/30 | Link to R1-S1 (Spoke1) y transit hacia R3 |
| Tunnel0 | 14.2.10.1/24 | mGRE Hub DMVPN |

### R1-S1 (Spoke1 + Transit para R3)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.1.2/30 | Link to ISP Hub |
| Ethernet0/2 | 20.25.2.5/30 | Link to R3 (transit) |
| Tunnel0 | 14.2.10.2/24 | GRE Spoke1 DMVPN |

### R3 (Spoke2)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.2.6/30 | Link to R1-S1 (transit hacia ISP) |
| Ethernet0/1 | 30.30.30.1/24 | LAN local |
| Tunnel0 | 14.2.10.3/24 | GRE Spoke2 DMVPN |

### Dirección Túnel

| Endpoint | IP Tunnel | NBMA (física) |
|----------|-----------|---------------|
| ISP (Hub) | 14.2.10.1 | 20.25.1.1 |
| R1-S1 (Spoke1) | 14.2.10.2 | 20.25.1.2 |
| R3 (Spoke2) | 14.2.10.3 | 20.25.2.6 |

> **Nota:** R3 llega al Hub ISP pasando por R1-S1 como router de tránsito. El tunnel source de R3 es `e0/0` (20.25.2.6); NHRP registra esa IP como NBMA ante el Hub.

---

## ⚙️ Configuración

El script completo de configuración se encuentra en:  
📄 [`EnmanuelFelizSoto_2025-1402_Lab08_P3.txt`](./EnmanuelFelizSoto_2025-1402_Lab08_P3.txt)

### Parámetros IKEv2/IPSec

| Parámetro | Valor |
|-----------|-------|
| Encryption | AES-256 |
| Hash/Integrity | SHA-256 |
| DH Group | 14 (2048-bit) |
| SA Lifetime (IKE) | 86400 s (24h) |
| Auth Method | Pre-Shared Key |
| IPSec Mode | Transport |
| Transform-set | esp-aes 256 esp-sha256-hmac |
| IKEv2 Proposal | PROP_IKEv2 |
| IKEv2 Keyring | KEYRING_DMVPN |
| IKEv2 Profile | PROF_IKEv2 |

### Parámetros NHRP

| Parámetro | Valor |
|-----------|-------|
| Network-ID | 100 |
| Authentication | NHRP2024 |
| Holdtime | 600 s |
| NHS (Hub) | 14.2.10.1 |
| Tunnel mode | gre multipoint |
| Hub | `ip nhrp redirect` |
| Spokes | `ip nhrp shortcut` |

---

## ▶️ Procedimiento de Ejecución

### 1. Cargar configuración en PNetLab

```
# Aplicar configuración en cada dispositivo en este orden:
# 1. ISP (Hub) → 2. R1-S1 (Spoke1) → 3. R3 (Spoke2)
```

### 2. Verificar la VPN

```
show dmvpn
```
```
show dmvpn detail
```
```
show ip nhrp
```
```
show ip nhrp detail
```
```
show ip eigrp neighbors
```
```
show crypto ikev2 sa
```
```
show crypto ipsec sa
```

### 3. Prueba de conectividad

```
ping 14.2.10.1 source Tunnel0
```
```
ping 14.2.10.3 source Tunnel0
```
```
ping 30.30.30.1
```

### 4. Verificar shortcut Fase 3 (diferenciador clave)

```
show ip nhrp detail
```
```
show ip cef 30.30.30.0
```

> El flag `shortcut` en `show ip nhrp detail` y la entrada CEF directa confirman que el tráfico Spoke-to-Spoke no pasa por el Hub.

---

## 📸 Capturas de Verificación

> 📸 <img width="561" height="249" alt="image" src="https://github.com/user-attachments/assets/da0ed2bc-ea83-458e-a0e0-6e0e6e62d41b" />

<!-- Captura mostrando spokes registrados con estado UP -->

> 📸 <img width="794" height="215" alt="image" src="https://github.com/user-attachments/assets/793fe9dc-6562-419e-acd2-5f80442a8a5d" />

<!-- Captura mostrando pkts encaps/decaps incrementando -->

> 📸 <img width="388" height="129" alt="image" src="https://github.com/user-attachments/assets/9fbd685a-a046-40d5-9085-e8f68739663b" />

<!-- Captura mostrando el shortcut instalado en CEF -->

> 📸 <img width="471" height="87" alt="image" src="https://github.com/user-attachments/assets/e48f1f91-5823-48a3-82ba-c1f1459978ff" />

<!-- Ping desde R1-S1 hacia 30.30.30.1 (LAN de R3) -->

---

## 🔍 Análisis y Comparativa

### Diferencia clave Fase 2 vs Fase 3

| Característica | Fase 2 | Fase 3 |
|----------------|--------|--------|
| Hub en path de datos | Sí (inicial) | No — solo señaliza |
| Comando Hub | — | `ip nhrp redirect` |
| Comando Spokes | — | `ip nhrp shortcut` |
| IKE | IKEv1 (`crypto isakmp`) | IKEv2 (`crypto ikev2`) |
| CEF shortcut | No | Sí |
| Escalabilidad | Media | Alta |

### Ventajas de este tipo de VPN
- Ver documentación técnica en el informe PDF

### Diferencias con otros labs
- Ver tabla comparativa en el README principal

---

## 📎 Recursos

| Recurso | Enlace |
|---------|--------|
| Repositorio Principal | [Enmafs/NetSec](https://github.com/Enmafs/NetSec) |
| Script de configuración | [`EnmanuelFelizSoto_2025-1402_DMVPN_Fase3_IKEv2_EIGRP_P3.txt`](./EnmanuelFelizSoto_2025-1402_DMVPN_Fase3_IKEv2_EIGRP_P3.txt) |
| Video demostración | 🎬 [Aquí](https://youtu.be/qR8-BsUhoFk) |

---

> ⚠️ *Laboratorio realizado en entorno controlado (PNetLab). Fines exclusivamente académicos.*
