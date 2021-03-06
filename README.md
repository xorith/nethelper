# Net Helper

[![Build Status](https://travis-ci.org/xorith/nethelper.svg?branch=master)](https://travis-ci.org/xorith/nethelper)

Net Helper is a small library that provides helper methods to make networking in Java easier.

## 1.1.1 Changes ##
Added a favored port setting on ports. The helper will attempt to use this port first if possible before falling back on the other possible ports allowed for a given port object.

## Usage ##

To use, follow the instructions under Set Me Up! on [Bintray](https://bintray.com/xorith/coffee/NetHelper#)

Example:
```java
package app;

import coffee.keenan.network.helpers.address.AddressHelper;
import coffee.keenan.network.helpers.port.Port;
import coffee.keenan.network.helpers.port.PortHelper;
import coffee.keenan.network.helpers.port.Protocol;
import coffee.keenan.network.wrappers.upnp.UPNPService;

import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.SocketException;

class Demo
{
    public static void main(String[] args) throws InterruptedException, SocketException
    {
        UPNPService.initialize(); // Required for port mapping services

        final InetAddress address = AddressHelper.getFirstValidAddress(); // Defaults to IPv4 w/ Internet access
        assert address != null;
        final NetworkInterface networkInterface = NetworkInterface.getByInetAddress(address);

        if (networkInterface == null) return;
        System.out.printf("Using interface %s (%s) with address of %s\n", networkInterface.getDisplayName(), networkInterface.getName(), address);

        Port[] ports = {
                PortHelper.assignPort(new Port(address, Protocol.Both).setDescription("Steam Authentication").addPort(8766).toMap()),
                PortHelper.assignPort(new Port(address, Protocol.TCP).setDescription("Wurm Unlimited").addPortRange(3724, 4724).toMap()),
                PortHelper.assignPort(new Port(address, Protocol.UDP).setDescription("Steam Queries").addPortRange(27016, 27030).toMap()),
        };

        while (!ports[0].isMapped())
            Thread.sleep(1000);

        for (final Port p : ports)
        {
            if (p.getAssignedPort() == 0 || (p.isToMap() && !p.isMapped()))
            {
                System.out.println(String.join("\n", p.getExceptions()));
                break;
            }
        }
        Thread.sleep(2000);
        UPNPService.shutdown();
    }
}
```