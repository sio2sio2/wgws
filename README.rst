wgws
****
Es un *script* en Bourne shell (ejecutable con dash_) que facilita la creación
de túneles VPN tal como hace `wg-quick`_ y usa para su configuración las mismas
variables que éste (*Address*, *Table*, *PreUp*, etc.). Permite, además,
encapsular el túnel VPN a través de websockets para burlar restricciones en
redes gestionadas por administradores excesivamente celosos.

Requisitos
==========
* nftables_.
* wstunnel_, instalado en algún directorio del *PATH*.
* wireguard_.

Configuración
=============
Basta con añadir la sección ``[Tunel]`` al fichero de configuración de la
interfaz que puede añadir la definición de tres variables:

========= ============= =================================================================
Variable   Valor         Descripción
========= ============= =================================================================
Address     IP[:PORT]    Dirección en la que escucha wstunnel_
Secure      0|1          Si se cifra con SSL.
WPath       /ruta/       (Sólo cliente) Ruta del servidor en que es asccesible wstunnel_
========= ============= =================================================================

**Cliente**
   La configuración sólo requiere añadir la sección ``[Tunel]`` para advertir a **wgws**
   de que se quiere salir a través de wstunnel_:

   .. code-block:: ini

      ; /etc/wireguard/wg0.conf

      [Interface]
      Address = 10.8.0.2/24
      PrivateKey = WB4TAWIIlaOyULudlcdhqctTl/pdzO7m+6x4DhAP+0k=

      [Peer]
      PublicKey = /Pr37VgN7GVvizJw9FpCL62DSwocdNEf7lwfdDRZXj8=
      Endpoint = vpn.example.net:1194
      AllowedIPs = 10.8.0.1/32

      [Tunnel]
      ; No es necesaria configuración adicional

   Tenemos tres alternativas al arrancar la interfaz:

   #. No usar wstunnel_ (esto es, conectar directamente al puerto *1194/UDP* de
      *vpn.example.net*):

      .. code-block:: console

         # wgws -n up wg0

   #. Usar wstunnel_ sin usar SSL:

      .. code-block:: console

         # wgws up wg0

   #. Usar wstunnel_ en una conexión SSL:

      .. code-block:: console

         # wgws -s up wg0

**Servidor**
   Como en el cliente, basta con añadir la sección ``[Tunnel]`` a la configuración:

   .. code-block:: ini

      ; /etc/wireguard/wg0.conf

      [Interface]
      Address = 10.8.0.1/24
      ListenPort = 1194
      PrivateKey = kEANNMfztMtzgwFyyaWOou7+c8ZPD/lyGhmcM7oFtXA=

      [Peer]
      PublicKey = f2CH3QXHiXwFhdATcDi42DU+PUOC9Ky8BgkHBigY5H4=
      AllowedIPs = 10.8.0.2/32

      [Tunnel]
      ; Sin configuración, si se desea que wstunnel se exponga directamente.

   Esta configuración expone el websocket directamente en la interfaz física
   bien a través del puerto **80**:

   .. code-block:: console

      # wgws up wg0

   o bien a través del puerto **443** (con cifrado SSL):

   .. code-block:: console

      # wgws -s up wg0

   En caso de que se desee que un *proxy* inverso reciba las peticiones y éste
   las derive a wstunnel_, es conveniente que éste último escuche en un puerto
   libre de la interfaz de *loopback*:

   .. code-block:: ini

      [Tunnel]
      Address = 127.0.0.1:8080

   y que se delegue la responsabilidad de usar o no SSL al *proxy*.

   En cualquier caso, wireguard_ sigue escuchando en el puerto *1194/UDP* (o
   donde se quiera colocar) de todas las interfaces, por lo que, si la red
   remota lo permite, la conexión VPN puede llevarse a cabo directamente sin
   usar el *websocket*.

Más información en `aquí
<https://sio2sio2.github.io/doc-linux/07.serre/04.vpn/02.wireguard/02.confalt.html#redes-restringidas>`_.

Agenda
======
Dar soporte a la variable ``DNS`` de `wg-quick`_.

.. _wireguard: https://www.wireguard.com/
.. _wstunnel: https://github.com/erebe/wstunnel
.. _nftables: https://wiki.nftables.org/wiki-nftables/index.php/Main_Page
.. _dash: http://gondor.apana.org.au/~herbert/dash/
.. _wg-quick: https://manpages.debian.org/unstable/wireguard-tools/wg-quick.8.en.html
