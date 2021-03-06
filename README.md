Rundeck WinRM Plugin
--------------------

This is a [Rundeck Node Execution plugin][1] that uses WinRM to connect to Windows and execute commands.  It uses the [OverThere Library][2] to provide the WinRM implementation, and uses Basic authentication over HTTPS.

[1]: http://rundeck.org/docs/manual/plugins.html#node-execution-plugins
[2]: https://github.com/xebialabs/overthere/

Compatible with Rundeck 1.5.x+

Install
====

Copy the `rundeck-winrm-plugin-1.1.jar` to the `libext/` directory for Rundeck.

Configure The Plugin
====

This plugin provides a NodeExecutor called `overthere-winrm`, which you can set as on your node defintion:

	<node name="winNode" node-executor="overthere-winrm" .../>

Or set as the default NodeExecutor for your project/framework properties file, with `service.NodeExecutor.default.provider=overthere-winrm`.

These Node attributes are used to connect to the remote host:

* `username` - Remote username. If using Kerberos, should be in the form "user@DOMAIN". If not, @hostname will be appended to produce the domain.
* `hostname` - Remote host. Can include "host:port" to specify port number other than the default 5985/5986 (http/https).

You must create a Rundeck Job with a [Secure Authentication Option][1], to pass in the password to use.  The default name
of this option should be "winrmPassword", but you can change the name that is expected, if necessary.

[1]: http://rundeck.org/docs/manual/job-options.html#secure-options

These additional configuration attributes can be set on the Node, or in the project.properties or framework.properties. To add them to project.properties, prefix them with "project." and for framework.properties prefix them with "framework.":

* `winrm-connection-timeout` - timeout in milliseconds for connection. (default 15000)
* `winrm-timeout` - WinRM protocol Timeout, in XML Schema Duration format. (Default: `PT60.000S`) see: <http://www.w3.org/TR/xmlschema-2/#isoformats>
* `winrm-port` - port number to use, if not set in the `hostname` of the Node. (Default: 5985/5986 for http/https)
* `winrm-user` - Username, if not set in the `username` of the Node
* `winrm-protocol` - Determine HTTP(S) protocol to use, either "http" or "https". Default: "https"
* `winrm-auth-type` - Type of authentication to use, "basic" or "kerberos", default: "kerberos".  If the username contains '@domain', then kerberos will be selected.
* `winrm-locale` - Locale to use, default: "en-us".
* `winrm-password-option` - Specifies a [Secure Authentication Option][1] from a Job to use as the authentication password. (format: "NAME" ).
	* default-value: "winrmPassword", so simply define a Secure Authentication Option on your Job with the name "winrmPassword".


Using Kerberos Authentication
====

Kerberos authentication can be used with the OverThere 2.0.0-SNAPSHOT library.

Configure these node properties, or set "framework.X" or "project.X" in your framework.properties or project.properties.

* `winrm-cert-trust` - (HTTPS only) certificate trust strategy, "all" (trust all certificates), "self-signed" (trust self-signed in addition to verified), or "default" (trust only verified certificates). Default: "default".
* `winrm-hostname-trust` - (HTTPS only) hostname trust strategy, "all", "strict" or "browser-compatible". Default: "browser-compatible".
* `winrm-kerberos-debug` - true/false, if true, enable debug output for Kerberos authentication. Default: false.
* `winrm-spn-add-port` - true/false, if true, add the port to the SPN used for authentication. Default: false.
* `winrm-spn-use-http` - true/false, if true, use 'HTTP' instead of 'WSMAN' as the protocol for the SPN used for authentication. Default: false.

Configure Kerberos
----

Kerberos authentication requires you to set up some Java System Properties, or a kb5.conf file to define which domains map to which Domain Controllers.  You can follow the [Kerberos Requirements](http://docs.oracle.com/javase/1.4.2/docs/guide/security/jgss/tutorials/KerberosReq.html) for Java.  To simply use a single domain and KDC, set these Java System Props at startup of the Rundeck server:

    -Djava.security.krb5.realm=<example.com> -Djava.security.krb5.kdc=<kdc.example.com>. 

Replace the values with the name of your domain/realm and the hostname of your domain controller.

FAQ
----

If you do not do follow the [Configure Kerberos](#configure-kerberos) section this you will see this error:

    failed: WinRM Error: javax.security.auth.login.LoginException: Could not load configuration from SCDynamicStore

If your KDC is not reachable you will see this error:

    failed: WinRM Error: javax.security.auth.login.LoginException: Receive timed out

If your password is incorrect you will see this error:

    failed: WinRM Error: javax.security.auth.login.LoginException: Pre-authentication information was invalid (24)

If your username is not found you will see this error:

    failed: WinRM Error: javax.security.auth.login.LoginException: Client not found in Kerberos database (6)

If the system clock differs too much between the nodes you will see this error:

    failed: WinRM Error: javax.security.auth.login.LoginException: Clock skew too great (37)

If you receive Kerberos authentication error `Server not found in Kerberos database` then you need to define a
"Service Principal Name" for the auth service on the Windows node:

* Run this command:

        setspn -a WSMAN/hostname.domain.com hostname

If your SPN uses a protocol of 'HTTP', or includes the port in the SPN (such as `HTTP/hostname.domain.com:5985`), see
the `winrm-spn-add-port` and `winrm-spn-use-http` configuration options above.

Configure a Windows Server for WinRM
====

This is a seperate topic, but you can follow the guide described in the OverThere project for [WINRM_HTTPS Host Setup][1].

[1]: https://github.com/xebialabs/overthere/#cifs_host_setup

Here are also some notes on the wiki: [wikilink]

Caveats
====

Note: This plugin will work against Rundeck 1.5.x or later.

Build
=====

Build with gradle or maven 2.

Gradle build, result is `build/libs/rundeck-winrm-plugin-1.1.jar`.

	gradle clean build

Maven build, result is `target/rundeck-winrm-plugin-1.1.jar`
    
    mvn clean package
