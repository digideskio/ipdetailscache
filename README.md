# ipdetailscache

A Python library to gather IP address details (ASN, prefix, resource holder, reverse DNS) using the RIPEStat API, with a basic cache to avoid flood of requests and to enhance performances.

## Requirements / Third-party Libraries

Part of this work is based on Google Python IP address manipulation library (https://code.google.com/p/ipaddr-py/) and Jeff Ferland IPy library (https://github.com/autocracy/python-ipy).

You need either ipaddr or IPy; version 0.2 of this library tries to import ipaddr and falls back on IPy.

You can install them using pip:
- `pip install ipaddr`
- `pip install IPy`

## Usage

Import the library, then setup a cache object and use it to gather IP address details.
The cache object will automatically load and save data to the local cache files.

Optionally, the cache object may be instantiated with the following arguments:
- `IP_ADDRESSES_CACHE_FILE`, path to the file where IP addresses cache will be stored (default: "ip_addr.cache");
- `IP_PREFIXES_CACHE_FILE`, path to the file where IP prefixes cache will be stored (default: "ip_pref.cache");
- `MAX_CACHE`, expiration time for cache entries, in seconds (default: 604800, 1 week);
- `Debug`, set to True to enable some debug messages (default: False).

Starting from version 0.3.0, results can be enriched with Internet Exchange Points (IXPs) IP address space information.
**WARNING**: currently this feature is based on PeeringDB.com (www.peeringdb.com) **beta** API: use it at your own risk.
To enable IXPs info gathering, call the `UseIXPs` method of the cache.

Results are given in a dictionary containing the following keys:
- ASN           ["<ASN>" | "unknown" | "not announced"]
- Holder        "string"
- Prefix        "string"
- HostName      "string"
- IsIXP         [ None | bool ]
- IXPName       "string"
- TS            int

Hostname is obtained using the local socket.getfqdn function.

```
import ipdetailscache
cache = ipdetailscache.IPDetailsCache( IP_ADDRESSES_CACHE_FILE = "ip_addr.cache", IP_PREFIXES_CACHE_FILE = "ip_pref.cache", MAX_CACHE = 604800, Debug = False )
cache.UseIXPs( WhenUse=1, IXP_CACHE_FILE="ixps.cache", MAX_CACHE=604800 )
result = cache.GetIPInformation( "IP_ADDRESS" )
```

The `WhenUse` argument of `UseIXPs` method has this meaning:
- 0: do not use IXPs info;
- 1: use IXPs info only when can't determine ASN (unknown or not announced)
- 2: always use IXPs info.

## Examples

```
:~# python
Python 2.7.2+ (default, Jul 20 2012, 22:15:08)
[GCC 4.6.1] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import ipdetailscache
>>> cache = ipdetailscache.IPDetailsCache()
>>> result = cache.GetIPInformation( "193.0.6.139" )
>>> result
{'HostName': 'www.ripe.net', 'TS': 1453068601, 'Prefix': u'193.0.0.0/21', 'IsIXP': None, 'IXPName': '', 'Holder': u'RIPE-NCC-AS Reseaux IP Europeens Network Coordination Centre (RIPE NCC),NL', 'ASN': '3333'}
```

### Example with UseIXPs, WhenUse=1 (default)

80.81.203.4 = DE-CIX Hamburg IP, not announced
80.249.208.1 = AMS-IX IP, announced

```
>>> cache.UseIXPs()
>>> decix_ip="80.81.203.4"     # DE-CIX Hamburg IP, not announced
>>> amsix_ip="80.249.208.1"    # AMS-IX IP, announced
>>> result = cache.GetIPInformation(decix_ip)
>>> result
{'HostName': 'ge1-1-12-br2.hamburg10.iphh.net', 'TS': 1453068691, 'Prefix': u'80.81.203.4', 'IsIXP': True, 'IXPName': u'DE-CIX Hamburg', 'Holder': '', 'ASN': 'not announced'}
>>> result = cache.GetIPInformation(amsix_ip)
>>> result
{'HostName': 'rtr-eun-01.ams-ix.net', 'TS': 1453068704, 'Prefix': u'80.249.208.0/21', 'IsIXP': None, 'IXPName': '', 'Holder': u'AMS-IX1 Amsterdam Internet Exchange B.V.,NL', 'ASN': '1200'}
```

AMS-IX IP is announced, so IsIXP is None because no IXP info have been used here.

### Example with UseIXPs, WhenUse=2

Clear local cache with `rm *.cache`, then:

```
>>> import ipdetailscache
>>> cache = ipdetailscache.IPDetailsCache()
>>> cache.UseIXPs(WhenUse=2)
>>> decix_ip="80.81.203.4"     # DE-CIX Hamburg IP, not announced
>>> amsix_ip="80.249.208.1"    # AMS-IX IP, announced
>>> result = cache.GetIPInformation(decix_ip)
>>> result
{'HostName': 'ge1-1-12-br2.hamburg10.iphh.net', 'TS': 1453068812, 'Prefix': u'80.81.203.4', 'IsIXP': True, 'IXPName': u'DE-CIX Hamburg', 'Holder': '', 'ASN': 'not announced'}
>>> result = cache.GetIPInformation(amsix_ip)
>>> result
{'HostName': 'rtr-eun-01.ams-ix.net', 'TS': 1453068956, 'Prefix': u'80.249.208.0/21', 'IsIXP': True, 'IXPName': u'AMS-IX', 'Holder': u'AMS-IX1 Amsterdam Internet Exchange B.V.,NL', 'ASN': '1200'}
```

Here, even if AMS-IX announces its peering LAN prefix, IXPs info have been used to enrich results because WhenUse is 2.

```
>>> result = cache.GetIPInformation( "193.0.6.139" )
>>> result
{'HostName': 'www.ripe.net', 'TS': 1453068965, 'Prefix': u'193.0.0.0/21', 'IsIXP': False, 'IXPName': '', 'Holder': u'RIPE-NCC-AS Reseaux IP Europeens Network Coordination Centre (RIPE NCC),NL', 'ASN': '3333'}
```

The www.ripe.net IP is not on an IXPs peering LAN, so IsIXP == False.

## Bug? Issues?

Have a bug? Please create an issue here on GitHub at https://github.com/pierky/ipdetailscache/issues

## Author

Pier Carlo Chiodi - http://pierky.com/aboutme

Blog: http://blog.pierky.com Twitter: <a href="http://twitter.com/pierky">@pierky</a>

