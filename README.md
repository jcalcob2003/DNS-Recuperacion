# Práctica de Configuración de Servidores DNS Maestro-Esclavo

Este repositorio contiene los archivos necesarios para configurar servidores DNS en modo maestro-esclavo usando Vagrant, BIND y Ubuntu.

## Estructura del Proyecto

- `Vagrantfile`: Configura dos máquinas virtuales en red privada con IPs específicas.
- `zones/`: Contiene los archivos de configuración de zonas DNS para la resolución directa e inversa.
- `named.conf.options` y `named.conf.local`: Configuración de opciones de red y declaración de zonas en BIND.
- `pruebas.md`: Evidencias de las consultas de resolución de DNS desde el servidor maestro y el servidor esclavo.

## Instrucciones de Uso

1. Clona este repositorio y navega a la carpeta.
2. Ejecuta `vagrant up` para iniciar las máquinas virtuales.
3. Usa herramientas como `dig` o `nslookup` para verificar la configuración del DNS.]

# 1. Configuración de BIND en DNSA (Maestro)

## Paso 1: Configuración de `/etc/bind/named.conf`.options
Haz una copia de seguridad del archivo:

```bash
sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
```
Configura IPv4 y acceso en /etc/bind/named.conf.options:

```bash
sudo nano /etc/bind/named.conf.options
```
Agrega la lista de acceso y opciones específicas:

```plaintext
acl confiables {
    192.168.57.0/24;
};

options {
    directory "/var/cache/bind";
    allow-transfer { none; };
    listen-on port 53 { 192.168.57.10; };
    recursion yes;
    allow-recursion { confiables; };
    dnssec-enable yes;
    dnssec-validation yes;
};
```
Verifica la configuración:

```bash
sudo named-checkconf /etc/bind/named.conf.options
```
Reinicia BIND:

```bash
sudo systemctl restart bind9
```

## Paso 2: Configuración de zonas en ```/etc/bind/named.conf.local```

Edita el archivo ```/etc/bind/named.conf.local``` para definir las zonas ies.test y sus subdominios como maestro:

```plaintext
zone "ies.test" {
    type master;
    file "/etc/bind/zones/db.ies.test";
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.57";
};
```
Crea los archivos de zona en ```/etc/bind/zones/```. Si no existe, crea el directorio:

```bash
sudo mkdir /etc/bind/zones
```
Archivo de zona directa ```/etc/bind/zones/db.ies.test```:

```plaintext
$TTL    604800
@       IN      SOA     dnsa.ies.test. root.ies.test. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

        IN      NS      dnsa.ies.test.

server01      IN      A       192.168.57.10
pcprofesor    IN      A       192.168.57.20
```
Archivo de zona inversa ```/etc/bind/zones/db.192.168.57```:

```plaintext
$TTL    604800
@       IN      SOA     dnsa.ies.test. root.ies.test. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

        IN      NS      dnsa.ies.test.

10      IN      PTR     server01.ies.test.
20      IN      PTR     pcprofesor.ies.test.
```
Verifica los archivos de zona:

```bash
sudo named-checkzone ies.test /etc/bind/zones/db.ies.test
sudo named-checkzone 57.168.192.in-addr.arpa /etc/bind/zones/db.192.168.57
```
Reinicia BIND:

```bash
sudo systemctl restart bind9
```
# 2. Configuración de BIND en DNSB (Esclavo)
Configura DNSB como esclavo en `/etc/bind/named.conf.local`:

```plaintext
zone "ies.test" {
    type slave;
    file "/var/cache/bind/db.ies.test";
    masters { 192.168.57.10; };
};

zone "57.168.192.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.192.168.57";
    masters { 192.168.57.10; };
};
```
Reinicia BIND en DNSB:

```bash
sudo systemctl restart bind9
```
# Verificación de Resoluciones de DNS
## Prueba en DNSA

Usa dig y nslookup para verificar en DNSA:

```bash
dig @192.168.57.10 server01.ies.test
nslookup pcprofesor.ies.test 192.168.57.10
```
Verifica la resolución inversa:

```bash
dig -x 192.168.57.10 @192.168.57.10
```
## Prueba en DNSB
Verifica que DNSB responde correctamente como esclavo:

```bash
dig @192.168.57.11 server01.ies.test
nslookup pcprofesor.ies.test 192.168.57.11
```
Confirma que la transferencia de zona funciona:

```bash
tail -f /var/log/syslog | grep named
```
