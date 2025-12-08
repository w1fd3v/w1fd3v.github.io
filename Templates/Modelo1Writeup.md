# 🖥️ Máquina: {{title}}
- **Plataforma:** HackTheBox
- **Dificultad:** 
- **Tipo:** (Linux / Windows)
- **Fecha de resolución:** 
- **IP:** 
- **OSCP-like:** Sí / No

---

## 🎯 Objetivo
> Explica brevemente cuál es el objetivo: obtener user.txt y root.txt, practicar una técnica concreta, aprender algún servicio, etc.

---

# 🔎 1. Reconocimiento inicial

## 1.1 Ping / Comprobación de host


> ¿El host responde? ¿Hay latencia sospechosa? ¿Filtra ICMP?

---

## 1.2 Escaneo de puertos (Nmap)
### Comando utilizado:


### Resultados relevantes:
- Puerto X → Servicio
- Puerto Y → Servicio
- Observaciones:

> Reflexión: ¿Qué servicios parecen más interesantes? ¿Cuál parece más vulnerable según versión?

---

# 🌐 2. Enumeración de servicios

## 2.1 Servicio web (si aplica)
- URL: 
- Tecnologías detectadas:
- Rutas encontradas:

### Enumeración (Gobuster/FFUF)

gobuster dir -u http://{{IP}} -w wordlist.txt -x php,txt,html


> Preguntas para ti:  
> - ¿El sitio tiene login?  
> - ¿Hay uploads?  
> - ¿Muestra versiones?  
> - ¿Hay endpoints internos sospechosos?

---

## 2.2 Otros servicios
### FTP / SSH / SMB / RDP / etc.
> Describe:
- Versiones
- Posibles vectores (fuerza bruta, misconfiguración, info leakage, etc.)

---

# 🛠️ 3. Explotación

## 3.1 Vector explotado
> Explica **por qué** este vector es vulnerable.  
Ejemplo: “El formulario no filtra input → SQLi”, “Servicio desactualizado con CVE”, etc.

### Pruebas realizadas
