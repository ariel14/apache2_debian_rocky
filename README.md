# apache2_debian_rocky
# 🌐 Guía Completa: Servicios Web con Apache HTTP Server

Esta es tu guía completa para aprender Apache HTTP Server desde cero hasta nivel avanzado.

## 📚 ESTRUCTURA DE LA GUÍA

La guía completa contiene 8 módulos:

### ✅ MÓDULO 1: Fundamentos (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Introducción a servicios web
- Historia de Apache
- Instalación en Debian y Rocky

### ✅ MÓDULO 2: Configuración Básica (COMPLETADO EN EL ARCHIVO ANTERIOR)  
- Estructura de directorios
- Archivos de configuración
- Directivas esenciales

### ✅ MÓDULO 3: Gestión del Servicio (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Control con systemd
- Primeras páginas web
- Despliegue desde GitHub

### ✅ MÓDULO 4: Virtual Hosts (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Conceptos y tipos
- Configuración multi-sitio
- Gestión de hosts virtuales

### ✅ MÓDULO 5: Seguridad HTTPS/SSL (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Fundamentos de criptografía
- Certificados autofirmados
- Configuración SSL/TLS

### ✅ MÓDULO 6: Let's Encrypt (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Instalación de Certbot
- Métodos standalone y plugin
- Renovación automática

### ✅ MÓDULO 7: Optimización (COMPLETADO EN EL ARCHIVO ANTERIOR)
- Logs y monitoreo
- Compresión y caché
- Mejores prácticas

### 🎯 MÓDULO 8: PROYECTO FINAL INTEGRADOR

# 🎯 Proyecto Final: Infraestructura Web Institucional Completa

## 🏫 Escenario Real

Implementarás la infraestructura web completa para una institución educativa con:

1. **Portal Público** (puerto 443 - HTTPS)
2. **Intranet Administrativa** (puerto 8080)
3. **Sistema de Alumnos** (puerto 8081)  
4. **Panel de Administración** (puerto 8082 - con autenticación)

## 📋 Arquitectura del Sistema

```
INTERNET → Firewall → Apache Server
                         ├── Portal (443/HTTPS)
                         ├── Intranet (8080/HTTP)
                         ├── Alumnos (8081/HTTP)
                         └── Admin (8082/HTTP+Auth)
```

## 🛠️ IMPLEMENTACIÓN PASO A PASO

### FASE 1: Preparación del Servidor

#### En Rocky Linux:
```bash
sudo dnf update -y
sudo dnf install -y httpd mod_ssl vim wget curl git
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --permanent --add-port={8080,8081,8082}/tcp
sudo firewall-cmd --reload
```

#### En Debian:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 vim wget curl git
sudo systemctl enable --now apache2
sudo ufw allow 80,443,8080,8081,8082/tcp
```

### FASE 2: Crear Estructura de Directorios

```bash
sudo mkdir -p /var/www/{portal,intranet,alumnos,admin}
sudo chown -R apache:apache /var/www/  # Rocky
#
sudo chown -R www-data:www-data /var/www/  # Debian
sudo find /var/www/ -type d -exec chmod 755 {} \;
sudo find /var/www/ -type f -exec chmod 644 {} \;
```

### FASE 3: Desplegar Contenido Web

#### Clonar sitio desde GitHub:
```bash
cd /tmp
git clone https://github.com/SGarcia710/one-page-portfolio-template.git
sudo cp -r one-page-portfolio-template/* /var/www/portal/
```

### FASE 4: Configurar Virtual Hosts

#### Portal Público (HTTPS):
```apache
<VirtualHost *:443>
    ServerName portal.local
    DocumentRoot /var/www/portal
    
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/servidor.crt
    SSLCertificateKeyFile /etc/pki/tls/private/servidor.key
    
    <Directory /var/www/portal>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

#### Intranet (Puerto 8080):
```apache
<VirtualHost *:8080>
    ServerName intranet.local
    DocumentRoot /var/www/intranet
    
    <Directory /var/www/intranet>
        Require ip 192.168.0.0/16 10.0.0.0/8 127.0.0.1
    </Directory>
</VirtualHost>
```

#### Sistema Alumnos (Puerto 8081):
```apache
<VirtualHost *:8081>
    ServerName alumnos.local
    DocumentRoot /var/www/alumnos
</VirtualHost>
```

#### Panel Admin (Puerto 8082 + Auth):
```apache
<VirtualHost *:8082>
    ServerName admin.local
    DocumentRoot /var/www/admin
    
    <Directory /var/www/admin>
        AuthType Basic
        AuthName "Panel Administración"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

### FASE 5: Generar Certificados SSL

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/servidor.key \
  -out /etc/pki/tls/certs/servidor.crt \
  -subj "/C=BO/ST=LaPaz/L=LaPaz/O=Institucion/CN=portal.local"
```

### FASE 6: Configurar Autenticación

```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
sudo htpasswd /etc/apache2/.htpasswd supervisor
```

### FASE 7: Optimización

Crear `/etc/httpd/conf.d/optimization.conf`:

```apache
# Compresión
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css text/javascript
</IfModule>

# Caché
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
</IfModule>

# Seguridad
ServerTokens Prod
ServerSignature Off
```

### FASE 8: Validación y Pruebas

```bash
# Validar sintaxis
sudo httpd -t

# Ver Virtual Hosts
sudo httpd -S

# Reiniciar Apache
sudo systemctl restart httpd

# Probar cada sitio
curl -k https://localhost/
curl http://localhost:8080/
curl http://localhost:8081/
curl -u admin:password http://localhost:8082/

# Ver logs
sudo tail -f /var/log/httpd/*-access.log
```

## 📊 ENTREGABLES DEL PROYECTO

### 1. Sistema Funcional
- ✅ 4 sitios web funcionando
- ✅ HTTPS en portal público
- ✅ Autenticación en panel admin

### 2. Documentación
- ✅ Archivo ARQUITECTURA.md
- ✅ Archivo TROUBLESHOOTING.md
- ✅ Capturas de pantalla

### 3. Configuraciones
- ✅ Archivos .conf de cada VirtualHost
- ✅ Certificado SSL
- ✅ Archivo .htpasswd

## 🎯 CHECKLIST DE EVALUACIÓN

| Item | Puntos | Estado |
|------|--------|--------|
| Apache instalado | 15 | [ ] |
| 4 Virtual Hosts | 25 | [ ] |
| HTTPS configurado | 20 | [ ] |
| Autenticación | 15 | [ ] |
| Optimización | 10 | [ ] |
| Logs separados | 5 | [ ] |
| Documentación | 10 | [ ] |
| **TOTAL** | **100** | |

## 🔧 TROUBLESHOOTING RÁPIDO

### Error 403 Forbidden
```bash
sudo chown -R apache:apache /var/www/
sudo chmod -R 755 /var/www/
sudo restorecon -Rv /var/www/  # SELinux
```

### Puerto no accesible
```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
sudo semanage port -a -t http_port_t -p tcp 8080
```

### Certificado inválido
Es normal con certificados autofirmados. En producción usar Let's Encrypt.

### Autenticación no funciona
```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
sudo chmod 644 /etc/apache2/.htpasswd
```

## 📚 RECURSOS ADICIONALES

- **Documentación Apache:** https://httpd.apache.org/docs/2.4/
- **Let's Encrypt:** https://letsencrypt.org/
- **SSL Labs Test:** https://www.ssllabs.com/ssltest/

## 🏆 CONCLUSIÓN

¡Felicitaciones! Has completado la implementación de una infraestructura web profesional.

**Habilidades adquiridas:**
- ✅ Instalación y configuración de Apache
- ✅ Virtual Hosts multi-sitio
- ✅ Implementación HTTPS/SSL
- ✅ Control de acceso y autenticación
- ✅ Optimización de performance
- ✅ Monitoreo y troubleshooting

**Próximos pasos:**
1. Implementar Let's Encrypt en producción
2. Configurar mod_proxy para reverse proxy
3. Balanceo de carga con mod_proxy_balancer
4. Contenerización con Docker
5. Automatización con Ansible

---

## 🙏 CRÉDITOS

**Guía creada para:** Curso de Administración de Servidores Linux
**Fecha:** Noviembre 2025
**Versión:** 1.0

Esta guía cubre Apache HTTP Server 2.4.x en distribuciones Debian/Ubuntu y Rocky Linux/RHEL.

---

# 🎉 ¡Éxito en tu carrera como Administrador de Sistemas! 🐧
