****************************************
:mod:`network` --- network configuration
****************************************

.. module:: network
   :synopsis: network configuration

This module provides network drivers and routing configuration.  Network
drivers for specific hardware are available within this module and are
used to configure a hardware network interface.  Configured interfaces
are then available for use via the :mod:`socket` module. To use this module
the network build of firmware must be installed.

For example::

    # configure a specific network interface
    # see below for examples of specific drivers
    import network
    nic = network.Driver(...)
    print(nic.ifconfig())

    # now use socket as usual
    import socket
    addr = socket.getaddrinfo('micropython.org', 80)[0][-1]
    s = socket.socket()
    s.connect(addr)
    s.send(b'GET / HTTP/1.1\r\nHost: micropython.org\r\n\r\n')
    data = s.recv(1000)
    s.close()

.. only:: port_wipy or port_2wipy or port_lopy

    .. _network.Server:

    class Server
    ============

    The ``Server`` class controls the behaviour and the configuration of the FTP and telnet
    services running on the WiPy. Any changes performed using this class' methods will
    affect both.

    Example::

        import network
        server = network.Server()
        server.deinit() # disable the server
        # enable the server again with new settings
        server.init(login=('user', 'password'), timeout=600)

    Constructors
    ------------

    .. class:: network.Server(id, ...)

       Create a server instance, see ``init`` for parameters of initialization.

    Methods
    -------

    .. method:: server.init(\*, login=('micro', 'python'), timeout=300)

       Init (and effectively start the server). Optionally a new ``user``, ``password``
       and ``timeout`` (in seconds) can be passed.

    .. method:: server.deinit()

       Stop the server

    .. method:: server.timeout([timeout_in_seconds])

       Get or set the server timeout.

    .. method:: server.isrunning()

       Returns ``True`` if the server is running (connected or accepting connections), ``False`` otherwise.

.. only:: port_pyboard

    class CC3K
    ==========

    This class provides a driver for CC3000 wifi modules.  Example usage::

        import network
        nic = network.CC3K(pyb.SPI(2), pyb.Pin.board.Y5, pyb.Pin.board.Y4, pyb.Pin.board.Y3)
        nic.connect('your-ssid', 'your-password')
        while not nic.isconnected():
            pyb.delay(50)
        print(nic.ifconfig())

        # now use socket as usual
        ...

    For this example to work the CC3000 module must have the following connections:

        - MOSI connected to Y8
        - MISO connected to Y7
        - CLK connected to Y6
        - CS connected to Y5
        - VBEN connected to Y4
        - IRQ connected to Y3

    It is possible to use other SPI busses and other pins for CS, VBEN and IRQ.

    Constructors
    ------------

    .. class:: CC3K(spi, pin_cs, pin_en, pin_irq)

       Create a CC3K driver object, initialise the CC3000 module using the given SPI bus
       and pins, and return the CC3K object.

       Arguments are:

         - ``spi`` is an :ref:`SPI object <pyb.SPI>` which is the SPI bus that the CC3000 is
           connected to (the MOSI, MISO and CLK pins).
         - ``pin_cs`` is a :ref:`Pin object <pyb.Pin>` which is connected to the CC3000 CS pin.
         - ``pin_en`` is a :ref:`Pin object <pyb.Pin>` which is connected to the CC3000 VBEN pin.
         - ``pin_irq`` is a :ref:`Pin object <pyb.Pin>` which is connected to the CC3000 IRQ pin.

       All of these objects will be initialised by the driver, so there is no need to
       initialise them yourself.  For example, you can use::

         nic = network.CC3K(pyb.SPI(2), pyb.Pin.board.Y5, pyb.Pin.board.Y4, pyb.Pin.board.Y3)

    Methods
    -------

    .. method:: cc3k.connect(ssid, key=None, \*, security=WPA2, bssid=None)

       Connect to a wifi access point using the given SSID, and other security
       parameters.

    .. method:: cc3k.disconnect()

       Disconnect from the wifi access point.

    .. method:: cc3k.isconnected()

       Returns True if connected to a wifi access point and has a valid IP address,
       False otherwise.

    .. method:: cc3k.ifconfig()

       Returns a 7-tuple with (ip, subnet mask, gateway, DNS server, DHCP server,
       MAC address, SSID).

    .. method:: cc3k.patch_version()

       Return the version of the patch program (firmware) on the CC3000.

    .. method:: cc3k.patch_program('pgm')

       Upload the current firmware to the CC3000.  You must pass 'pgm' as the first
       argument in order for the upload to proceed.

    Constants
    ---------

    .. data:: CC3K.WEP
    .. data:: CC3K.WPA
    .. data:: CC3K.WPA2

       security type to use

    class WIZNET5K
    ==============

    This class allows you to control WIZnet5x00 Ethernet adaptors based on
    the W5200 and W5500 chipsets (only W5200 tested).

    Example usage::

        import network
        nic = network.WIZNET5K(pyb.SPI(1), pyb.Pin.board.X5, pyb.Pin.board.X4)
        print(nic.ifconfig())

        # now use socket as usual
        ...

    For this example to work the WIZnet5x00 module must have the following connections:

        - MOSI connected to X8
        - MISO connected to X7
        - SCLK connected to X6
        - nSS connected to X5
        - nRESET connected to X4

    It is possible to use other SPI busses and other pins for nSS and nRESET.

    Constructors
    ------------

    .. class:: WIZNET5K(spi, pin_cs, pin_rst)

       Create a WIZNET5K driver object, initialise the WIZnet5x00 module using the given
       SPI bus and pins, and return the WIZNET5K object.

       Arguments are:

         - ``spi`` is an :ref:`SPI object <pyb.SPI>` which is the SPI bus that the WIZnet5x00 is
           connected to (the MOSI, MISO and SCLK pins).
         - ``pin_cs`` is a :ref:`Pin object <pyb.Pin>` which is connected to the WIZnet5x00 nSS pin.
         - ``pin_rst`` is a :ref:`Pin object <pyb.Pin>` which is connected to the WIZnet5x00 nRESET pin.

       All of these objects will be initialised by the driver, so there is no need to
       initialise them yourself.  For example, you can use::

         nic = network.WIZNET5K(pyb.SPI(1), pyb.Pin.board.X5, pyb.Pin.board.X4)

    Methods
    -------

    .. method:: wiznet5k.ifconfig([(ip, subnet, gateway, dns)])

       Get/set IP address, subnet mask, gateway and DNS.

       When called with no arguments, this method returns a 4-tuple with the above information.

       To set the above values, pass a 4-tuple with the required information.  For example::

        nic.ifconfig(('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))

    .. method:: wiznet5k.regs()

       Dump the WIZnet5x00 registers.  Useful for debugging.

.. _network.WLAN:

.. only:: port_esp8266

    Functions
    =========

    .. function:: phy_mode([mode])

        Get or set the PHY mode.

        If the ``mode`` parameter is provided, sets the mode to its value. If
        the function is called without parameters, returns the current mode.

        The possible modes are defined as constants:
            * ``MODE_11B`` -- IEEE 802.11b,
            * ``MODE_11G`` -- IEEE 802.11g,
            * ``MODE_11N`` -- IEEE 802.11n.

    class WLAN
    ==========

    This class provides a driver for WiFi network processor in the ESP8266.  Example usage::

        import network
        # enable station interface and connect to WiFi access point
        nic = network.WLAN(network.STA_IF)
        nic.active(True)
        nic.connect('your-ssid', 'your-password')
        # now use sockets as usual

    Constructors
    ------------
    .. class:: WLAN(interface_id)

    Create a WLAN network interface object. Supported interfaces are
    ``network.STA_IF`` (station aka client, connects to upstream WiFi access
    points) and ``network.AP_IF`` (access point, allows other WiFi clients to
    connect). Availability of the methods below depends on interface type.
    For example, only STA interface may ``connect()`` to an access point.

    Methods
    -------

    .. method:: wlan.active([is_active])

        Activate ("up") or deactivate ("down") network interface, if boolean
        argument is passed. Otherwise, query current state if no argument is
        provided. Most other methods require active interface.

    .. method:: wlan.connect(ssid, password)

        Connect to the specified wireless network, using the specified password.

    .. method:: wlan.disconnect()

        Disconnect from the currently connected wireless network.

    .. method:: wlan.scan()

        Scan for the available wireless networks.

        Scanning is only possible on STA interface. Returns list of tuples with
        the information about WiFi access points:

            (ssid, bssid, channel, RSSI, authmode, hidden)

        `bssid` is hardware address of an access point, in binary form, returned as
        bytes object. You can use ``ubinascii.hexlify()`` to convert it to ASCII form.

        There are five values for authmode:

            * 0 -- open
            * 1 -- WEP
            * 2 -- WPA-PSK
            * 3 -- WPA2-PSK
            * 4 -- WPA/WPA2-PSK

        and two for hidden:

            * 0 -- visible
            * 1 -- hidden

    .. method:: wlan.status()

        Return the current status of the wireless connection.

        The possible statuses are defined as constants:

            * ``STAT_IDLE`` -- no connection and no activity,
            * ``STAT_CONNECTING`` -- connecting in progress,
            * ``STAT_WRONG_PASSWORD`` -- failed due to incorrect password,
            * ``STAT_NO_AP_FOUND`` -- failed because no access point replied,
            * ``STAT_CONNECT_FAIL`` -- failed due to other problems,
            * ``STAT_GOT_IP`` -- connection successful.

    .. method:: wlan.isconnected()

        In case of STA mode, returns ``True`` if connected to a wifi access
        point and has a valid IP address.  In AP mode returns ``True`` when a
        station is connected. Returns ``False`` otherwise.

    .. method:: wlan.ifconfig([(ip, subnet, gateway, dns)])

       Get/set IP-level network interface parameters: IP address, subnet mask,
       gateway and DNS server. When called with no arguments, this method returns
       a 4-tuple with the above information. To set the above values, pass a
       4-tuple with the required information.  For example::

        nic.ifconfig(('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))

    .. method:: wlan.config('param')
    .. method:: wlan.config(param=value, ...)

       Get or set general network interface parameters. These methods allow to work
       with additional parameters beyond standard IP configuration (as dealt with by
       ``wlan.ifconfig()``). These include network-specific and hardware-specific
       parameters. For setting parameters, keyword argument syntax should be used,
       multiple parameters can be set at once. For querying, parameters name should
       be quoted as a string, and only one parameter can be queries at time::

        # Set WiFi access point name (formally known as ESSID) and WiFi channel
        ap.config(essid='My AP', channel=11)
        # Queey params one by one
        print(ap.config('essid'))
        print(ap.config('channel'))

       Following are commonly supported parameters (availability of a specific parameter
       depends on network technology type, driver, and MicroPython port).

       =========  ===========
       Parameter  Description
       =========  ===========
       mac        MAC address (bytes)
       essid      WiFi access point name (string)
       channel    WiFi channel (integer)
       hidden     Whether ESSID is hidden (boolean)
       authmode   Authentication mode supported (enumeration, see module constants)
       password   Access password (string)
       =========  ===========



.. only:: port_wipy or port_2wipy or port_lopy

    class WLAN
    ==========

    This class provides a driver for the WiFi network processor in the module. Example usage::

        import network
        import time
        # setup as a station
        wlan = network.WLAN(mode=WLAN.STA)
        wlan.connect('your-ssid', auth=(WLAN.WPA2, 'your-key'))
        while not wlan.isconnected():
            time.sleep_ms(50)
        print(wlan.ifconfig())

        # now use socket as usual
        ...

    Constructors
    ------------

    .. class:: WLAN(id=0, ...)

       Create a WLAN object, and optionally configure it. See ``init`` for params of configuration.

    .. note::

       The ``WLAN`` constructor is special in the sense that if no arguments besides the id are given,
       it will return the already existing ``WLAN`` instance without re-configuring it. This is
       because ``WLAN`` is a system feature of the WiPy. If the already existing instance is not
       initialized it will do the same as the other constructors an will initialize it with default
       values.

    Methods
    -------

    .. method:: wlan.init(mode, \*, ssid, auth, channel, antenna)

       Set or get the WiFi network processor configuration.

       Arguments are:

         - ``mode`` can be either ``WLAN.STA`` or ``WLAN.AP``.
         - ``ssid`` is a string with the ssid name. Only needed when mode is ``WLAN.AP``.
         - ``auth`` is a tuple with (sec, key). Security can be ``None``, ``WLAN.WEP``,
           ``WLAN.WPA`` or ``WLAN.WPA2``. The key is a string with the network password.
           If ``sec`` is ``WLAN.WEP`` the key must be a string representing hexadecimal
           values (e.g. 'ABC1DE45BF'). Only needed when mode is ``WLAN.AP``.
         - ``channel`` a number in the range 1-11. Only needed when mode is ``WLAN.AP``.
         - ``antenna`` selects between the internal and the external antenna. Can be either
           ``WLAN.INT_ANT`` or ``WLAN.EXT_ANT``.

       For example, you can do::

          # create and configure as an access point
          wlan.init(mode=WLAN.AP, ssid='wipy-wlan', auth=(WLAN.WPA2,'www.wipy.io'), channel=7, antenna=WLAN.INT_ANT)

       or::

          # configure as an station
          wlan.init(mode=WLAN.STA)

    .. method:: wlan.connect(ssid, \*, auth=None, bssid=None, timeout=None)

       Connect to a wifi access point using the given SSID, and other security
       parameters.

          - ``auth`` is a tuple with (sec, key). Security can be ``None``, ``WLAN.WEP``,
            ``WLAN.WPA`` or ``WLAN.WPA2``. The key is a string with the network password.
            If ``sec`` is ``WLAN.WEP`` the key must be a string representing hexadecimal
            values (e.g. 'ABC1DE45BF').
          - ``bssid`` is the MAC address of the AP to connect to. Useful when there are several
            APs with the same ssid.
          - ``timeout`` is the maximum time in milliseconds to wait for the connection to succeed.

    .. method:: wlan.scan()

       Performs a network scan and returns a list of named tuples with (ssid, bssid, sec, channel, rssi).
       Note that channel is always ``None`` since this info is not provided by the WiPy.

    .. method:: wlan.disconnect()

       Disconnect from the wifi access point.

    .. method:: wlan.isconnected()

       In case of STA mode, returns ``True`` if connected to a wifi access point and has a valid IP address.
       In AP mode returns ``True`` when a station is connected, ``False`` otherwise.

    .. method:: wlan.ifconfig(if_id=0, config=['dhcp' or configtuple])

       With no parameters given eturns a 4-tuple of ``(ip, subnet_mask, gateway, DNS_server)``.

       if ``'dhcp'`` is passed as a parameter then the DHCP client is enabled and the IP params
       are negotiated with the AP.

       If the 4-tuple config is given then a static IP is configured. For instance::

          wlan.ifconfig(config=('192.168.0.4', '255.255.255.0', '192.168.0.1', '8.8.8.8'))

    .. method:: wlan.mode([mode])

       Get or set the WLAN mode.

    .. method:: wlan.ssid([ssid])

       Get or set the SSID when in AP mode.

    .. method:: wlan.auth([auth])

       Get or set the authentication type when in AP mode.

    .. method:: wlan.channel([channel])

       Get or set the channel (only applicable in AP mode).

    .. method:: wlan.antenna([antenna])

       Get or set the antenna type (external or internal).

    .. only:: port_wipy

        .. method:: wlan.mac([mac_addr])

           Get or set a 6-byte long bytes object with the MAC address.

        .. method:: wlan.irq(\*, handler, wake)

            Create a callback to be triggered when a WLAN event occurs during ``machine.SLEEP``
            mode. Events are triggered by socket activity or by WLAN connection/disconnection.

                - ``handler`` is the function that gets called when the irq is triggered.
                - ``wake`` must be ``machine.SLEEP``.

            Returns an irq object.

    .. only:: port_2wipy or port_lopy

        .. method:: wlan.mac()

           Get a 6-byte long ``bytes`` object with the WiFI MAC address.

    Constants
    ---------

    .. data:: WLAN.STA
    .. data:: WLAN.AP

       selects the WLAN mode

    .. data:: WLAN.WEP
    .. data:: WLAN.WPA
    .. data:: WLAN.WPA2

       selects the network security

    .. data:: WLAN.INT_ANT
    .. data:: WLAN.EXT_ANT

       selects the antenna type


.. only:: port_lopy

    class LoRa
    ==========

    This class provides a driver for the LoRa network processor in the module. Example usage::

      from network import LoRa
      import socket

      # Initialize LoRa in LORAWAN mode.
      lora = LoRa(mode=LoRa.LORAWAN)
      # create an OTAA authentication tuple (AppKey, AppEUI, DevEUI)
      auth = (bytes([0,1,2,3,4,5,6,7,8,9,2,3,4,5,6,7]), bytes([1,2,3,4,5,6,7,8]), lora.mac()))
      # join a network using OTAA (Over the Air Activation)
      lora.join(activation=LoRa.OTAA, auth=auth, timeout=0)

      # wait until the module has joined the network
      while not lora.has_joined():
          time.sleep(2.5)
          print('Not yet joined...')

      # create a LoRa socket
      s = socket.socket(socket.AF_LORA, socket.SOCK_RAW)
      s.setblocking(False)

      # send some data
      s.send(bytes([0x01, 0x02, 0x03]))

      # get any data received...
      data = s.recv(64)
      print(data)

    Constructors
    ------------

    .. class:: LoRa(id=0, ...)

       Create and configure a LoRa object. See ``init`` for params of configuration.

    Methods
    -------

    .. method:: lora.init(mode, \*, frequency=868000000, tx_power=14, bandwidth=LoRa.868000000, sf=7, preamble=8, coding_rate=LoRa.CODING_4_5, power_mode=LoRa.ALWAYS_ON, tx_iq=false, rx_iq=false, adr=false, public=true, tx_retries=1)

       Set the LoRa subsystem configuration

       The arguments are:

         - ``mode`` can be either ``LoRa.LORA`` or ``LoRa.LORAWAN``.
         - ``frequency`` accepts values between 863000000 and 870000000 in the 868 band, or between 902000000 and 928000000 in the 915 band.
         - ``tx_power`` is the transmit power in dBm. It accepts between 2 and 14 for the 868 band, and between 5 and 20 in the 915 band.
         - ``bandwidth`` is the channel bandwidth in KHz. In the 868 band the accepted values are ``LoRa.BW_125KHZ`` and ``LoRa.BW_250KHZ``. In the 915 band the accepted values are ``LoRa.BW_125KHZ`` and ``LoRa.BW_500KHZ``.
         - ``sf`` sets the desired spreading factor. Accepts values between 7 and 12.
         - ``preamble`` configures the number of pre-amble symbols. The default value is 8.
         - ``coding_rate`` can take the following values: ``LoRa.CODING_4_5``, ``LoRa.CODING_4_6``,
           ``LoRa.CODING_4_7`` or ``LoRa.CODING_4_8``.
         - ``power_mode`` can be either ``LoRa.ALWAYS_ON``, ``LoRa.TX_ONLY`` or ``LoRa.SLEEP``. In ``ALWAYS_ON`` mode, the radio is always listening for incoming packets whenever a transmission is not taking place. In ``TX_ONLY`` the radio goes to sleep as soon as the transmission completes. In ``SLEEP`` mode the radio is sent to sleep permanently and won't accept any commands until the power mode is changed.
         - ``tx_iq`` enables TX IQ inversion.
         - ``rx_iq`` enables RX IQ inversion.
         - ``adr`` enables Adaptive Data Rate.
         - ``public`` selects wether the network is public or not.
         - ``tx_retries`` sets the number of TX retries in ``LoRa.LORAWAN`` mode.

        .. note:: In ``LoRa.LORAWAN`` mode, only ``adr``, ``public`` and ``tx_retries`` are used. All the other
          params will be ignored as theiy are handled by the LoRaWAN stack directly. On the other hand, these same 3
          params are ignored in ``LoRa.LORA`` mode as they are only relevant for the LoRaWAN stack.

       For example, you can do::

          # create and configure as an access point
          lora.init(mode=LoRa.LORA, tx_power=14, sf=12)

       or::

          # configure as an station
          lora.init(mode=LoRa.LORAWAN)

    .. method:: lora.add_channel(index, \*, frequency, dr_min, dr_max, duty_cycle)

        Add a LoRaWAN channel on the specified index. If there's already a channel with that index it will be replaced with the new one.

        The arguments are:

          - ``index``: Index of the channel to add. Accepts values between 0 and 15 for EU and between 0 and 71 for US.
          - ``frequency``: Center frequency in Hz of the channel.
          - ``dr_min``: Minimmum data rate of the channel (0-7).
          - ``dr_min``: Maximum data rate of the channel (0-7).
          - ``duty_cycle``: Need to be always zero for now.

    .. method:: lora.remove_channel(index)

         Removes the channel from the specified index. Channels 0 to 2 cannot be removed, they can only be replaced by other channels using the ``lora.add_channel`` method.

    .. method:: lora.mac()

       Returns a byte object with the 8-Byte MAC address of the LoRa radio.
