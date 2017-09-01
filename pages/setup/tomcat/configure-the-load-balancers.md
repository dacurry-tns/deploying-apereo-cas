---
title: Configure the load balancers
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_configure-the-load-balancers.html
summary:
---

Once all the CAS servers have Tomcat up and running, we can configure the load balancers to route requests to them.

{% include note.html content="The configuration steps in this section are for F5 BIG-IP Local Traffic Manager (LTM) load balancers; other load balancers should provide similar capabilities." %}

{% include callout.html type="primary" content="**Correcting our biggest mistake**<br /><br />Our current CAS server listens for connections on TCP port 8447 instead of the more common 8443 or 443. The history of this decision isn't important, but experience has taught us that it was a Really Big Mistake. Many organizations limit outbound Internet access from their local area networks to small(ish) lists of well-known ports or services, and TCP port 8447 is almost never on those lists.<br /><br />If access to port 8447 is blocked, then users have no way to authenticate to CAS-enabled New School applications. This has been a problem for some of our users when using the EDUROAM roaming access service at other universities, and also for some of our students and part-time faculty who work at financial firms or other security-conscious organizations in New York City. Port 8443 (the \"usual\" port used by CAS) would have been a somewhat better choice, but not a perfect one, because there are organizations that block outbound access to that port as well.<br /><br />To eliminate this problem, and ensure that our users can authenticate regardless of what network they're using, our new CAS environment will be configured to listen for connections on TCP port 443, the standard HTTPS port." %}

## Define the CAS server nodes

Define each of the CAS server nodes so that they can be referenced elsewhere in the configuration.

```tcl
ltm node /Common/casdev-srv01 {
    address 192.168.100.101
    description "CAS Development Server 01"
}
ltm node /Common/casdev-srv02 {
    address 192.168.100.102
    description "CAS Development Server 02"
}
ltm node /Common/casdev-srv03 {
    address 192.168.100.103
    description "CAS Development Server 03"
}
```

## Define the CAS server pool

Group the CAS server nodes into a server pool. Pool members can be assigned to connections via round-robin or any other reasonable method that achieves the organization's desired results.

```tcl
ltm pool /Common/casdev_pool {
    description "CAS Development 8443 Pool"
    members {
        /Common/casdev-srv01:8443 {
            address 192.168.100.101
        }
        /Common/casdev-srv02:8443 {
            address 192.168.100.102
        }
        /Common/casdev-srv03:8443 {
            address 192.168.100.103
        }
    }
    monitor /Common/https_8443
}
```

Define a monitor for the pool to keep track of which servers are up and responding to requests. The `https_8443` monitor is based on F5's standard `https` monitor; it connects to each pool server via HTTPS on port 8443 every 5 seconds and issues a `GET /` HTTP request.

```tcl
ltm monitor https /Common/https_8443 {
    cipherlist DEFAULT:+SHA:+3DES:+kEDH
    compatibility enabled
    defaults-from /Common/https
    description "HTTPS Port 8443 Port Monitor"
    destination *:8443
    interval 5
    ip-dscp 0
    recv none
    recv-disable none
    send "GET /\\r\\n"
    time-until-up 0
    timeout 16
}
```

This is a basic monitor that just checks whether Tomcat and the server it's running on are responding. Later, after building and deploying the CAS server application, we will configure a more specific monitor (see [Define a CAS-specific service monitor on the load balancers][building_server_install-and-test-the-cas-application.html#loadbalancer]).

## Define a client SSL profile

Define a client SSL profile to enable the F5 to accept and terminate client requests using TLS/SSL. This profile specifies the TLS/SSL certificate that will be used for the connection.

```tcl
ltm profile client-ssl /Common/casdev_clientssl {
    app-service none
    cert /Common/casdev.crt
    cert-key-chain {
        casdev_ThawteSHA256IntermediateCA_Use_for_SHA256-NoSHA1crossroot {
            cert /Common/casdev.crt
            chain /Common/ThawteSHA256IntermediateCA_Use_for_SHA256-NoSHA1crossroot.crt
            key /Common/casdev.key
        }
    }
    chain /Common/ThawteSHA256IntermediateCA_Use_for_SHA256-NoSHA1crossroot.crt
    defaults-from /Common/nsu_clientssl
    inherit-certkeychain false
    key /Common/casdev.key
    passphrase none
}
```

The TLS/SSL private key and certificate files referenced in this profile are the same ones that were created for the Tomcat server in [Configure TLS/SSL settings][setup_tomcat_configure-tlsssl-settings]; they should be installed on the load balancer for use by the profile.

## Define a server profile

Define a server SSL profile to direct the F5 to access the Tomcat server pool using HTTPS instead of HTTP.

```tcl
ltm profile server-ssl /Common/casdev_serverssl {
    app-service none
    defaults-from /Common/serverssl
}
```

## Define a persistence profile

Although the basic CAS login sequence is stateless, there are some features of the server that implement flows whose steps must all be performed on the same server to preserve state. To achieve this, define a persistence profile with a timeout equal to the `server.session.timeout` property of the CAS server (5 minutes by default).

```tcl
ltm persistence source-addr /Common/casdev_persistence_profile {
    app-service none
    defaults-from /Common/cookie
    expiration 5:0
}
```

The above profile uses a session cookie based persistence profile in which the F5 sets a cookie in the user's browser. This will ensure that all connection requests from the browser session where the cookie is set are directed to the same pool member for the duration of the timeout period. Another alternative would be to use a source address based persistence profile, which would ensure that all connection requests from a particular IP address are directed to the same pool member. The cookie based approach is preferred as it is more granular, resulting in better load balancing performance.

## Enable the insertion of `X-Forwarded-For` headers
In [Configure X-Forwarded-For header processing][setup_tomcat_configure-xforwardedfor-header-processing], we configured Tomcat to process `X-Forwarded-For` HTTP headers inserted by a load balancer. Define an HTTP profile on the F5 to enable the insertion of those headers.

```tcl
ltm profile http /Common/http-casdev-profile {
    app-service none
    defaults-from /Common/http
    enforcement {
        unknown-method allow
    }
    insert-xforwarded-for enabled
    proxy-type reverse
}
```

## Define the virtual interface

Define the virtual interface that will listen on the user-facing side of the load balancer. As discussed at the beginning of this section, we want our CAS service to be available on TCP port 443, the standard HTTPS port. Configure the virtual interface to listen for connections on TCP port 443 and redirect them to TCP port 8443 on one of the pool servers. Set the interface to use the persistence profile defined above.

```tcl
ltm virtual /Common/casdev_https_vs {
    description "CAS Development https VIP"
    destination /Common/192.168.200.10:443
    ip-protocol tcp
    mask 255.255.255.255
    persist {
        /Common/casdev_persistence_profile {
            default yes
        }
    }
    pool /Common/casdev_pool
    profiles {
        /Common/casdev_clientssl {
            context clientside
        }
        /Common/casdev_serverssl {
            context serverside
        }
        /Common/http-casdev-profile { }
        /Common/tcp { }
    source 0.0.0.0/0
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
}
```

As discussed previously, CAS communications should always take place over a secure channel. Configure the virtual interface to redirect HTTP connections to HTTPS.

```tcl
ltm virtual /Common/casdev_http_vs {
    description "CAS Development http VIP"
    destination /Common/192.168.200.10:80
    ip-protocol tcp
    mask 255.255.255.255
    profiles {
        /Common/http { }
        /Common/tcp { }
    }
    rules {
        /Common/http_to_https_irule
    }
    source 0.0.0.0/0
    translate-address enabled
    translate-port enabled
}
```

## Test the servers through the load balancer

Once the F5 has been configured, repeat the testing performed earlier using the virtual address assigned to the load balancer's virtual interface:

```
https://casdev.newschool.edu
```

Since this host name corresponds to the host name in the TLS certificate installed on the CAS servers, check to ensure that no browser warnings about the certificate appear, and that the certificate chain is valid all the way back up to the root certificate authority.

Perform tests from multiple client systems with different addresses to ensure that the round-robin (or other selected method) of distributing requests across the pool is working properly.

Confirm that the IP address of the client system is displayed in the top (dark green) section of the page rather than the IP address of the load balancer. This will indicate that the `X-Forwarded-For` header processing has been properly configured.

## Perform a TLS/SSL check on the servers (optional)

If the load balancer’s virtual interface is accessible from the Internet, use the [Qualys® SSL Labs SSL Server Test][qualys-ssltest] to check that TLS/SSL is correctly configured and that the server receives an overall 'A' rating. If any other rating is received, check the test report for errors and correct them.

If the load balancer's virtual interface is not accessible from the Internet, use the [`testssl.sh` command line tool ][testssl-sh] instead. This tool performs a similar battery of tests; the principal difference is that it doesn’t assign a letter grade to the overall results.

{% include reflinks.md %}
{% include links.html %}
