# Configuración de WireGuard en Sistemas Basados en Debian

Este documento detalla los pasos necesarios para instalar y configurar una VPN punto a punto con WireGuard en sistemas Debian o basados en Debian.

## Requisitos previos

- Sistema operativo Debian, Ubuntu u otro basado en Debian.
- Acceso a root o privilegios de `sudo`.
- Conexión a internet para descargar paquetes.

---

## Instalación de WireGuard

```bash
sudo apt update
sudo apt install wireguard
```
En caso de querer usar un cliente windows o de alguna otra distribucion de linux, referirse a las descargas en el [sitio oficial de wireguard](https://www.wireguard.com/install/)

---

## Generar llaves pública y privada

Ejecuta el siguiente comando para crear una clave privada y su correspondiente clave pública:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

Esto generará dos archivos en el directorio actual:

* `privatekey`
* `publickey`

---

## Configuración del servidor WireGuard

Crea el archivo `/etc/wireguard/wg0.conf` en el servidor con el siguiente contenido:

```ini
[Interface]
PrivateKey = <server-private-key>
Address = <server-ip-address>/<subnet>
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o <public-interface> -j MASQUERADE;
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o <public-interface> -j MASQUERADE;
ListenPort = 51820
```

* Reemplaza `<server-private-key>` con la clave privada del servidor.
* Reemplaza `<server-ip-address>/<subnet>` con una IP interna (ej. `10.0.0.1/24`).
* Reemplaza `<public-interface>` con tu interfaz de red pública (ej. `eth0` o `ens3`).

---

## Configuración del cliente

En el cliente, crea el archivo `/etc/wireguard/wg0.conf` con el siguiente contenido:

```ini
[Interface]
PrivateKey = <client-private-key>
Address = <client-ip-address>/<subnet>
SaveConfig = true

[Peer]
PublicKey = <server-public-key>
Endpoint = <server-public-ip-address>:51820
AllowedIPs = 0.0.0.0/0
```

* Reemplaza `<client-private-key>` con la clave privada del cliente.
* Reemplaza `<client-ip-address>/<subnet>` con una IP interna (ej. `10.0.0.10/32`).
* Reemplaza `<server-public-key>` con la clave pública del servidor.
* Reemplaza `<server-public-ip-address>` con la IP pública del servidor.

---

## Levantar interfaces WireGuard

Tanto en el servidor como en los clientes, puedes levantar la interfaz con:

```bash
wg-quick up wg0
```

Para detenerla:

```bash
wg-quick down wg0
```

---

## Agregar un cliente (peer) desde el servidor

Puedes registrar un nuevo cliente desde el servidor con:

```bash
sudo wg set wg0 peer <client-public-key> allowed-ips <client-ip-address>/32
```
> Es importante el uso del `/32` ya que de esta forma se entiende que la ip es unica del cliente
---

## Verificar estado de la VPN

En cualquier nodo con WireGuard activo, usa el siguiente comando para ver el estado actual de la conexión:

```bash
sudo wg
```

---

## Notas adicionales

* Asegúrate de que el puerto UDP `51820` esté abierto en el firewall del servidor.
* Es recomendable habilitar el reenvío de IP en el servidor editando `/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward=1
```

Y luego aplicar con:

```bash
sudo sysctl -p
```

---

¡Listo! Ahora tu red privada con WireGuard está activa y funcional.

