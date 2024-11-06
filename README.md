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
3. Usa herramientas como `dig` o `nslookup` para verificar la configuración del DNS.
